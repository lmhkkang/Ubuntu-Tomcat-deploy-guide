
# multi-tomcat-deploy-guide

ubuntu-tomcat-deploy-guide 가이드 문서 환경세팅을 끝마친 시점을 기준환경으로 함.(아래 링크참조)  [https://github.com/lmhkkang/ubuntu-tomcat-deploy-guide](https://github.com/lmhkkang/ubuntu-tomcat-deploy-guide)

목표: 하나의 ubuntu 서버에 tomcat을 두개 올려고 하나의 톰캣당 프로젝트하나를 할당함. Jenkins 서버


## 포트지정

tomcat 1 이용시 보통 3개의 포트를 이용함.

1. tomcat 내부 포트

2. apache 연동을 위한 ajp포트

3. 서비스 포트

|       
|tomcat_1 |tomcat_2 |
|----------------|-------------------------------|-----------------------------|
# tomcat 복사

ubuntu-tomcat-deploy-guide 가이드문서를 수행했다면 `/opt` 디렉토리 하위에 `apache-tomcat-9.0.41` 가 존재함.

톰캣 2개가 필요하기 때문에 루트 티렉토리 하위에 `/opt2` 디렉토리 생성함.

# tomcat 복사

ubuntu-tomcat-deploy-guide 가이드문서를 수행했다면  `/opt`  디렉토리 하위에  `apache-tomcat-9.0.41`  가 존재함. 톰캣 2개가 필요하기 때문에 루트 티렉토리 하위에  `/opt2`  디렉토리 생성함.

```bash
sudo su
cd /
mkdir opt2
cd opt2
cp -R /opt/apache-tomcat-9.0.41 .

```

tomcat-latest 폴더 생성 후 심볼릭 링크 걸기

```bash
cd /opt2 
ln -s apache-tomcat-9.0.41 tomcat-latest

```

생성된 두번째 톰캣폴더 권한설정(폴더 생성자 변경)

```bash
cd opt2
chown -r tomcat9:tomcat9 apache-tomcat-9.0.41

```

## catalina.sh 파일수정

각각 설치된 톰캣의 bin/catalina.sh 파일에 아래 노란색 내용을 추가한다.  
단 경로는 해당 톰캣이 설치되어 있는 경로로 지정할것.

```bash
vi /opt/apache-tomcat-9.0.41/bin/catalina.sh

```

아래의 내용추가

```bash
export CATALINA_HOME=/opt/apache-tomcat-9.0.41
export TOMCAT_HOME=/opt/apache-tomcat-9.0.41
export CATALINA_BASE=/opt/apache-tomcat-9.0.41 
CATALINA_PID=/opt/apache-tomcat-9.0.41/tomcat9.pid

```

```bash
vi /opt2/apache-tomcat-9.0.41/bin/catalina.sh

```

아래의 내용추가

```bash
export CATALINA_HOME=/opt2/apache-tomcat-9.0.41
export TOMCAT_HOME=/opt2/apache-tomcat-9.0.41
export CATALINA_BASE=/opt2/apache-tomcat-9.0.41
CATALINA_PID=/opt2/apache-tomcat-9.0.41/tomcat9.pid

```

## server.xml 포트수정

```bash
vi /opt/apache-tomcat-9.0.41/conf/server.xml

```

설정했던 포트번호를 기입

```bash
vi /opt/apache-tomcat-9.0.41/conf/server.xml

```

복사 붙여넣기

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Server port="8005" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />

  <GlobalNamingResources>
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>

  <Service name="Catalina">

    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />

    <!-- Define an AJP 1.3 Connector on port 8009 -->
    <Connector protocol="AJP/1.3"
               address="::1"
               port="8009"
               redirectPort="8443" />

    <Engine name="Catalina" defaultHost="localhost">


      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>

      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">


        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />

      </Host>
    </Engine>
  </Service>
</Server>

```

```
vi /opt2/apache-tomcat-9.0.41/conf/server.xml

```

복사 붙여넣기

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Server port="18005" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />

  <GlobalNamingResources>
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>

  <Service name="Catalina">

    <Connector port="18080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />

    <!-- Define an AJP 1.3 Connector on port 8009 -->
    <Connector protocol="AJP/1.3"
               address="::1"
               port="18009"
               redirectPort="8443" />

    <Engine name="Catalina" defaultHost="localhost">


      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>

      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">


        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />

      </Host>
    </Engine>
  </Service>
</Server>

```

## tomcat 서비스 재시작 및 상태체크

```bash
service tomcat restart
service tomcat status

```

## tomcat2 서비스 만들기

톰캣 서비스를 하나 추가하였기 때문에  `service tomcat2 start/stop/restart`  커맨드를 수행할 수 있도록 서비스생성

```bash
cd /etc/systemd/system
vi tomcat2.service

```

tomcat2.service 복사후 저장.

```
[Unit]
Description=Tomcat9
After=network.target

[Service]
Type=forking
User=tomcat9
Group=tomcat9

Environment=JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
Environment="CATALINA_OPTS=-Xms512m -Xmx512m"
Environment="JAVA_OPTS=-Dfile.encoding=UTF-8 -Dnet.sf.ehcache.skipUpdateCheck=true -XX:+UseConcMarkSweepGC -XX:+CMSClassUnloadingEnabled -XX:+UseParNewGC"

ExecStart=/opt2/tomcat-latest/bin/startup.sh
ExecStop=/opt2/tomcat-latest/bin/shutdown.sh

[Install]
WantedBy=multi-user.target
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbODk5NTcxNjQ1LC0xMjczNDA2NSwxMTgxNT
A0MzEyXX0=
-->