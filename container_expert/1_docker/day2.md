# Notes of Day 2

## Description

This file have notes about:
1. Volumes;
2. DockerFile;
3. Dockerhub.

### 1 Volumes

All data is lost when the container is finalized, for resolve this issue then use the volumes.  A volume is a piece of the filesystem that can store all data of a container.

#### Create a volume

Types of volumes:
- Bind: directory of host is mount inside container
- Volume: 

##### Create volume bind type

Step 1: create folder in host

```
mkdir /opt/folder_name
```

Step 2: 

```
docker container run ti --mount type=bind,src=/opt/giropops,dst=/giropops debian
```

### 2 DockerFile

Step 1: Create image with DockerFile

:exclamation: Third-party images may have vulnerabilities. So, create your own images!

:exclamation: Each RUN command creates a slice, only last slice is possible read and write. Therefore, use the RUN only once in your Dockerfile.

#### DockerFile 1

```
FROM debian
RUN apt-get update && apt-get install -y apache2 && apt-get clean

# setup enviroment variables
ENV APACHE_LOCK_DIR="/var/lock" 
#arquivo de lock, impede que vc tenha dois processos da apache rodadando
ENV APACHE_PID_FILE="/var/run/apache2.pid" 
#qual o pid do linux
ENV APACHE_RUN_USER="www-data" 
#usuario responsável
ENV APACHE_RUN_GROUP="www-data" 
#grupo
ENV APCHE_LOG_DIR="/var/log/apache2" 
#onde salva os logs

LABEL description="WebServer"
LABEL version="1.0.0"

VOLUME /var/www/html #cria automáticamente o volume
EXPOSE 80 
# open port TCP 80 (HTTP)
#called with P paramether (bind with host random port)

```

Build Image example:

```
docker image build -t meu_apache:1.0.0 .
```

Checking image:

```
docker image ls
```

Run image:

```
docker container run -ti meu_apache:1.0.0 
```

#### DockerFile 2 

Up container with apache running.

```
FROM debian
RUN apt-get update && apt-get install -y apache2 && apt-get clean

ENV APACHE_LOCK_DIR="/var/lock" 
ENV APACHE_PID_FILE="/var/run/apache2.pid" 
ENV APACHE_RUN_USER="www-data" 
ENV APACHE_RUN_GROUP="www-data" 
ENV APCHE_LOG_DIR="/var/log/apache2" 

LABEL description="WebServer"
LABEL version="1.0.0"

VOLUME /var/www/html #cria automáticamente o volume
EXPOSE 80 

#principal processo do container (EXEC MODE - RECOMENDADO)
ENTRYPOINT ["/usr/sbin/apachectl"]
#apache em primeiro plano
CMD ["-D", "FOREGROUND"]

#SHELL MODE
#CMD /usr/sbin/apachectl -D FOREGROUND

```

Build

```
docker image build -t meu_apache:2.0.0 .
```

Run image:

```
docker container run -d -p 8080:80 meu_apache:2.0.0
```
view apache2 web page

ip_of_host_where_up_container:8080
