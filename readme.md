# K8S

## Container Orchestration


### K8S

- 스케쥴링
- 자원 할당
- Service Discorvery
- 스케일링
- 로드밸런싱
- Self Healing
- Rollback / Rollout
- 설정 관리
- 스토리지 오케스트레이션


여러 머신으로 구성된 클러스터 상에서 컨테이너를 효율적으로 관리하기 위한 시스템


### Prerequisite

- EKS는 VPC 안에서 돌아간다. 따라서 테스트용 VPC 환경을 구축해야한다.
- AWS에서 제공하는 샘플 VPC cloudformation 양식과 비슷하게 `serverless.yml` 로 작성하였다.

배포는 `sls deploy --stage dev --region ap-northeast-2 --profile your-profile` 로 배포한다.

만약 `default` profile로 배포하고싶으면 `--profile` 생략해서 배포해도 무방하며, `stage` 또한 생략해도 무방하다.

### 샘플 VPC 도식

- 작성 예정

### 쿠버네티스 클러스터 구성요소

#### 컨트롤 플레인(마스터 노드)

1개 ~ n개 (보통 홀수개)

- 클러스터 관리하는 역할
- 상태 관리 및 명령어 처리

#### 노드 (워커 노드)

- 어플리케이션 컨테이너 실행

쿠버네티스 API 서버는 컨트롤 플레인에 존재하며, 사용자는 kubectl을 이용하여 API 호출한다.

etcd, controller manager, scheduler 등 컨트롤 플레인에 있는 요소 또한 쿠버네티스 API 서버와 상호작용 한다.

워커노드의 경우 kubelet(노드 에이전트)이 컨트롤 플레인의 API 서버와 상호작용한다.

클라우드 내에서 k8s를 배포하고 사용하는 경우 엔드 유저는 Cloud의 LoadBalancer 에 요청을 보내게 되며, LoadBalancer가 적절한 워커 노드에 요청을 전달한다. (정확히는 워커노드의 Pods)

### 컨트롤 플레인 

#### API 서버

- 쿠버네티스 리소스와 클러스터 관리를 위한 API 제공
- etcd를 데이터 저장소로 사용한다.

#### 스케쥴러

- 노드의 자원 사용 상태를 관리하며 새로운 워크로드를 어디에 배포할지 관리한다.

#### 컨트롤 매니저

- 여러 컨트롤러 프로세스를 관리
- 각 컨트롤러는 클러스터로부터 특정 리소스 상태의 변화를 감지하여 클러스터에 반영하는 과정을 반복 수행

#### etcd

분산 Key - Value 저장소로 클러스터 상태 저장 (분산 key-value 저장소... Dynamo와 비슷..?)


### 노드

#### kubelet

- 컨테이너 런타임과 통신하며 컨테이너 라이프사이클 관리
- API 서버와 통신하며 노드의 리소스 관리 (리소스 보고)

#### CRI 

- kubelet이 컨테이너 런타임과 통신할 때 사용되는 인터페이스
- 쿠버네티스는 Docker, containerd, cri-o 컨테이너 런타임을 지원한다.

#### kube-proxy

- 오버레이 네트워크 구성
- 네트워크 프록시 및 내부 로드밸런서 역할 수행


### API 리소스 및 오브젝트

#### API 리소스 (오브젝트에 대한 명세. Class)

- 쿠버네티스가 관리 할 수 있는 오브젝트의 종류

- Pod, Service, ConfigMap, Secret, Node, ServiceAccount, Role

#### 오브젝트 (instance)

- API 리소스를 인스턴스화 한 것


`kubectl api-resources` 현재 쿠버네티스 클러스터가 지원하는 API 리소스 목록 출력

`kubectl explain pod` 특정 API 리소스에 대해 간단한 설명 확인


```console
~ ❯ kubectl get pod --all-namespaces                                                                       ○ minikube 23:02:50
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   coredns-64897985d-tk4mg            1/1     Running   0          6d8h
kube-system   etcd-minikube                      1/1     Running   0          6d8h
kube-system   kube-apiserver-minikube            1/1     Running   0          6d8h
kube-system   kube-controller-manager-minikube   1/1     Running   0          6d8h
kube-system   kube-proxy-vpkrc                   1/1     Running   0          6d8h
kube-system   kube-scheduler-minikube            1/1     Running   0          6d8h
kube-system   storage-provisioner                1/1     Running   0          6d8h
```

### 메니페스트 파일

쿠버네티스는 오브젝트를 yaml 기반 매니페스트 파일로 관리

```yml

apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: "nginx"
    type: "web"
  annotations:
    my-annotations1: "hello"
    my-annotations2: "world"
  spec:
    containers:
    - name: nginx
      image: nginx:latest
      ports:
      - name: http
        containerPort: 80
```

#### 메니페스트 파일에서 메타데이터 (Labels, Annotations)
Labels

- 오브젝트 식별 목적
- 검색, 분류, 필터링 목적으로 사용
- 쿠버네티스 내부 여러 기능에서 Label Selector 기능 제공

Annotations

- 식별이 아닌 다른 목적으로 사용
- 보통 쿠버네티스의 애드온이 해당 오브젝트를 어떻게 처리할지 결정하기 위한 설정 용도로 사용


### kubectl 명령형 / 선언형

aws cli vs cloudformation 과 비슷한 위치인 듯.

#### 명령형 (like aws cli)

- 수행하고자 하는 액션을 지시
- 적은 리소스에 대해 빠르게 처리 가능
- 여러 명령어를 알아야 함

#### 선언형 (like aws cloudformation / terraform / serverlesss)

- 도달하고자 하는 상태를 선언
- 코드로 관리 가능 -> GitOps
    - 변경사항에 대한 Audit (커밋 히스토라)
    - 코드리뷰를 통한 협업
- 멱등성 보장
- 많은 리소스에 대해서도 메니페스트 관리 방법에 따라 빠르게 처리 가능
- 알아야 할 명령어 수가 적음

#### 명령형 명령어

`kubectl run -i -t ubuntu ubuntu:focal bash`

ubuntu focal 이미지로 ubuntu 파드 생성 (bash 명령어)

`kubectl expose deployment grafana --type=NodePort --port=80 --target-prot=3000`

그라파나 deployment 오브젝트에 대해 NodePort 타입의 Service 오브젝트 생성 (노드에 포트 개방)

`kubectl set image deployment/frontend www=image:v2`

프론트엔드 deployment 의 www 컨테이너 이미지를 image:v2로 변경

`kubectl rollout undo deployment/frontend --to-revision=2`

프론트엔드 디플로이먼트를 리비전 2로 롤백

#### 선언형 명령어

`kubetl apply -f deployment.yaml`

deployment.yaml에 정의된 쿠버네티스 오브젝트 클러스터에 반영

`kubectl delete -f deployment.yaml`

오브젝트 제거


#### 명령형 실습

- 그라파나 deployment 오브젝트를 생성하고 서비스 오브젝트를 생성한다.

`kubectl create deployment grafana --image=grafana/grafana --port=3000`

`kubectl expose deployment grafana --image=grafana/grafana --port=3000`

`minkube service grafana`

```console
~/De/w/fastcampus-devops/3/5-k8s-start main ❯ minikube service grafana                                                23:18:46
|-----------|---------|-------------|---------------------------|
| NAMESPACE |  NAME   | TARGET PORT |            URL            |
|-----------|---------|-------------|---------------------------|
| default   | grafana |          80 | http://192.168.49.2:32757 |
|-----------|---------|-------------|---------------------------|
🏃  grafana 서비스의 터널을 시작하는 중
|-----------|---------|-------------|------------------------|
| NAMESPACE |  NAME   | TARGET PORT |          URL           |
|-----------|---------|-------------|------------------------|
| default   | grafana |             | http://127.0.0.1:57526 |
|-----------|---------|-------------|------------------------|
🎉  Opening service default/grafana in default browser...
❗  Because you are using a Docker driver on darwin, the terminal needs to be open to run it.
```


```console
~ ❯ kubectl get node -o wide                                                                               ○ minikube 23:03:48
NAME       STATUS   ROLES                  AGE    VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
minikube   Ready    control-plane,master   6d9h   v1.23.1   192.168.49.2   <none>        Ubuntu 20.04.2 LTS   5.10.76-linuxkit   docker://20.10.12
```

internal ip 가 192.168.49.2로 지정되었다. 그래서 그라파나 또한 쿠버네티스 환경 내 ip가 192.168.49.2로 ip가 바인딩 되었다 



#### 선언형 실습

`./minikube/deployment.yaml` 파일과 `./minikube/pod.yaml` 파일을 apply 함으로서 사용.

```console
~/De/w/fastcampus-devops/3/5-k8s-start main ❯ kubectl apply -f deployment.yaml                             ○ minikube 23:23:26
deployment.apps/grafana created

~/De/w/fastcampus-devops/3/5-k8s-start main ❯ kubectl apply -f service.yaml                                ○ minikube 23:23:42
service/grafana created

~/De/w/fastcampus-devops/3/5-k8s-start main ❯ kubectl apply -f service.yaml                                ○ minikube 23:23:47

~/De/w/fastcampus-devops/3/5-k8s-start main ❯ minikube service grafana                                          х INT 23:23:49
|-----------|---------|-------------|---------------------------|
| NAMESPACE |  NAME   | TARGET PORT |            URL            |
|-----------|---------|-------------|---------------------------|
| default   | grafana | http/80     | http://192.168.49.2:31494 |
|-----------|---------|-------------|---------------------------|
🏃  grafana 서비스의 터널을 시작하는 중
|-----------|---------|-------------|------------------------|
| NAMESPACE |  NAME   | TARGET PORT |          URL           |
|-----------|---------|-------------|------------------------|
| default   | grafana |             | http://127.0.0.1:57690 |
|-----------|---------|-------------|------------------------|
🎉  Opening service default/grafana in default browser...
❗  Because you are using a Docker driver on darwin, the terminal needs to be open to run it.
```

