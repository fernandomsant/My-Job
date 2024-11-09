Ambiente:<br>
Ubuntu 24.04.1<br>
Docker Client 24.0.5<br>
Docker Server 24.0.5

Criando a rede na qual os containers serão inseridos
```bash
sudo docker network create jenkins
```

Adicione o seguinte Dockerfile ao seu build environment
```Dockerfile
FROM jenkins/jenkins:2.479.1-jdk17
USER root
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean docker-workflow"
```

Construa a imagem do Jenkins a partir do Dockerfile acima
```bash
sudo docker build -t myjenkins-blueocean:2.479.1-1 .
```

Crie o container a partir desta imagem
```bash
sudo docker run \
  --name jenkins-blueocean \
  --restart=on-failure \
  --detach \
  --network jenkins \
  --publish 8080:8080 \
  --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  --volume /var/run/docker.sock:/var/run/docker.sock \
  myjenkins-blueocean:2.479.1-1
```

O usuário jenkins do container recém criado ainda não possui permissão sobre o socket file docker.sock.
Para conceder permissão, adicionaremos jenkins ao grupo docker e associaremos este grupo ao socket file.

No host, visualizar o GID do grupo docker<br>
*Caso grupo docker já exista:*
```bash
getent group docker
>> docker:x:988:
# GID 988
```
*Caso grupo docker não exista:*
```bash
sudo groupadd group docker
getent group docker
>> docker:x:1001:
# GID 1001

sudo chgrp docker /var/run/docker.sock
```

No container, criar o grupo docker associando o GID do grupo no host e adicionar jenkins a ele
```bash
# No host
sudo docker exec -it --user root jenkins-blueocean bash

# No container
root> groupadd -g 988 docker
root> usermod -aG docker jenkins

# ctrl + d para sair do container

sudo docker restart jenkins-blueocean
```
