# Docker, Portainer, Watch Tower 설치 방법

- Portainer는 도커를 웹으로 관리하는 툴입니다.
- Watch Tower는 도커 이미지를 자동으로 업데이트하는 관리 툴입니다.

## 0. 작업 환경

- Cloud 인스턴스: AWS, Azure, GCP, OCI 등
- OS: Ubuntu 20.04, 22.04 등

## 1. apt 업데이트

```
$ sudo apt update

$ sudo apt upgrade -y
```

## 2. Docker 설치

```
$ curl -fsSL https://get.docker.com -o get-docker.sh

$ sudo sh get-docker.sh

$ sudo systemctl enable docker

$ sudo service docker status

$ sudo apt install docker-compose -y

```

## 3. Portainer 설치

```
$ cd /

$ sudo mkdir -p /data/portainer

$ sudo vi /data/portainer/docker-compose.yml

$ cat /data/partainer/docker-compose.yml
```

```
version: '3'

services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: always
    security_opt:
      - no-new-privileges:true
    ports:
      - 9443:9443
      - 8000:8000
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /data/portainer:/data

volumes:
  portainer_data:
```

```
$ cd /data/portainer/

$ sudo docker-compose up -d
```


## 4. Watchtower 설치

```
$ mkdir -p /data/watchtower/

$ sudo vi /data/watchtower/docker-compose.yml

$ cat /data/watchtower/docker-compose.yml

```

```
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
```

```
$ cd /data/watchtower

$ sudo docker-compose up -d
```

## 5. 디렉토리 구조

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
