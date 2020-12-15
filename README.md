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

### 2. install tomcat9
