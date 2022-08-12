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
