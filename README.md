Ambiente:<br>
Ubuntu 24.04.1<br>
Docker Client 24.0.5<br>
Docker Server 24.0.5

Criando a rede na qual os containers serão inseridos
```bash
docker network create jenkins
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
docker build -t myjenkins-blueocean:2.479.1-1 .
```

Crie o container a partir desta imagem
```bash
docker run \
  --name jenkins-blueocean \
  --restart=on-failure \
  --detach \
  --network jenkins \
  --publish 8080:8080 \
  --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:2.479.1-1
```
