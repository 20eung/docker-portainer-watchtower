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

version: '3'

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
      - /etc/localtime:/etc/localtime
      - /var/run/docker.sock:/var/run/docker.sock
      - ./data:/data
    command: -H tcp://portainer_agent:9001 --tlsskipverify

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
      - /etc/localtime:/etc/localtime
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
volumes:
  portainer_data:

networks:
  default:
    name: my_bridge
    driver: bridge
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
cat << EOF > /tmp/docker-compose.yml
version: "3"
services:
  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      TZ: Asia/Seoul
      WATCHTOWER_POLL_INTERVAL: 86400
    restart: unless-stopped

networks:
  default:
    name: my_bridge
    driver: bridge
EOF
```

### docker-compose.yml 파일 이동

```
sudo mv /tmp/docker-compose.yml /data/watchtower/docker-compose.yml
```

### docker 기동

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
