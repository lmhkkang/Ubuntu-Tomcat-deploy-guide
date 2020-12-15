# ubuntu16.04 deploy java project

### 1. install jdk8 

```
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
```
mkdir /usr/lib/jvm
sudo tar -xvzf jdk-8u261-linux-x64.tar.gz
sudo mv ~/jdk1.8.0_261 /usr/lib/jvm
```
##### 환경변수 등록
```
vi /etc/environment
```
```
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/lib/jvm/jdk1.8.0_261/bin:/usr/lib/jvm/jdk1.8.0_261/jre/bin"

J2SDKDIR="/usr/lib/jvm/jdk1.8.0_261"
J2REDIR="/usr/lib/jvm/jdk1.8.0_261/jre*
JAVA_HOME="/usr/lib/jvm/jdk1.8.0_261"
```
##### java 등록
```
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

```
sudo apt-get update
sudo apt-get install default-jdk
```

apt repository에 등록된 모듈 버전 확인
```
sudo apt-cache policy <모듈>
```

### 옵션

해외망이 느릴 경우 사용
```
sudo vi /etc/apt/sources.list
```
```
:%s/archive.ubuntu.com/ftp.daum.net/g
:%s/security.ubuntu.com/ftp.daum.net/g 
```

### 2. install tomcat9

##### tomcat9 사용자 추가
```
sudo useradd -r tomcat9 --shell /bin/false
```

##### tomcat9 설치
```
cd /opt
sudo wget http://mirrors.sonic.net/apache/tomcat/tomcat-9/v9.0.41/bin/apache-tomcat-9.0.41.tar.gz
sudo tar -zxf apache-tomcat-9.0.41.tar.gz
sudo ln -s apache-tomcat-9.0.41 tomcat-latest
sudo chown -hR tomcat9: tomcat-latest apache-tomcat-9.0.41
```
##### tomcat9 service 설정파일 생성
```
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
```
sudo su
systemctl daemon-reload
systemctl start tomcat
systemctl enable tomcat
```
##### 테스트
```
sudo service tomcat status
```