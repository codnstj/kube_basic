# Kube start with @subicura

### POD create Analysis

> Pod 구성도

![pod](https://subicura.com/k8s/build/imgs/guide/pod/pod-single.png)
- pod 안에 단일 혹은 다중 컨테이너가 존재합니다. 그리고 POD 은 클러스터로 감싸져있습니다.

> `kubectl run` 을 실행하고 POD 이 생성되는 과정입니다.

<img width="606" alt="Screen Shot 2022-08-10 at 5 17 27 PM" src="https://user-images.githubusercontent.com/69895368/183851582-23300171-1f7b-4f7b-b005-cca9e11a62e9.png">

### Container Status Monitoring
 - `컨테이너 생성` 과 실제 `서비스 준비` 는 약간의 차이가 있다. 서버를 시랳ㅇ하면 바로 접속할수 없고 짧게는 수초, 기렉는 수분의 초기화 시간이 필요한데 실제로 접속이 가능할때 `서비스가 준비되었다.` 라고 말할수 있습니다.

![container-monitor](https://subicura.com/k8s/build/imgs/guide/pod/pod-monitoring.png)

쿠버네티스는 컨테이너가 생성되고 서비스가 준비되었다는것을 체크하는 옵션을 제공하여 초기화하는 동안 서비스되는것을 막을수 있습니다.

- LivenessProbe
	- 컨테이너가 정상적으로 동작하는지 체크하고 정상적으로 동작하지 않는다면 컨테이너를 재시작하여 문제를 해결합니다.

	- `정상` 이라는 것은 여러 가지 방식으로 체크할 수 있는데 저는  `http get` 요청을 보내 확인하는 방법을 사용했습니다.
	> httpGet 이외에 `tcpSocket`,`exec` 방법으로 체크할 수 있습니다.
	
- readinessProbe
	- 컨테이너가 준비되었는지 체크하고 정상적으로 준비되지 않았다면 POD 으로 들어오는 요청을 제외합니다.

	- LivenessProbe 와 차이점은 문제가 있어도 Pod 을 재시작하지 않고 요청만 제외한다는 점입니다

### Multi Container

- 대부분 `1 pod = 1 container` 지만 여러 개의 컨테이너를 가진 경우도 흔하다.

- 하나의 Pod 에 속한 컨테이너는 서로 네트워크를 localhost 로 공유하고 동일한 디렉토리를 공유할수도 있습니다.

> 다중 컨테이너 POD 구성도 
![Multi-container](https://subicura.com/k8s/build/imgs/guide/pod/pod-multi.png)

### ReplicaSet

- pod 을 단독으로 만들면 pod 에 어떤 문제가 생겼을떄 자동으로 복구되지 않습니다. 이러한 pod 을 정해진 수만큼 복제하고 관리하는 것이 ReplicaSet 입니다.

- Kind 를 ReplicaSet 으로 지정해주면 됩니다.

> ReplicaSet 구성도 
![rs](https://subicura.com/k8s/build/imgs/guide/replicaset/rs.png)

 - ReplicaSet 은 `labels을 체크` 해서 원하는수의 Pod 이 없으면 새로운 Pod 을 생성합니다. 이를 설정으로 표현하면 다음과 같습니다.

 |정의|설명|
 |--|--|
 |`spec.selector`|label의 체크조건|
 |`sepc.replicas`|원하는 Pod의 개수|
 |`spec.template`|생성할 Pod의 명세|
	 
> ReplicaSet 동작 원리

<img width="606" alt="Screen Shot 2022-08-10 at 5 17 27 PM" src="https://user-images.githubusercontent.com/69895368/184362467-e53882b6-6410-405f-98c6-e7753220881e.png">

- Scaleout
	- ReplicaSet 을 이용하면 손쉽게 Pod 을 여러개로 복제할수 있습니다.
		- spec: replicas:n
### Deployment

- Deployment 는 쿠버네티스에서 가장 널리 사용되는 오브젝트 이다. Replicset 을 이용하여 Pod 을 업데이트하고 이력을 관리하여 롤백하거나 특정버전으로 돌아갈수 있습니다.
	> Pod 업데이트 
	> 엄밀히 말하면 Pod 을 새로운 버전으로 업데이트 한다는 잘못된 표현이고 새로운 버전의 Pod 을 생성하고 기존의 Pod 을 제고한다가 가장 정확한 표현 입니다.
- Deployment Replicaset Update
	- Deployment 는 새로운 이미지를 업데이트하기 위해 ReplicaSet 을 이용합니다. 버전을 업데이트하면 새로운 Replicaset 을 생성하고 해당 ReplicaSet 이 새로운 버전의 Pod 을 생성합니다.
	- ReplicaSet 을 업데이트 하는 과정은 Rolling 방식으로 배포됩니다.


	<img width="1012" alt="Screen Shot 2022-08-13 at 12 01 32 AM" src="https://user-images.githubusercontent.com/69895368/184383550-f1c39202-1fd4-4be1-9d7a-e8b06d7d2a5f.png">
	
	- 각 컨트롤러는 다음과 같이 동작합니다.
	
	<img width="643" alt="Screen Shot 2022-08-13 at 12 03 10 AM" src="https://user-images.githubusercontent.com/69895368/184383718-1f2a5d31-e868-4c96-b949-c298f5818c5e.png">
	
- Version 관리
	- `kubectl rollout history` 를 통해 Deployment 의 버전 을 확인할수 있습니다.
	- `kubectl rollout undo` 를 통해 Deployment 의 특정 버전으로 rollback 할수 있습니다.
<img width="600" alt="Screen Shot 2022-08-13 at 12 05 21 AM" src="https://user-images.githubusercontent.com/69895368/184384154-394b7d3d-5e8f-4ea9-bfd4-d09c9dcac39a.png">

- Strategy 설정
<img width="187" alt="Screen Shot 2022-08-13 at 12 17 07 AM" src="https://user-images.githubusercontent.com/69895368/184386341-ff3f8509-09b1-4dd6-b930-b050dd83096e.png">

- rolling 배포 형식으로 3개씩 POD 들이 생성됩니다.
<img width="960" alt="Screen Shot 2022-08-13 at 12 16 59 AM" src="https://user-images.githubusercontent.com/69895368/184386500-5cd08a88-5f57-4048-b9b6-2746460d3584.png">


### Service

- Pod 은 자체 IP 를 가지고 다른 Pod 과 통신할 수 있지만, 쉽게 사라지고 생성되는 특징 때문에 직접 통신하는 방법은 권장하지 않습니다. 쿠버네티스는 Pod 과 직접 통신하는 방법대신, 별도의 고정된 IP 를 가진 서비스를 만들고 그 서비스를 통해 Pod 에 접근하는 방식을 사용합니다.

- ClusterIP
	- ClusterIP 는 클러스터 내부에 새로운 IP 를 할당하고 여러 개의 POD 을 바라보는 로드밸런서 기능을 제공합니다. 그리고 서비스 이름을 내부 도메인 서버에 등록하여 Pod 간에 서비스 이름으로 통신할수 있습니다.
	> ### 구분자 
	> 하나의 YAML 파일에 여러 개의 리소스를 정의할 떈 "---"를 구분자로 사용합니다.

- ***Example***

	- redis Deployment 와 Service 생성 (./service/counter-redis-svc.yaml) 
	![redis-service](https://subicura.com/k8s/build/imgs/guide/service/clusterip.png)

	- 같은 클러스터에 생성된 Pod 이라면 `redis` 라는 도메인으로 redis Pod에 접근할 수 있습니다.
	(`redis.default.svc.cluster.local` 로도 접근이 가능합니다. 서로 다른 namespace 와 cluster 를 구분할수 있습니다.)

	- ClusterIP 서비스 설정
		|정의|설명|
		|--|--|
		|`spec.ports.port`|서비스가 생성할 Port|
		|`spec.ports.targetPort`|서비스가 접근할 Pod 의 Port (Default: port 랑 동일)
		|`spec.selector`| 서비스가 접근할 Pod 의 Label 조건|

		redis Service 의 Selector 는 redis Deployment에 정의한 Label 을 사용했기 때문에 해당 Pod 을 가리킵니다. 그리고 해당 Pod 의 6379 포트의 연결하였습니다.

	- 앱에서 도메인으로 Redis ClusterIP 연결
		- `spec.containers.env.REDIS_HOST` 를 `"redis"` 라는 도메인 이름을 사용하여 연결하였습니다.
	
- Service 생성 흐름
	-  Service는 각 Pod를 바라보는 로드밸런서 역할을 하면서 내부 도메인서버에 새로운 도메인을 생성합니다. 
	
	<img width="608" alt="Screen Shot 2022-08-16 at 5 08 20 PM" src="https://user-images.githubusercontent.com/69895368/184833214-7dffad9c-d0e8-4931-9b96-a4644f0733c2.png">


	- iptables
		- iptables 는 커널 레벨의 네트워크 도구이고 `COREDNS` 는 빠르고 편리하게 사용할 수 있는 클러스터 내부용 도메인 네임 서버 입니다. 각각의 역할은 `iptables` 설정으로 여러 IP에 트래픽을 전달하고 `COREDNS` 를 이용하여 IP 대신 도메인 이름을 사용합니다.
		> iptables 는 규칙이 많아지면 성능이 느려지는 이슈가 있어, ipvs 를 사용하는 옵션도 있습니다.

		> COREDNS 는 클러스터에서 호환성을 위해 `kube-dns` 라는 이름으로 생성됩니다.

### NodePort 만들기

- ClusterIP 는 클러스터 내부에서만 접근할 수 있습니다. 클러스터 외부(노드)에서 접근할수 있도록 NodePort 서비스를 생성해줍니다.

- Example

	<img width="211" alt="Screen Shot 2022-08-16 at 5 49 04 PM" src="https://user-images.githubusercontent.com/69895368/184842122-6a8dba3b-a10e-4857-b94c-68fd56716fc4.png">

	
	
	|정의|설명|
	|--|--|
	|`spec.ports.nodePort`|노드에 오픈할 Port(미지정시 30000 ~ 32768 중 자동할당)|

	> Docker driver 를 사용중이라면 minikube service <서비스명> 명령어를 이용해서 접속해보자.

	![node1](https://subicura.com/k8s/build/imgs/guide/service/nodeport.png)

	- NodePort는 클러스터의 모든 노드에 포트를 오픈합니다. 지금은 테스트라서 하나의 노드밖에 없지만 여러 개의 노드가 있다면 아무 노드로 접근해도 지정한 Pod으로 쏘옥 접근할 수 있습니다.

	![node2](https://subicura.com/k8s/build/imgs/guide/service/nodeport-multi.png)

	> NodePort 는 기본적으로 ClusterIP 기능을 기본적으로 포함합니다.

### LoadBalancer

- nodeport 의 단점은 노드가 사라졌을 떄 자동으로 다른 노드를 통해 접근이 불가능하다는 점입니다. 예를 들어, 3개의 노드가 있다면 3개중에 아무 노드로 접근해도 NodePort 로 연결할 수 있지만 어떤 노드가 살아 있는지는 알수가 없습니다.

![lb1](https://subicura.com/k8s/build/imgs/guide/service/lb.png)

- 자동으로 살아 있는 노드에 접근하기 위해 모든 노드를 바라보는 `LoadBalancer` 가 필요합니다. 브라우저는 NodePort에 직접 요청을 보내는것이 아니라 LoadBalancer 에 요청하고 LoadBalancer 가 알아서 살아있는 노드에 접근하면 NodePort 의 단점을 없앨 수 있습니다.

- Example 

	<img width="664" alt="Screen Shot 2022-08-17 at 1 55 16 AM" src="https://user-images.githubusercontent.com/69895368/184938107-d3e5de68-78d2-4549-b132-696b76283ff9.png">

	- counterlb가 생성되었지만, EXTERNAL-IP가 <pending>인 것을 확인할 수 있습니다. 사실 Load Balancer는 AWS, Google Cloud, Azure 같은 클라우드 환경이 아니면 사용이 제한적입니다. 특정 서버(노드)를 가리키는 무언가(Load Balancer)가 필요한데 이런 무언가가 가상머신이나 로컬 서버에는 존재하지 않습니다.
	- `minikube addons enable metallb` 를 통해 minikube 에 가상 로드밸런서를 생성해주겠습니다.
	- `minikube addons configure metallb` 에 Start IP 와 End IP 모두 `minkube ip` 에서 나온 아이피를 등록해주면 됩니다.

	<img width="670" alt="Screen Shot 2022-08-17 at 2 06 12 AM" src="https://user-images.githubusercontent.com/69895368/184938140-4be3ec75-6ca0-492c-b17f-9d19d0bdc40c.png">


> `metallb` 는 로드밸런서를 사용할수 없는 환경에서 가상환경을 만들어 주는 것이 `metallb` 라는 것입니다. minikube 에서는 현재 떠있는 노드를 LoadBalancer 로 설정합니다.

### Ingress

- 하나의 클러스터 에서 여러가지 서비스를 운영한다면 외부연결할시  NodePort 를 이용하면 서비스 개수만큼 포트를 오픈하고 사용자에게 어떤 포트인지 알려줘야합니다.

- minikube ingess 활성화 시키기
	```shell
	minikube addons enable ingress 

	#ingress 컨트롤러 확인할
	kubectl -n ingress-nginx get pod
	```
- ingress 생성 흐름
	
	동적방식을 보면 YAML로 만든 INGRESS 설정을 단순히 nginx 설정으로 바꾸는 걸 알 수 있습니다. 이러한 과정을 수동으로 하지 않고 Ingress Controller가 하는 것 뿐입니다.

	Ingress는 도메인, 경로만 연동하는 것이 아니라 요청 timeout, 요청 max size 등 다양한 프록시 서버 설정을 할 수 있습니다.
