
# ubuntu16.04 deploy java project

  

### 1. install jdk8 (manually)

  

```bash

cd ~

wget http://download.oracle.com/otn-pub/java/jdk/8u151-b12/e758a0de34e24606bca991d704f6dcbf/jdk-8u151-linux-i586.tar.gz

```

##### 이슈

```

라이센스 이슈로 직접 다운로드 불가능

```

##### 해결

```

공식사이트에서 직접 다운로드 후 파일 이동

```

```bash

mkdir /usr/lib/jvm

sudo tar -xvzf jdk-8u261-linux-x64.tar.gz

sudo mv ~/jdk1.8.0_261 /usr/lib/jvm

```

##### 환경변수 등록

```bash
vi /etc/environment
```

```
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/lib/jvm/jdk1.8.0_261/bin:/usr/lib/jvm/jdk1.8.0_261/jre/bin"

J2SDKDIR="/usr/lib/jvm/jdk1.8.0_261"
J2REDIR="/usr/lib/jvm/jdk1.8.0_261/jre*
JAVA_HOME="/usr/lib/jvm/jdk1.8.0_261"
```

##### java 등록

```bash
sudo update-alternatives --install "/usr/bin/java" "java" "/usr/lib/jvm/jdk1.8.0_261/bin/java" 0
sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/lib/jvm/jdk1.8.0_261/bin/javac" 0
sudo update-alternatives --set java /usr/lib/jvm/jdk1.8.0_261/bin/java
sudo update-alternatives --set javac /usr/lib/jvm/jdk1.8.0_261/bin/javac

  
update-alternatives --list java
update-alternatives --list javac
```

##### 테스트

```
java -version
```

  

### 1. install jdk8 (apt-get)

  

```bash
sudo apt-get update
sudo apt-get install default-jdk
```

  

apt repository에 등록된 모듈 버전 확인

```bash
sudo apt-cache policy <모듈>
```

  

### 옵션

해외망이 느릴 경우 사용

```bash
sudo vi /etc/apt/sources.list
```

```
:%s/archive.ubuntu.com/ftp.daum.net/g
:%s/security.ubuntu.com/ftp.daum.net/g
```

  

### 2. install tomcat9

  

레퍼런스

https://www.rosehosting.com/blog/install-tomcat-9-on-an-ubuntu-16-04-vps/

  

https://websiteforstudents.com/setup-apache-tomcat9-on-ubuntu-16-04-17-10-18-04/

##### tomcat9 사용자 추가

```bash
sudo useradd -r tomcat9 --shell /bin/false
```

  

##### tomcat9 설치

```bash
cd /opt

sudo wget http://mirrors.sonic.net/apache/tomcat/tomcat-9/v9.0.41/bin/apache-tomcat-9.0.41.tar.gz

sudo tar -zxf apache-tomcat-9.0.41.tar.gz

sudo ln -s apache-tomcat-9.0.41 tomcat-latest

sudo chown -hR tomcat9: tomcat-latest apache-tomcat-9.0.41
```

##### tomcat9 service 설정파일 생성

```bash
sudo vi /etc/systemd/system/tomcat.service
```

```
[Unit]
Description=Tomcat9
After=network.target
 
[Service]
Type=forking
User=tomcat9
Group=tomcat9

Environment=CATALINA_PID=/opt/tomcat-latest/tomcat9.pid
Environment=JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
Environment=CATALINA_HOME=/opt/tomcat-latest
Environment=CATALINA_BASE=/opt/tomcat-latest
Environment="CATALINA_OPTS=-Xms512m -Xmx512m"
Environment="JAVA_OPTS=-Dfile.encoding=UTF-8 -Dnet.sf.ehcache.skipUpdateCheck=true -XX:+UseConcMarkSweepGC -XX:+CMSClassUnloadingEnabled -XX:+UseParNewGC"

ExecStart=/opt/tomcat-latest/bin/startup.sh
ExecStop=/opt/tomcat-latest/bin/shutdown.sh

[Install]
WantedBy=multi-user.target
```

##### service 재기동

```bash
sudo su

systemctl daemon-reload

systemctl start tomcat

systemctl enable tomcat
```

##### 테스트

```bash
sudo service tomcat status
```

  

### 3. deploy project

  
  
  

```bash
sudo su

vi /opt/apache-tomcat-9.0.41/conf/web.xml
```

  

##### 이슈

이클립스 프로젝트 내에 web.xml 파일세팅을 못가져옴

```
java.io.FileNotFoundException: class path resource [config/logging/log4j2-${spring.profiles.active}.xml] cannot be resolved to URL because it does not exist
```

##### 해결

unbuntu 의 tomcat9 > web.xml 파일에 추가.

```xml
<context-param>
	<param-name>spring.profiles.active</param-name>
	<param-value>dev</param-value>
</context-param>
```

위 내용을 추가해야함

  

##### 이슈

```
images, files 경로 문제
```

##### 해결

```
config/initialize/config-dev-properties
```

```
file.upload.dir= /tmp/kice_book/files
image.upload.dir= /tmp/kice_book/images
```

```bash
mkdir /tmp/kice_book/files

mkdir /tmp/kice_book/images
```

##### 배포 방법

```
/opt/apache-tomcat-9.0.41/webapps
```

위 경로에 war를 넣어주면 몇 초 뒤 읽음

  

##### 로그 디버깅 방법

```bash
tail -f /opt/apache-tomcat-9.0.41/logs/*
```

  

### 4. install jenkins

```bash
cd ~

wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -

echo deb https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list

sudo apt-get update

sudo apt-get install jenkins

sudo service jenkins start

sudo service jenkins status
```

##### jenkins 포트 변경

```bash
sudo vi /etc/default/jenkins
```

```
HTTP_PORT=9000
```

```bash
sudo service jenkins restart
```

  

##### jenkins 초기 비밀번호 확인

```

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

```

  

#### 5. install maven

```

sudo apt-get -y install maven

```

  

##### 권한 설정

```

ls -al /etc/sudoers

sudo chmod 660 /etc/sudoers

sudo vi /etc/sudoers

```

```

# User privilege specification

root ALL=(ALL:ALL) ALL

jenkins ALL=(ALL:ALL) NOPASSWD: ALL

```

```

sudo chmod 440 /etc/sudoers

ls -al /etc/sudoers

```

  

##### deploy 쉘 작성

```

echo "start deploy"

  

cd /home/vagrant/workspace/KiceBook

git pull

mvn install

mv /home/vagrant/workspace/KiceBook/target/kb-1.0.0.war /home/vagrant/workspace/KiceBook/target/ROOT.war

sudo mv /home/vagrant/workspace/KiceBook/target/ROOT.war /opt/tomcat-latest/webapps/ROOT.war

  

echo "end deploy"

```

  

##### jenkins 연동

```

sudo -Hu vagrant bash -c 'bash /home/vagrant/workspace/deploy.sh'

```

##### Vagrantfile  port-forwarding 
```
port-forwarding 시 Host의 Port가 Virtual Box에 올라와있는 다른 Box의 Host와 겹치면 안됨
Guest의 Port는 Vagrantfile 내부에서만 겹치지 않으면됨.
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTA0ODgwNzkzOF19
-->