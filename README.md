# Kubernetes Java Example

## Project Overview

Spring Boot 프레임워크와 쿠버네티스를 이용한 방명록 애플리케이션 예제 프로젝트.

## Architecture

![Logical Architecture](./doc/images/logical_architecture.png)
<p align="center">[Figure 1: Logical Architecture]</p>

이 프로젝트에는 다음의 소프트웨어 및 프레임워크가 사용되었음:

- **Minikube** (로컬 개발용 싱글 노드 쿠버네티스 클러스터)
- **Skaffold** (CI/CD tool)
- **Spring Boot Framework** (Backend/Frontend)
- **MongoDB** (데이터베이스)

## Project Structure

다음의 프로젝트 구조를 반드시 따를 필요는 없지만 관례상 프로젝트 root에 skaffold.yaml, kubernetes-manifests 그리고 src 디렉토리 구조를 유지하는 것을 추천함.

~~~bash
.
|---- .vscode
|      └---- launch.json
|---- kubernetes-manifests
|     |---- guestbook-backend.deployment.yaml
|     |---- guestbook-backend.service.yaml
|     |---- guestbook-frontend.deployment.yaml
|     |---- guestbook-frontend.service.yaml
|     |---- mongo.deployment.yaml
|     └---- mongo.service.yaml
|---- src
|     |---- backend
|     |     |---- Dockerfile
|     |     |---- index.js
|     |     |---- app.js
|     |     └---- package.json
|     |---- frontend
|           |---- Dockerfile
|           |---- index.js
|           |---- app.js
|           └---- package.json
└---- skaffold.yaml
~~~

## Prerequisites

### AdoptOpenJDK 14

Java 기반의 프로젝트이기 때문에 반드시 JDK가 설치 되어야 함. 다음의 링크를 따라서 본인의 개발 환경에 맞는 JDK로 설치 필요. 가급적 버전 11 이상으로 설치 권고.

- [AdoptOpenJDK](https://adoptopenjdk.net/)

OpenJDK 설치 이후에 JAVA_HOME 환경변수 설정이 필요하다. Linux/MacOS 사용자는 ~/.bashrc 혹은 ~/.bash_profile 파일에 다음과 같이 JAVA_HOME 환경변수를 설정한다.

~~~bash
# Setting for OpenJDK
export JAVA_HOME=$(/usr/libexec/java_home)
~~~

### Visual Studio Code

본 예제는 Visual Studio Code를 IDE로 사용하고 Java, Kubernetes 및 Skaffold와 관련된 Extension을 필요로 함. 다음의 링크를 따라 설치 필요.

- [Visual Studio Code](https://code.visualstudio.com)
- [Java Dependency Viewer](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-dependency)
- [Java Extension Pack](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack)
- [Java Test Runner](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-test)
- [Cloud Code](https://marketplace.visualstudio.com/items?itemName=GoogleCloudTools.cloudcode)

### Minikube, Kubernetes and Skaffold

모든 쿠버네티스 애플리케이션은 쿠버네티스 클러스터를 필요로 한다. 본 프로젝트에서는 로컬 개발환경 구성을 위하여 Minikube를 사용한다. Minikube는 컨테이너 런타임을 제공하기 때문에 별도의 Docker 엔진 설치가 필요 없다.

CI/CD를 구성하기 위한 많은 옵션이 있지만 본 프로젝트에서는 Skaffold 사용을 권고한다.

다음의 링크를 따라 필요한 구성 요소를 설치한다.

- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [Minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/)
- [Skaffold](https://skaffold.dev/docs/install/)

> Note: MacOS에서는 Minikube 실행에 필요한 VM Driver로 가볍고 빠른 Hyperkit을 추천.

## Deployment

### Minikube 설정

Minikube에서 사용할 Memory, CPU, VM-Driver 및 Disk 사이즈는 다음의 명령어로 설정할 수 있다.

~~~bash
# set memory, cpu and vm driver
minikube config set memory 4096
minikube config set cpus 4
minikube config set vm-driver hyperkit

# increase minikube disk size, default is 2GB
minikube config set disk-size 8000
~~~

변경된 설정은 ```~/.minikube/config/config.json``` 파일에 저장되며 ```minikube delete``` 명령어를 통해 minikube cluster를 삭제하기 전까지 유효하다.

현재 Minikube의 설정은 다음의 명령어로 확인할 수 있다.

~~~ bash
minikube config view
~~~

### Minikube 시작하기

다음의 명령어로 Minikube를 실행할 수 있다.

~~~ bash
minikube start
~~~

Minikube로 개발할 때는 별도의 Docker for Windows나 Docker for Mac 없이 Minikube가 내장하고 있는 Docker 데몬 및 Image Registry를 사용하여 개발 효율을 높일 수 있다.

Linux나 Mac 환경에서 다음과 같이 ```minikube docker-env``` 명령어로 Minikube에 내장되어 있는 Docker 데몬을 사용할 수 있다.

~~~ bash
Command Prompt> minikube docker-env
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.64.5:2376"
export DOCKER_CERT_PATH="/Users/nuno/.minikube/certs"
export MINIKUBE_ACTIVE_DOCKERD="minikube"

# To point your shell to minikube's docker-daemon, run:
# eval $(minikube -p minikube docker-env)

Command Prompt> eval $(minikube -p minikube docker-env)
~~~

위의 명령어를 실행하였다면 Minikube의 내장 Docker 데몬을 사용할 수 있으며 다음의 명령어로 확인이 가능하다.

~~~ bash
docker ps
~~~

Minikube에 대하여 더 자세한 내용은 [Installing Kubernetes with Minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/)를 참고할 것.

### Minikube Addon 활성화 시키기

Minikube의 Ingress 및 Dashboard와 같은 Addon들은 아래의 명령어로 활성화시킬 수 있다. Minikube Cluster가 생성된 후 (즉 ```minikube start``` 명령어 실행 후 ```minikube delete``` 명령어로 Cluster 삭제 전까지) 한번만 활성화 시키면 Cluster 삭제전까지 유효하다.

~~~ bash
# show entire minikube addons
minikube addons list

# enable specific addon
minikube addons enable dashboard
minikube addons enable ingress
minikube addons enable metrics-server

# check above addons are enabled properly
minikube addons list
~~~

### Application 시작하기

~~~ bash
# Move to the project root directory
cd java-guestbook

# Switch to minikube context
kubectl config use-context minikube

# Start minikube first if it is not running
minikube start

# Deploy application
skaffold dev -v info --port-forward
~~~

### Kubernetes Dashboard 실행하기

모든 Pod들이 제대로 실행되었는지 확인하기 위하여 다음의 명령어로 Dashboard로 확인한다.

~~~ bash
# Launch kubernetest dashboard
minikube dashboard
~~~

### Frontend 접근하기

Frontend Pod의 Endpoint를 확인하기 위하여 다음의 명령어를 실행한다.

~~~ bash
minikube service list
~~~

그러면 다음과 같이 현재 배포된 Pod들의 endpoint를 확인할 수 있다.

~~~ bash
|----------------------|------------------------------------|--------------|-----------------------------|
|      NAMESPACE       |                NAME                | TARGET PORT  |             URL             |
|----------------------|------------------------------------|--------------|-----------------------------|
| default              | java-guestbook-backend             | No node port |
| default              | java-guestbook-frontend            |           80 | http://192.168.64.106:31632 |
| default              | java-guestbook-mongodb             | No node port |
| default              | kubernetes                         | No node port |
| kube-system          | ingress-nginx-controller-admission | No node port |
| kube-system          | kube-dns                           | No node port |
| kube-system          | metrics-server                     | No node port |
| kubernetes-dashboard | dashboard-metrics-scraper          | No node port |
| kubernetes-dashboard | kubernetes-dashboard               | No node port |
|----------------------|------------------------------------|--------------|-----------------------------|
~~~

위에서 확인한 Endpoint 주소를 브라우저에 입력하여 정상적으로 Application이 실행되는지 확인한다.

### Application 중지하기

~~~ bash
(CTRL-C to stop running application)

# Stop minikube
minikube stop
~~~

## Next Step

실제 Production Level로 Application 변경하기.

- Redis를 이용하여 Session Clustering 구성하기
- Namespace 추가하고 Sticky Session 지원하기
- ConfigMap으로 환경 변수 관리하기
- Ingress 추가
- MongoDB를 MySQL로 변경
- PersistentVolume, PersistentVolumeClaim 추가하여 Redis 및 MySQL에 사용하기
- Job 추가하여 MySQL에 Application에서 사용할 User 추가 및 Initial Table, Data Population 처리하기
- StatefulSet을 이용하여 3 Node로 MySQL 구성하기
- Deployment에 resources 추가하여 CPU 및 Memory의 Min/Max 값 지정하기
- Frontend/Backend Pod에 HorizontalPodAutoscaler 적용하기
- skaffold.yaml 파일에 Redis/MySQL Port를 Port Forward 설정에 추가하여 클라이언트로 localhost:port로 접속하기
