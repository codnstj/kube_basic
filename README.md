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

