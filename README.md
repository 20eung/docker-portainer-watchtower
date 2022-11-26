# Docker, Portainer, Watchtower 설정 방법

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