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

### ※ How to connect another linux os by ssh
 - using remote_host as internal dns
1. create Dockerfile
```
FROM centos:7

RUN yum -y install openssh-server

RUN useradd remote_user && \
    echo "1234" | passwd remote_user --stdin && \
    mkdir /home/remote_user/.ssh && \
    chmod 700 /home/remote_user/.ssh

COPY remote-key.pub /home/remote_user/.ssh/authorized_keys

RUN chown remote_user:remote_user -R /home/remote_user/.ssh/ && \
    chmod 600 /home/remote_user/.ssh/authorized_keys

RUN /usr/sbin/sshd-keygen

CMD /usr/sbin/sshd -D

```
2. add remote_host to docker-compose.yml
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
  remote_host:
    container_name: remote-host
    image: remote-host
    build:
      context: centos7
    networks:
      - net
networks:
  net:

```
3. run jenkins process
```
$ docker exec -it jenkins bash

-- inside jenkins process

$ ssh remote_user@remote_user
-- you should enter password to login

-- ※Conneting to remote_host by ssh key(inside jenkins)
$ ssh -i remote-key remote_user@remote_host
```

## Connet to aws cli

 - centos7/Dockerfile
```
FROM centos:7

RUN yum -y update && yum -y install openssh-server

RUN useradd remote_user && \
    echo "1234" | passwd remote_user --stdin && \
    mkdir /home/remote_user/.ssh && \
    chmod 700 /home/remote_user/.ssh

COPY remote-key.pub /home/remote_user/.ssh/authorized_keys

RUN chown remote_user:remote_user -R /home/remote_user/.ssh/ && \
    chmod 600 /home/remote_user/.ssh/authorized_keys

RUN /usr/sbin/sshd-keygen

RUN yum -y install mysql

RUN yum -y install epel-release && \
    yum -y install python3-pip && \
    pip3 install --upgrade pip && \
    pip3 install awscli

CMD /usr/sbin/sshd -D
```

 - docker-compose.yml
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
  remote_host:
    container_name: remote-host
    image: remote-host
    build:
      context: centos7
    networks:
      - net
  db_host:
    container_name: db
    image: mysql:5.7
    environment:
      - "MYSQL_ROOT_PASSWORD=1234"
    volumes:
      - "$PWD/db_data:/var/lib/mysql"
    networks:
      - net
networks:
  net:
```

### Mysql dump
```
# mysqldump -u root -h db_host -p testdb > /tmp/db.sql
```

### Upload file to aws
```
# export AWS_ACCESS_KEY_ID=AKIA4XWXSBRET3WGGWNO
# export AWS_SECRET_ACCESS_KEY=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
```
 - https://docs.aws.amazon.com/cli/latest/reference/s3/cp.html
```
-- upload fild to aws bucket
# aws s3 cp /tmp/db.sql s3://jenkins-mysql-backup-feelong/db.sql
```


### Createting dbdump script
1. Creating a file /tmp/script.sh
```
-- vi /tmp/script.sh

#/bin/bash

DB_HOST=$1
DB_PASSWORD=$2
DB_NAME=$3

mysqldump -u root -h $DB_HOST -p$DB_PASSWORD $DB_NAME > /tmp/db.sql
```
2. Make it executable 
```
# chmod +x /tmp/script.sh
```
3. Execute
```
# /tmp/script.sh db_host 1234 testdb
```

### uploa db backup using Jenkins

```
#/bin/bash

DATE=$(date +%H-%M-%S)
BACKUP=db-$DATE.sql
DB_HOST=$1
DB_PASSWORD=$2
DB_NAME=$3
AWS_SECRET=$4
BUCKET_NAME=$5

mysqldump -u root -h $DB_HOST -p$DB_PASSWORD $DB_NAME > /tmp/$BACKUP && \
export AWS_ACCESS_KEY_ID=AKIA4XWXSBRET3WGGWNO && \
export AWS_SECRET_ACCESS_KEY=$AWS_SECRET && \
echo "Uploading your db backup" && \
aws s3 cp /tmp/$BACKUP s3://$BUCKET_NAME/$BACKUP
```

### ※ 도커 프로세스 강제로 죽이기
```
$ docker rm -fv remote-host # --force, --volumns
```

## Ansible
 - hosts
```
[all:vars]

ansible_connection = ssh

[test]
test1 ansible_host=remote_host ansible_user=remote_user ansible_private_key_file=/var/jenkins_home/ansible/remote-key
```

 - play.yml
```
- hosts: test1
  tasks:

    - shell: echo Hello World from Ansible > /tmp/ansible-file
```
 - 파일 복사
```
$ cp play.yml ../jenkins_home/ansible
```
 - playbook 실행
```
$ ansible-playbook -i hosts play.yml
```
### Install Ansible plugin on Jenkins
1. Jenkins 관리 - 플러그인 관리 - 설치가능 - Ansible 검색
2. 설치 후 재부팅 체크
3. 재부팅후 Jenkins 관리 - 플러그인 관리 - 설치된 플러그인 목록에서 확인

#### ※ 리눅스에서 ~기호의 의미
 - 홈디렉토리를 의미한다



### run ansible playbook on jenkins

 - ansible playbook(play.yml)
```
- hosts: test1
  tasks:

    - shell: echo Hello World from Ansible again a > /tmp/ansible-file
```

### Send Parameters at Jenkins Ansible
 1. Set Parameter insde play.yml
```
- hosts: test1
  tasks:
    - debug:
      msg: "{{ MSG }}"
```
 2. set parameter at Jenkins configuration

![image](https://user-images.githubusercontent.com/22423285/147009994-3a100c78-de73-4c83-965a-22de55d23606.png)

![image](https://user-images.githubusercontent.com/22423285/147010027-9a79c99e-38ff-439d-8778-1181437ee478.png)

 3. Click Build - Advanced

![image](https://user-images.githubusercontent.com/22423285/147010098-cb2e39d2-900d-4360-a5e1-5c77374de2e6.png)

 4. Connect extra variables

![image](https://user-images.githubusercontent.com/22423285/147010140-1b5a31ae-fb38-435d-b38c-584e49bb765a.png)


5. Save

### Colorize jenkins output

 1. Installing ansible color plugin: Manage Jenkins - plugin Manager - Available - AnsiColor - Install without restart
 2. go to project and configure
 3. go to Build Enviroment and check Color ANSI Console Output
 4. Build - Advanced section: check Colorize stdout


#### Linux command: nl 파일명
 - 내용을 라인넘버를 붙여서 보여준다

![image](https://user-images.githubusercontent.com/22423285/147011729-0983146d-8eff-4905-aaf0-5ea9ad5cc37b.png)

#### Linux command : grep -w옵션
 - 단어 단위로 매칭

![image](https://user-images.githubusercontent.com/22423285/147011833-87aa244e-1fc6-48c4-9924-fdad2da9204b.png)

#### Linux command : awk
 1. awk '{print $숫자}'
  - 단어를 스페이스?단위로 잘라 보여준다
  - 1부터 시작

![image](https://user-images.githubusercontent.com/22423285/147012015-3f8c93f1-f547-4e52-aa7a-490ba21c0c29.png)

 2. awk -F ',' '{print $1}'
  - peoplo.txt
```
Denice,Caudle  
Cherise,Olenick  
Nohemi,Overlock  
Tom,Fellers  
Teri,Mess  

```
  - 콤마로 잘라서 첫번째거를 보여준다
```
nl people.txt | grep -w 1 | awk '{print $2}' | awk -F ',' '{print $2}'
```

![image](https://user-images.githubusercontent.com/22423285/147012214-2a310983-6d38-48e3-aae6-d529dcf4dbca.png)




### Creating sh
 - put.sh
```

#!/bin/bash

counter=0

while [ $counter -lt 50 ]; do
  let counter=counter+1

  name=$(nl people.txt | grep -w $counter | awk '{print $2}' | awk -F ',' '{print $1}')
  lastname=$(nl people.txt | grep -w $counter | awk '{print $2}' | awk -F ',' '{print $2}')
  # echo "name for person with id $counter is $name, $lastname"
done

```

### Linux command: shuf
 - 랜덤넘버 생성
 - 최대 i 옵션의 개수만큼만 반환
 - -i 시작숫자-끝숫자 : 시작숫자 ~ 끝숫자 까지 랜덤넘버를 표시

```
$ shuf -i 20-25
```
![image](https://user-images.githubusercontent.com/22423285/147167484-47b5de57-1266-45cc-82cd-cb7d9071747b.png)

 - -n 숫자 : 숫자 개수만큼 반환
```
$ shuf -i 20-25 -n 3
```
![image](https://user-images.githubusercontent.com/22423285/147167604-660d3e4a-a8e4-4778-87e3-19b1deceeb1e.png)



