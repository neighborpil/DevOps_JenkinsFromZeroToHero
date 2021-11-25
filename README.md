# DevOps_JenkinsFromZeroToHero
Example Codes for practicing

## Installing Docker
### 1. Installing Docker
 - centos install guide: https://docs.docker.com/engine/install/centos/
 - hwo to install
```
$ sudo yum install -y yum-utils

$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
-- install docker engine    
$ sudo yum install docker-ce docker-ce-cli containerd.io
-- start docker 
$ sudo systemctl docker start

-- jenkins사용자에게 docker에 대해 그룹을 추가한다(-aG: appendGroup)
 $ sudo usermod -aG docker jenkins
--  그리고 재부팅
```
### 2. Installing Docker compose
 - https://docs.docker.com/compose/install/
```
-- docker compose 설치
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
-- docker compose를 실행 할 수 있는 권한 부여
$ sudo chmod + /usr/local/bin/docker-compose
```
