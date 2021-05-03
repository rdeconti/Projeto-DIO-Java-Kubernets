:spiral_calendar: Atualizado em 10 de abril de 2021 :heart:

<img align="right" alt="GIF" height="160px" src="https://github.com/rdeconti/rdeconti-resources/blob/main/Digital%20Innovation%20One%20-%20Logotipo.png" />

# Projeto Digital Innovation One Java

# Rodando sua aplicação Java no Kubernetes. Do deploy ao debug sem medo!

- Este projeto foi proposto pela Digital Innovation One - Link do código original: https://github.com/sandrogiacom/java-kubernetes
  e link com ferramentas diversas: https://github.com/sandrogiacom/k8s
- Professor: 
Sandro Giacomozzi
- Aulas: https://web.digitalinnovation.one/project/rodando-sua-aplicacao-java-no-kubernetes-do-deploy-ao-debug-sem-medo/learning/86a86c6c-f7ad-440e-9921-45132c98d81a?back=/track/inter-java-developer&bootcamp_id=a531bc7a-f29e-4293-85eb-e4efd6072f2b

# Descrição

Neste projeto você terá o desafio de construir um ambiente Kubernetes local para que possamos aprender a tecnologia sem medo de errar. Vamos criar os recursos necessários para fazer o deploy no cluster e configurar nossa aplicação a fim de fazer debug enquanto ela está rodando no Kubernetes.

# Detalhes da aula - Java and Kubernetes

Show how you can move your spring boot application to docker and kubernetes.
This project is a demo for the series of posts on dev.to
https://dev.to/sandrogiacom/kubernetes-for-java-developers-setup-41nk

## Part one - base app:

### Requirements:

**Docker and Make (Optional)**

**Java 14**

Help to install tools:

https://github.com/sandrogiacom/k8s

### Build and run application:

Spring boot and mysql database running on docker

**Clone from repository**
```bash
git clone https://github.com/sandrogiacom/java-kubernetes.git
```

**Build application**
```bash
cd java-kubernetes
mvn clean install
```

**Start the database**
```bash
make run-db
```

**Run application**
```bash
java --enable-preview -jar target/java-kubernetes.jar
```
**Check**

http://localhost:8080/app/users

http://localhost:8080/app/hello

## Part two - app on Docker:

Create a Dockerfile:

```yaml
FROM openjdk:14-alpine
RUN mkdir /usr/myapp
COPY target/java-kubernetes.jar /usr/myapp/app.jar
WORKDIR /usr/myapp
EXPOSE 8080
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -jar app.jar" ]
```

**Build application and docker image**

```bash
make build
```

Create and run the database
```bash
make run-db
```

Create and run the application
```bash
make run-app
```

**Check**

http://localhost:8080/app/users

http://localhost:8080/app/hello

Stop all:

`
docker stop mysql57 myapp
`
## Part three - app on Kubernetes:

We have an application and image running in docker
Now, we deploy application in a kubernetes cluster running in our machine

Prepare

### Start minikube
`make k-setup` start minikube, enable ingress and create namespace dev-to

### Check IP

`minikube -p dev.to ip`

### Minikube dashboard

`
minikube -p dev.to dashboard
`

### Deploy database

create mysql deployment and service

`
make k-deploy-db
`

`
kubectl get pods -n dev-to
`

`
kubectl logs -n dev-to -f <pod_name>
`

`
kubectl port-forward -n dev-to <pod_name> 3306:3306
`

## Build application and deploy

build app

`
make k-build-app
`

create docker image inside minikube machine:

`
make k-build-image
`

OR

`
minikube cache add java-k8s
`


create app deployment and service:

`
make k-deploy-app
`

**Check**

`
kubectl get services -n dev-to
`

To access app:

`
minikube -p dev.to service -n dev-to myapp --url
`

http://172.17.0.5:32594/app/users
http://172.17.0.5:32594/app/hello

## Check pods

`
kubectl get pods -n dev-to
`

`
kubectl -n dev-to logs myapp-6ccb69fcbc-rqkpx
`

## Map to dev.local

get minikube IP
`
minikube -p dev.to ip
`

Edit `hosts`

`
sudo vim /etc/hosts
`

Replicas
`
kubectl get rs -n dev-to
`

Get and Delete pod
`
kubectl get pods -n dev-to
`

`
kubectl delete pod -n dev-to myapp-f6774f497-82w4r
`

Scale
`
kubectl -n dev-to scale deployment/myapp --replicas=2
`

Test replicas
`
while true
do curl "http://dev.local/app/hello"
echo
sleep 2
done
`
Test replicas with wait

`
while true
do curl "http://dev.local/app/wait"
echo
done
`

## Check app url
`minikube -p dev.to service -n dev-to myapp --url`

Change your IP and PORT as you need it

`
curl -X GET http://dev.local/app/users
`

Add new User
`
curl --location --request POST 'http://dev.local/app/users' \
--header 'Content-Type: application/json' \
--data-raw '{
"name": "new user",
"birthDate": "2010-10-01"
}'
`

## Part four - debug app:

add   JAVA_OPTS: "-agentlib:jdwp=transport=dt_socket,address=*:5005,server=y,suspend=n -Xms256m -Xmx512m -XX:MaxMetaspaceSize=128m"
change CMD to ENTRYPOINT on Dockerfile

`
kubectl get pods -n=dev-to
`

`
kubectl port-forward -n=dev-to <pod_name> 5005:5005
`

## KubeNs and Stern

`
kubens dev-to
`

`
stern myapp
`

## Start all

`make k:all`


## References

https://kubernetes.io/docs/home/

https://minikube.sigs.k8s.io/docs/