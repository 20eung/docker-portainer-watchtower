# Docker, Portainer, Watch Tower 설치 방법

- Portainer는 도커를 웹으로 관리하는 툴입니다.
- Watch Tower는 도커 이미지를 자동으로 업데이트하는 관리 툴입니다.

## 0. 작업 환경

- Cloud 인스턴스: AWS, Azure, GCP, OCI 등
- OS: Ubuntu 20.04, 22.04 등

## 1. apt 업데이트

```
sudo apt update

sudo apt upgrade -y
```

## 2. Docker 설치

```
curl -fsSL https://get.docker.com -o get-docker.sh

sudo sh get-docker.sh

sudo systemctl enable docker

sudo service docker status

sudo apt install docker-compose -y

```

> 도커 그룹에 사용자 추가

```
$ sudo usermod -aG docker $USER

```

> 도커 브릿지 네트워크 추가

```
$ sudo docker network create my_bridge
```

## 3. Portainer 설치

```
sudo mkdir -p /data/portainer
```

### docker-compose.yml 파일 생성

```
cat << EOF > /tmp/docker-compose.yml

version: '3.8'

services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    hostname: portainer
    restart: always
    environment:
      - TZ=Asia/Seoul
    security_opt:
      - no-new-privileges:true
    ports:
      - 8000:8000
      - 9443:9443
    volumes:
      - ./data:/data
      - /var/run/docker.sock:/var/run/docker.sock
    command: -H tcp://portainer_agent:9001 --tlsskipverify
    networks:
      - server-base-net  # 외부 네트워크 연결

  agent:
    image: portainer/agent:latest
    container_name: portainer_agent
    hostname: portainer_agent
    restart: always
    environment:
      - TZ=Asia/Seoul
    ports:
      - 9001:9001
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    networks:
      - server-base-net  # 외부 네트워크 연결

networks:
  server-base-net:
    external: true

EOF
```

### docker-compose.yml 파일 이동

```
sudo mv /tmp/docker-compose.yml /data/portainer/docker-compose.yml
```

### docker 기동

```
cd /data/portainer/

sudo docker-compose up -d
```


## 4. Watchtower 설치

```
sudo mkdir -p /data/watchtower/

```

### docker-compose.yml 파일 생성

```
version: '3.8'

services:
  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    hostname: watchtower
    restart: unless-stopped
    environment:
      - TZ=Asia/Seoul
      - WATCHTOWER_POLL_INTERVAL=86400
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_INCLUDE_STOPPED=false
      - TELEGRAM_BOT_TOKEN=${TELEGRAM_BOT_TOKEN}
      - TELEGRAM_CHAT_ID=${TELEGRAM_CHAT_ID}
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - server-base-net  # 외부 네트워크 연결

  watchtower-notifier:
    image: my-watchtower-notifier:latest  # 커스텀 이미지 사용
    container_name: watchtower_notifier
    hostname: watchtower_notifier
    restart: unless-stopped
    environment:
      - TZ=Asia/Seoul
      - TELEGRAM_BOT_TOKEN=${TELEGRAM_BOT_TOKEN}
      - TELEGRAM_CHAT_ID=${TELEGRAM_CHAT_ID}
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./watchtower-notifier-custom/scripts:/scripts
    networks:
      - server-base-net

networks:
  server-base-net:
    external: true
```

### .env 파일 생성

```
# Telegram 설정
TELEGRAM_BOT_TOKEN="561745752:AAGQcvoy9fTNgm7kIav-GvaPIIKx6WJeTRU"
TELEGRAM_CHAT_ID="204089935"
```

### 도커 컨테이너 업데이트 메시지를 Telegram으로 전송하기 위한 설정

#### 1. 커스텀 컨테이너 생성을 위한 Dockerfile 생성

##### 파일 위치: ./watchtower-notifier-custom/Dockerfile

```
# watchtower-notifier-custom/Dockerfile

FROM alpine:latest

# 패키지 매니저 업데이트 및 curl, docker-cli, bash 설치
RUN apk update && \
    apk add --no-cache curl docker-cli bash

# 스크립트 디렉토리 생성
RUN mkdir -p /scripts

# 스크립트 파일 복사
COPY scripts/notify_telegram.sh /scripts/notify_telegram.sh
COPY scripts/monitor_watchtower.sh /scripts/monitor_watchtower.sh

# 스크립트에 실행 권한 부여
RUN chmod +x /scripts/notify_telegram.sh /scripts/monitor_watchtower.sh

# 작업 디렉토리 설정
WORKDIR /scripts

# 엔트리포인트 설정
ENTRYPOINT ["sh", "/scripts/monitor_watchtower.sh"]

```

#### 2. watchtower 로그를 모니터링 하는 스크립트

##### 파일 위치: ./watchtower-notifier-custom/scripts/monitor_watchtower.sh

```
#!/bin/bash

# watchtower-notifier-custom/scripts/monitor_watchtower.sh

# Watchtower 로그 파일 위치 (도커 로그를 직접 모니터링)
DOCKER_CONTAINER_NAME="watchtower"

# 실시간으로 Watchtower 로그를 모니터링
docker logs -f "$DOCKER_CONTAINER_NAME" 2>&1 | while read -r line; do
    # 업데이트 완료 메시지 패턴 예시 (Watchtower 로그 형식에 따라 조정 필요)
    if [[ "$line" == *"has been updated"* ]]; then
        # 알림 메시지 구성
        MESSAGE="$line"

        # Telegram 알림 스크립트 실행
        /scripts/notify_telegram.sh "$MESSAGE"
    fi
done

```

#### 3. watchtower 로그를 텔레그램으로 전송하는 스크립트

##### 파일 위치: ./watchtower-notifier-custom/scripts/notify_telegram.sh

```
#!/bin/bash

# Telegram 봇 토큰과 채팅 ID 환경 변수
BOT_TOKEN="${TELEGRAM_BOT_TOKEN}"
CHAT_ID="${TELEGRAM_CHAT_ID}"

# 메시지 내용: 실제 줄바꿈 포함
MESSAGE=$(printf "🎉 **Watchtower 알림** 🎉\n\n%s" "$1")

# Telegram API를 통해 메시지 전송
RESPONSE=$(curl -s -X POST "https://api.telegram.org/bot${BOT_TOKEN}/sendMessage" \
     -d chat_id="${CHAT_ID}" \
     -d parse_mode="Markdown" \
     -d text="${MESSAGE}")

echo "Telegram API Response: ${RESPONSE}"
```

#### 4. watchtower 로그를 텔레그램으로 전송하는 테스트 스크립트

##### 파일 위치: ./watchtower-notifier-custom/scripts/test_telegram.sh

```
#!/bin/bash

# Telegram 봇 토큰과 채팅 ID 환경 변수
BOT_TOKEN="561745752:AAGQcvoy9fTNgm7kIav-GvaPIIKx6WJeTRU"
CHAT_ID="204089935"

# 테스트 메시지 내용
MESSAGE=$(printf "**Watchtower 알림**\n\n%s**컨테이너:** Watchtower\n%s**이미지:** latest\n%s**상태 코드:** Success\n%s**로그:**\n%sThanks")

# Telegram API를 통해 메시지 전송
curl -s -X POST "https://api.telegram.org/bot${BOT_TOKEN}/sendMessage" \
     -d chat_id="${CHAT_ID}" \
     -d parse_mode="Markdown" \
     -d text="${MESSAGE}"
```

#### 5. docker 기동

```
cd /data/watchtower

sudo docker-compose up -d
```


## 5. connect docker container

```
docker exec -it portainer /bin/sh

docker exec -it watchtower /bin/sh
```


## 6. 디렉토리 구조

```
/data
├── portainer
│   ├── bin
│   ├── certs
│   │   ├── cert.pem
│   │   └── key.pem
│   ├── compose
│   ├── docker-compose.yml
│   ├── docker_config
│   │   └── config.json
│   ├── portainer.db
│   ├── portainer.key
│   ├── portainer.pub
│   └── tls
└── watchtower
    └── docker-compose.yml
```

***
## 참고 URL

- Ubuntu 20.04 Docker 설치하기: https://blog.dalso.org/linux/ubuntu-20-04-lts/13118
- Docker 컨테이너 자동시작: https://help.iwinv.kr/manual/read.html?idx=572
- Docker 이미지 자동 업데이트 툴 Watch Tower 설치하기: https://it-svr.com/docker-image-update-service-watchtower/
- Docker를 Web에서 관리하는 Portainer 설치방법: https://www.wsgvet.com/ubuntu/120
