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
docker container run ti --mount type=bind,src=/opt/folder_name,dst=/folder_name debian
```

:construction_worker: Testing:
- Verify if a directory folder_name exist in container
- Creates file file_name in directory folder_name and writes something in it
- Kill container
- Verify if a file is in the directory folder_name
- Run other container and repeat the the previous steps

If everything is ok proceed :heavy_check_mark:

### 2 DockerFile

Step 1: Create image with DockerFile

:warning: Third-party images may have vulnerabilities. So, create your own images!

:warning: Each RUN command creates a slice, only last slice is possible read and write. Therefore, use the RUN only once in your Dockerfile.

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


#### DockerFile 3

Up container with apache running and a HTLM page.

```
FROM debian
RUN apt-get update && apt-get install -y apache2 && apt-get clean

ENV APACHE_LOCK_DIR="/var/lock" 
ENV APACHE_PID_FILE="/var/run/apache2.pid" 
ENV APACHE_RUN_USER="www-data" 
ENV APACHE_RUN_GROUP="www-data" 
ENV APCHE_LOG_DIR="/var/log/apache2" 

#copiar arquivo index.htlm de dentro do dockerfile
COPY index.html /var/www/html/

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

Buind and run image.

#### DockerFile 4

Up container with apache running and a HTLM page.

```
FROM debian
RUN apt-get update && apt-get install -y apache2 && apt-get clean
#add permissions for user data
RUN chown www-data:www-data /var/lock/ && chown www-data:www-data /var/run/ && chown www-data:www-data /var/log/ 
#problem if bind port < 100

ENV APACHE_LOCK_DIR="/var/lock" 
ENV APACHE_PID_FILE="/var/run/apache2.pid" 
ENV APACHE_RUN_USER="www-data" 
ENV APACHE_RUN_GROUP="www-data" 
ENV APCHE_LOG_DIR="/var/log/apache2" 

#copy descompacting files (if necessary) and download remote files
ADD index.html /var/www/html/

LABEL description="WebServer"
LABEL version="1.0.0"

#definir usuario
USER root

#Default directory
WORKDIR /var/www/html/

VOLUME /var/www/html #cria automáticamente o volume
EXPOSE 80 

#principal processo do container (EXEC MODE - RECOMENDADO)
ENTRYPOINT ["/usr/sbin/apachectl"]
#apache em primeiro plano
CMD ["-D", "FOREGROUND"]

#SHELL MODE
#CMD /usr/sbin/apachectl -D FOREGROUND
```

Buind and run image.


#### MultiStage Dockerfile 5

In folder of the Dockerfile create Golang program

```
package main

import (
        "fmt"
)

func main() {
        fmt.Println("Golang inside the container")
}
```

Edit DockerFile

```
FROM golang

WORKDIR /app
ADD . /app
RUN go build -o meugo
ENTRYPOINT ./meugo
```

Build image: 

```
docker image build -t meugo:1.0 .
```

Running image:

```
docker container run -ti meugo:1.0
```


#### MultiStage Dockerfile 6

Copy all things of the Dockerfile 5 folder in a new folder

```
cp -R folder_of_dockerfile_5 new_folder
```

Dockerfile aplaing Multistage (pipeline)

FROM golang AS buildando

```
#pre container
WORKDIR /app
ADD . /app
RUN go build -o meugo

#result of pre container is used here
#aplaing multistage
FROM alpine
WORKDIR /giropops
#copiando coisas de buildando
COPY --from=buildadando /app/meugo /giropops/

ENTRYPOINT ./meugo
```

Build

```
docker image build -t meugo:2.0 .
```

Run

```
docker container run -ti meugo:2.0
```

### IMAGES DEEP DIVE

Multi-line example:

```
RUN command \
command \
command
```

:warning: Each run command create one layer. Multiple RUN cammands in Dockefile create multiple layers increasing complexity and only being able to write in the last.

#### BestPratices
- Use the .Dockerignore https://docs.docker.com/engine/reference/builder/
- Add non-root user
- Use the exec form
- Sometimes it can be good not to use the cache --no-cache
- Remmenber: one must be container is immutable and ephemeral

#### Tips
- CMD -> execute command
- ENTRYPOINT -> Main container execution process
- COPY OU ADD -> the COPY command extracts files compacted and internet files

#### Variables

Example:

```
ENV app_dir /opt/app
WORKDIR ${app_dir}
ADD .$app_dir

#LABELS (anotações)
LABEL "com.example.vendor"="LINUXtips"
LABEL com.example.label-with-value="VAIIII"
LABEL version="1.0"
LABEL description ="Aqui vc pode \
usar o multi-line" 
```

#### HEALTHECHECK

Add HEALTHECHECK in Dockerfile:
- Install Curl
- Extract ID $(docker ps -q)

```
#HEALTHECHECK
HEALTHECHECK --interval =5m --timeout=3s \
CMD curl -f http://localhost/ || exit 1
```

#### ARGS 

Example:

```
FROM ubuntu
ARG CONT_IMG_VER
ENV CONT_IMG_VER v1.0.0
RUN echo $CONT_IMG_VER


$docker build --build-arg
CONT_IMG_VER=v2.0.0 Dockerfile

#HEALTHECHECK
HEALTHECHECK --interval =5m --timeout=3s \
CMD curl -f http://localhost/ || exit 1
```

### Dockerhub

