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
$ sudo systemctl start docker

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
$ sudo chmod +x /usr/local/bin/docker-compose
```

※ du -sh 폴더
 - 폴더의 용량을 확인하는 명령어
```
$ sudo du -sh /var/lib/docker
```

※ mv 기존폴더명 바꿀폴더명
 - 폴더명 바꾸기
```
$ mv jenkins/ jenkins-data
```

※ $PWD
 - 현재 폴더의 절대경로를 변수로 나타낸다
```
$ echo $PWD
-- 결과: /home/jenkins/jenkins-data와 같이 현재 폴더를 나타낸다
```
※ id
 - 현재 계정의 아이디 확인
```
$ id
-- 결과
-- uid=1000(jenkins) gid=1000(jenkins) groups=1000(jenkins),10(wheel),994(docker) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```
※ sudo chown 아이디:그룹아이디 폴더명 -R
 - 폴더의 소유권을 설정
 - id를 통하여 uid:gid로 아이디를 특정
 - -R: Recursive 옵션명
```
$ sudo chown 1000:1000 jenkins_home -R
```

### 3. Run Jenkins image
#### docker-compose.yml
 - 도커 설정파일, 3버전으로 작성
```
version: '3'
services:
  jenkins:
    container_name: jenkins
    image: jenkins/jenkins
    ports:
      - "8080:8080"
    volumes:
      - "$PWD/jenkins_home:/var/jenkins_home"
    networks:
      - net
networks:
  net:
```
 - 프로그램 실행
 - docker-comopse.yml파일을 읽어들여 실행
```
$ docker-compose up -d
```
 - 실행뒤 로그 확인
```
$ docker logs -f jenkins
-- 로그에서 pwd를 확인해야 한다.
```
 - 프로그램 종료
```
$ docker-compose stop
```
 - 프로그램 재시작
```
$ docker-compose restart jenkins
```
 - 프로그램 삭제
   + 관련된 모든 것을 지운다
   + 하지만 volumes에 저장된 것은 삭제하지 않는다
```
$ docker-compose down
```
 - 도커 프로세스안으로 접속
```
$ docker exec -ti jenkins bash
```
 - 텍스트 출력
```
$ echo "Hello, $NAME. Current date and tiem is$(date)"
```

 - 젠킨스에서 실행할 스크립트
```
NAME=Feelong
echo "Hello, $NAME. Current date and tiem is$(date)" > /tmp/info
```

### Run script file inside docker process
1. create script file
```
$ vi script.sh
```
```
-- inside script.sh
#!/bin/bash

NAME=$1
LASTNAME=$2

echo "Hello, $NAME $LASTNAME"

```
2. give excutable permission
```
$ chmod +x script.sh
```

3. copy file to docker process
```
$ docker cp script.sh jenkins:/tmp/script.sh
```

4. check that the copied file is exists
```
$ docker exec -it jenkins bash

$ cat /tmp/script.sh
```

5. run script on Jenkins dashboard
 - select job
 - change text inside execute sheel box
```
/tmp/script.sh Feelong Park
```
```
NAME=Feelong
LASTNAME=PARK
/tmp/script.sh $NAME $LASTNAME
```

### Use boolean parameter on the Jenkins
```
#!/bin/bash

NAME=$1
LASTNAME=$2
SHOW=$3

if [ "$SHOW" = "true" ]; then
        echo "Hello, $NAME $LASTNAME"
else
        echo "If you want to see the name, please mark show option"
fi

```
