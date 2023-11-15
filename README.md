패스트 캠퍼스 [백엔드 개발자를 위한 Kubernetes :클라우드 네이티브 프로그래밍](https://fastcampus.co.kr/dev_online_k8s) 강의를 보고 정리한 repository 입니다.

# 쿠버네티스 - BE

### 하드웨어 레벨의 가상머신이란?

말 그대로, 컴퓨터와 본체와 같은 하드웨어를 가상화 시킨 것을 의미한다.

즉, 사용자는 실제로 하드웨어를 사용하고 있다고 느끼겠지만 CPU 나 Memory 가 소프트웨어적으로 처리되고 있음을 의미한다.

### 운영체제 레벨의 가상화 머신이란?

컨테이너. 실제 운영체제가 존재하며, 그 운영체제 내부에 가상의 운영체제를 만들어주는 기술로 컨테이너 내부에 존재하는 프로세스들 입장에서는 진짜 운영체제라고 믿으면서 동작을 하게 되는 것을 의미한다.

**Control Plane (=Master Node)**

- API Server : 사용자의 명령을 받아서 상태를 저장하고, 노드(=워커노드) 와 통신하며 필요한 동작들을 수행
- etcd : 쿠버네티스 클러스터 및 컨테이너와 관련된 모든 데이터를 저장하는 데이터 저장소
- Scheduler : 컨테이너가 생성될 때 **적합한 노드를 선택**해주는 역할을 수행
    
    → 모든 노드가 균등한 부하를 받을 만할 정도로 분배를 한다 정도로만 이해하자.
    
- Controller Manager : 쿠버네티스의 다양한 컨트롤러들을 관리하며, API Server 가 해야할 일을 알려주는 역할
- Cloud Controller Manager : 다양한 클라우드 환경과 쿠버네티스를 통합해주는 역할 
→ AWS 에서 GCP 로 변경한다고 해서 클라우드 컨트롤러 매니저가 일부분 설정을 지원함.

**Node (=Worker Node, 컨테이너가 실제로 실행되는 공간)**

- Kubelet : 컨테이너를 실행하거나 종료하고, 상태를 모니터링해주는 역할
→ 컨테이너의 실행과 종료를 큐블렛이 자체적으로 결정하는 것은 아니고, API Server 와 상호작용 한다.
- kube-proxy : 컨테이너의 네트워크를 관리해주는 역할

---

**사용자의 컨테이너 생성 요청 시, 쿠버네티스 동작 과정**

1. 사용의 요청
2. API Server 는 사용자의 요청을 검토 (인증, 인가, 문법적 오류 등등)
3. API Server 는 etcd 에 전달받은 스펙을 저장한다.
4. Scheduler 는 적절한 노드를 선택한다.
5. API Server 를 통해서 kubelet 은 컨테이너를 실행하게 된다.
6. kubelet 은 컨테이너를 실행 하면서 발생하는 여러가지 모니터링 작업을 API 서버에 전달하고, 모니터링 내용들을 etcd 에 저장한다.
7. 마지막으로 API Server 를 통해 kube-prox 가 네트워크 세팅(라우팅 등)을 한다.

---

### 쿠버네티스의 명령어 수행 방식 (명령형 vs 선언형)

**Reconciliation pattern (레컨실레이션 패턴)**
<img width="697" alt="스크린샷 2023-11-12 오후 11 01 14" src="https://github.com/kihyuk-jeong/k8s/assets/39195377/089f2f6c-eed1-4cf4-bb18-6e7c6d0cf40f">

결론부터 말하면 쿠버네티스는 선언형 시스템이다. (Declarative System)

쿠버네티스의 명령어 수행 방식으로,  프로그램에 직접 명령을 주는 것이 아니라 목표로 하는 상태를 제시해주는 방식이다. 즉, 그 상태에 도달하기 위한 방법은 프로그램 (쿠버네티스) 에 위임한다.

- 컨테이너를 2개 생성해라 (Imperative)
→ 컨테이너가 현재 몇 개 생성되어 있든 상관하지 말고 2개를 추가로 생성해라.
- 컨테이너가 2개 있어야 한다 (Declarative)
→ 지금 컨테이너가 없으면 2개 생성, 1개만 있다면 1개 생성, 3개라면 1개 종료

(참고)imperative System : 명령을 통해 원하는 작업을 수행하는 방식으로, 대부부느이 프로그램과 서비스가 이 방식으로 수행된다.

**Self-healing**

- 쿠버네티스의 선언적 특징에서 나타나는 기능
- 컨테이너의 상태가 목표에서 벗어난다면 이를 자동으로 수정
- 프로세스가 종료 또는 컨테이너가 비정상이라면 새로운 컨테이너로 교체
- 쿠버네티스는 목표상태와 실제 상태가 일치하지 않으면 무한히 목표상태로 변경을 시도한다.
- 운영 환경에서 활용하면 굉장히 강력한 기능!

**컨테이너 재시작 (in k8s)**

- 컨테이너를 재시작 하는 명령은 쿠버네티스에서 조금 애매하다.
- 선언적으로 컨테이너를 재시작 한다는 개념 자체가 정의되기 어렵기 때문이다.
    - 목표로 하는 컨테이너 수는 1개, 현재 떠 있는 컨테이너도 1개.
    - 만약 재시작을 하고 싶다면 어떻게 해야할까?
- k8s 의 아이덴티티 자체는 선언형이지만 명령형 커맨드도 지원을 하기 때문에 재시작 가능.
    - 재시작 명령어를 보내기 보다는, 컨테이너 자체를 제거해버린다.
    - 쿠버네티스는 목표 상태를 유지하기 위해 다시 컨테이너를 띄울 것.
- 만약 명령형을 쓰기가 싫다면, 목표 컨테이너 수를 0으로 변경했다가 다시 1개로 변경

**Reconciliation for Application Developer**

- Declarative : 모든 동작이 선언적으로 이루어짐
- Idempotency : 같은 정의를 여러 번 적용해도 항상 같은 결과가 나옴
- Self-Healing : 현재 상태에 예상하지 못한 불일치가 생겨도 스스로 복구
- Consistency : 쿠버네티스의 현재 상태나 동작과 관계 없이 최종 스펙에 상태를 맞춤

---

### Selector 와 Label

- Label : 각 컨테이너에 라벨 (택) 을 붙힌다고 생각하면 된다.
    
    ```java
    labels:
    	app: my-sapp
    	type: backend
    	available : true
    ```
    
- selector: 붙힌 라벨을 기준으로, 사용 할 컨테이너를 선택한다.
    
    ```java
    selector:
    	app: my-app
    	available: true
    ```
    
    - app: my-app 이고 available 이 true 인 label 이 붙은 컨테이너를 선택한다.
    - available 이든 app 이든 type 이든 고정적으로 정해져 있는 것이 아닌 개발자가 임의로 지정할 수 있는 key-value 다.

**쿠버네티스의 추상화 컨셉**

- 구체적이고 실질적이기 보다는 추상적인 개념 위주로 정의가 이루어짐.
- 어떤 특정 대상보다는 동일한 역할을 수행하는 미상의 집단을 대상으로 지정하는 느낌
- 인스턴스를 다룬다기 보다는 클래스를 다루는 느낌
- 스펙을 잘 정의해 놓으면 개발자가 딱히 할 게 없는 느낌

**쿠버네티스의 네트워크**
<img width="662" alt="스크린샷 2023-11-12 오후 11 01 58" src="https://github.com/kihyuk-jeong/k8s/assets/39195377/bba2e787-d926-44af-84aa-0ce6157208f5">


쿠버네티스는 기본적으로 **네트워크 오브젝트**를 한 번 거친다.

- 예를 들어 my-app 컨테이너에서 다른 컨테이너로 요청을 보내려고 하면 my-app-inbound-network 를 거쳐서 부하 분산이 진행되며 요청이 넘어간다.
- 내부 통신의 경우 (internal API)
    - your-app 에서 http://my-app-network/api/check  요청
    - my-app-network 는 내부에 가지고있는 라우팅 테이블을 기반으로 5개의 My-app 컨테이너 중 하나로 요청을 전달함.
- 외부 통신의 경우 (public API)
    - 외부 트래픽이 들어오면, 외부 트래픽을 처리하기 위한 객체가 먼저 트래픽을 받고 네트워크 오브젝트로 전달된 이후 적절한 컨테이너로 트래픽을 전달한다. (in-bound 규칙)
    

### 저장소와 볼륨

- my-app 이라는 컨테이너의 로그를 쌓기 위해 컨테이너 외부에 볼륨을 생성해서 (in 노드) 컨테이너가 의도치 않게 종료된 이후에도 노드에 저장된 로그를 조회할 수 있다
→ BUT, 컨테이너는 내려갔다가 다시 올라오는 사이클이 반복되며 그 때 마다 동일한 노드에서 실행된다는 보장이 없다.

**해결책 (노드 외부에 위치시켜야 한다.)**
<img width="652" alt="스크린샷 2023-11-12 오후 11 02 26" src="https://github.com/kihyuk-jeong/k8s/assets/39195377/47b4720b-a8ce-441d-87dd-9e267deaf293">

기본적으로 쿠버네티스에서 볼륨은 **임시볼륨 과 영구볼륨이 존재한다.**

- 임시볼륨 : 일반적인 컨테이너 저장소와 동일하게, 컨테이너가 종료되면 사라지는 저장 공간
(컨테이너와 라이프사이클이 동일하고, 지워져도 상관 없는 파일을 저장할 때 사용)
- 영구 볼륨 : 저장공간이 노드 외부에 위치해서 어떤 경우에도 일관성 있게 참조할 수 있는 저장소

---

### Pod

쿠버네티스가 사용하는 가장 기본적인 배포 단위 

- 쿠버네티스에서 가장 작은 단위의 용어이다.
- 즉, pod 보다 작은 단위는 배포가 불가능하다는 뜻.   
  <img width="457" alt="스크린샷 2023-11-12 오후 11 03 00" src="https://github.com/kihyuk-jeong/k8s/assets/39195377/f99d1a10-4b98-44a1-9b21-73eb92dc8fc9">
    
- 파드의 구조를 보면, 파드 내부에 여러 개의 컨테이너가 존재하고 있으며, 컨테이너 독자적으로는 배포가 불가능하다.
- 같은 파드 내에 존재하는 컨테이너는 서로 다른 포트를 가져야 한다.
- 일반적으로는 1파드 = 1컨테이너로 구성하는 경우가 많으며, 파드 = 컨테이너 의 개념은 아니지만 거의 그렇게 생각해도 된다.
    
 <img width="643" alt="스크린샷 2023-11-12 오후 11 03 23" src="https://github.com/kihyuk-jeong/k8s/assets/39195377/cfce23ca-746a-48ed-ad9e-69dbbb888457">

**파드의 라이프 사이클**

- Pending : 파드가 노드에 배치되는 단계 (아직 실행되지 않은 상태. 일반적으로 길지 않기 때문에 실제로 관찰하기 어렵지만, 이미지 유실이나 용량 부족과 같은 예외 상황에서 주로 관찰된다.)
- Running : 파드가 실행 중인 단계
    - 파드의 상태가 Running 이라고 해도 무조건 정상적인 상태라고 볼 수는 없다.
    - 스프링 애플리케이션 컨테이너가 뜬 상태에서, 예를들어 내부 디비 연결 정보가 잘못된 경우 컨테이너는 정상적인 실행 상태가 아니지만 파드는 정상 러닝 상태로 보여진다.
    - 즉, 파드의 상태를 모니터링 해야한다.
    - 파드에 여러 컨테이너가 존재하는 경우, 하나 이상의 컨테이너가 실행중인 경우에도 해당 상태임
- Failed : 컨테이너가 비정상 종료된 경우
    - 컨테이너의 재시작 정책으로 인해 해당 상태가 자주 관찰 되지는 않음
    - Always : 항상 종료된 컨테이너를 재시작 함
    - OnFailure : 실패한 컨테이너를 재시작 함
    - Never : 컨테이너를 재시작 하지 않음
- Succeeded : 컨테이너가 정상적으로 종료된 상태
- Unknown : 파드의 상태를 알 수 없는 경우

**컨테이너의 상태**

쿠버네티스 입장에서는 파드가 관리의 주체이기 때문에, 컨테이너의 상태는 비교적 단순하다.

- Waiting : 대기
- Running : 실행
- Terminated : 종료

**Probe**

파드의 상태를 모니터링 하기 위한 경로

<img width="442" alt="스크린샷 2023-11-12 오후 11 03 58" src="https://github.com/kihyuk-jeong/k8s/assets/39195377/d91c81c5-7147-4487-956e-0937f81a6ffd">


- readiness : 파드를 네트워크에 연결할 지 결정
    - 만약 readiness probe 체크에 성공을 하면 해당 파드는 네트워크 연결이 가능한 상태로 인지
- liveness : 파드를 재시작 할 지 결정
    - 컨테이너 자체가 정상적인 상태인지 체크. 만약 비정상인 경우 파드를 재시작
- startup : 프로브를 보내도 될 지 결정
    - 즉, 먼저 startup 체크를 하고 레디니스와 라이브니스 체크를 한다.
    

**Graceful shutdown**

파드를 종료할 때 주는 유예기간으로, 파드를 바로 종료하는 것이 아닌 네트워크를 중단하면서 신규 요청을 받지 않고 기존에 받은 요청들을 처리한 후 우아하게 종료하는 방법

- 기본 설정의 경우 30초의 유예기간을 준다.

### Multi Container Pod

파드 내의 컨테이너들은 같은 환경을 공유한다.

**멀티 컨테이너의 필요성**

- 쿠버네티스의 파드는 애플리케이션의 단위와 비슷하다. (대부분의 애플리케이션은 여러 개의 프로세스로 동작한다.)
- 하나의 컨테이너는 하나의 프로세스를 가져야 한다.
    - 기술적으로 하나의 컨테이너가 2개 이상의 프로세스를 가질 수 있다.
    - but, 컨테이너 관리 측면에서 어려움이 발생한다. (2개의 프로세스 중 1개는 죽고 1개는 살아있을 때)

<img width="288" alt="스크린샷 2023-11-12 오후 11 04 33" src="https://github.com/kihyuk-jeong/k8s/assets/39195377/a0fb1e0e-1871-4c78-9caa-8f9819ce4e57">


- 위와 같이 하나의 파드에 여러 컨테이너를 구성하는 경우 컨테이너들의 라이프사이클이 동일한지 살펴보아야 한다.
    - 동일 파드 내 컨테이너 A 가 종료 되었을 때, 컨테이너 B 의 실행이 의미가 있는가 ?
    - 스키엘링 요구사항이 동일한가? (컨테이너 A 가 10개로 증설될 때, B 도 함께 증설 될 필요가 있는가?)
- 멀티 컨테이너를 구성할 때 재시작 정책은 아래와 같이 설정한다.
    - Always : 종료된 컨테이너를 항상 재시작 하며, 모든 컨테이너가 상시 실행되어야 할 때
    - OnFailure : 실패 컨테이너만 재시작 하며, 파드 내 특정 컨테이너는 작업 후 종료가 가능할 때
    - Never : 종료된 (실패 포함) 컨테이너를 재시작 하지 않으며 파드 종료 상태를 보고 별도로 처리할 때

**init Container**

파드를 멀티 컨테이너로 구성할 때 가장 먼저 실행되어 파드의 초기화를 담당하는 컨테이너

- init Container 는 어떠한 작업을 하고 반드시 종료가 되어야 함
- 쿠버네티스는 init Container 가 정상적으로 종료되지 않으면 다음 작업을 수행하지 않는다.

### Scheduler

쿠버네티스는 여러가지 이유로 인해서 파드가 신규 노드에 배치가 되어야 한다면, 스케쥴러에 의해 배치될 노드가 결정된다.

1. 필터링 : 파드를 배치할 수 없는 노드를 거르는 과정
    - 파드를 배치할 수 없는 상태는 여러가지가 있는데, 디스크가 가득 찬 노드나 패치와 같은 이슈로 사용이 불가능한 노드를 걸러낸다.
  
   
<img width="687" alt="스크린샷 2023-11-14 오후 11 07 15" src="https://github.com/kihyuk-jeong/k8s/assets/39195377/c2549fb2-9620-408e-89d9-0fbac723def3">



2.스코어링 : 파드를 노드에 배치하고 나면 남은 자원이 어느정도일지, 또는 이미 동일한 파드가 특정 노드에 배치되어있는지 등등 여러 요소를 고려하여 스코어를 매겨서 가장 적합한 노드를 결정한다.
→ 결정이 완료되면 kubelet 에 의해 노드 내부에 파드가 생성된다.

<img width="681" alt="스크린샷 2023-11-14 오후 11 07 29" src="https://github.com/kihyuk-jeong/k8s/assets/39195377/4ec66f6f-1498-4c3a-a0b5-2348c5263d1b">


-> 쿠버네티스에 의해 배치된 파드는 대부분 최적의 노드에 배치되었을 확률이 높으며, 설령 최적이 아니더라도 파드가 제거/생성을 반복하면서 최적의 노드를 찾아가기 때문에 개발자의 입장에서 크게 신경쓸 부분이 많지 않다.

**Node Selector**

파드에 배치될 노드를 Selector 를 이용하여 직접 결정할 수 있다.

- 모든 조건을 만족하는 경우에만 배치한다.

```yaml
spec:
	nodeSelector:
		hw-type: gpu
		cost: high
```

- 이 경우 hw-type 이 gpu 이고 cost 가 high 를 만족하는 Selector 가 존재하는 경우에만 배치를 한다.
- 만약 모든 조건 중 하나라도 일치하지 않는다면 파드는 pending 상태로 만족하는 조건이 생길 때 까지 대기한다.



**Taint and Tolerations**

**Taint**

- NoSchedule : 파드가 스케줄 되지 않음 (즉, 파드가 배치되지 않음)
- PreferNoSchedule : 되도록 파드를 배치하지 않음
- NoExecute : 이미 있는 파드도 모두 제거함

**Tolerations**

**Taint** 옵션으로 인해 죽은 Node 가 되어버리는 것을 방지하기 위해, Pod 가 **Taint** 를 무시할 수 있는 옵션이다.

- 즉, **Taint 옵션이 NoSchedule 이라면,** 기본적으로 파드를 배치하지 않되, **Tolerations** 스펙에 부합하는 경우에는 파드를 배치 하라는 선언

```yaml
spec:
	tolerations:
	-	key: "hw-type"
		operator: "Equal"
		value: "gpu"
		effect: "NoSchedule"
```

- 모든 tolerations 조건이 만족해야 Taint 를 무시하고 파드의 배치가 가능하다.

**Pod Disruption Budget (PDB)**

자발적인 종료 상황에서도 유지할 파드 수를 지정

```yaml
sepc:
	minAvailable: 2
```

**Node Affinity**

노드의 선택을 조금 더 구체적으로 지정하는 기능 
→ (총 2개가 있는데 이름 자체가 매우 길다..)
→ 하지만 자세히 보면 앞 단어만 다르다! (required, preferred)

- requiredduringscheduling**Ignoredduringexecution**
    
    ```yaml
    spec:
    	affinity:
    		nodeAffinity:
    			requiredduringscheduling**Ignoredduringexecution:
    				nodeSelectorTerms:
    				- matchExpression:
    					-key: hw-type
    						operator: In
    						values:
    						- gpu**
    ```
    
    - requiredduringscheduling : 반드시 모든 조건을 만족해야 한다.
    - Ignoredduringexecution **:**  단, 이미 실행중인 파드는 제외한다.
    → 즉, 이미 실행중인 파드는 그대로 냅두고 신규로 생성되는 파드는 아래 조건을 지키며 워커노드에 배치가 되어야 한다.
    - 참고로 **matchExpression** 는 여러 개 존재할 수 있는데 이는 or 조건으로 여러 조건 중 하나만 일치해도 된다.
- preferredduringscheduling**Ignoredduringexecution**
    
    ```yaml
    spec:
    	affinity:
    		nodeAffinity:
    			preferredduringscheduling**Ignoredduringexecution:
    			- weight: 10
    				perference:
    				- matchExpression:
    					-key: hw-type
    						operator: In
    						values:
    						- gpu
    			- weight: 20
    				perference:
    				- matchExpression:
    					-key: sw-type
    						operator: In
    						values:
    						- temp**
    ```
    
    - preferredduringscheduling : 선호하는 조건의 가중치를 두고 가장 높은 선호도를 가진 조건에 해당하는 노드에 우선적으로 파드를 배치한다.
    - Ignoredduringexecution **:**  단, 이미 실행중인 파드는 제외한다.
    → 즉, 이미 실행중인 파드는 그대로 냅두고 신규로 생성되는 파드는 아래 조건을 지키며 워커노드에 배치가 되어야 한다.

**PodAffinity, PodAntiAffinity**

파드를 **`이런 노드에 배치하고 싶다 또는 이런 노드는 피했으면 좋겠다.` 를 지정**

```yaml
spec:
	affinity:
		podAntiAffinity:
			preferredDuringSchedulingIgnoredDuringExecution:
			- weight: 100
				podAffinityTerm:
					labelSelector:
						matchExpressions:
							- key : type
								operator: In
								values:
								- frontend
					topologyKey: "kubernetes.io/hostname"
```

위 파일을 해석해보면, 해당 파드가 배치되지 않았으면 하는 노드에 가중치를 부여하고 matchExpressions 조건에 해당하는 노드에는 **최대한** 배치하지 않는 정책이다.
→ type 이 frontend  인 파드가 많이 배치된 노드는 피해주세요!

---

### ReplicaSet and Deployment

```yaml
$ kubectl apply -f my-simple-pod.yaml
-> pod/my-simple-pod created
$ kubectl apply -f my-simple-pod.yaml
-> pod/my-simple-pod unchanged
$ kubectl apply -f my-simple-pod.yaml
-> pod/my-simple-pod unchanged
```

→ 첫 번째 명령어 이후에는 아무리 동일한 명령어를 실행해도 파드의 개수가 추가되거나 하지 않는다.

- 만약 스펙이 바뀌었다면, 바뀐 스펙으로 적용이 된다.
- 쿠버네티스가 파드를 식별하는 방법은 metadata 의 name 값으로, 이 값을 다르게 설정해서 여러 번 apply 명령어를 실행하는 경우 동일한 파드를 추가할 수 있다.
    
    → but, 관리가 너무 어려워진다. 무엇보다도 파드 수를 늘리기 위해 spec 을 계속해서 만들어야 하고, 제거할 때도 스펙 하나하나를 관리해야 한다.
    

**그래서 제공하는 ReplicaSet**

여러 파드의 복제본을 생성하고 관리할 수 있는 객체

```yaml
appVersion: apps/v1
kind: ReplicaSet
metadata:
	name: nginx-replicaset
spec:
	replicas: 3
	selector:
		matchLabels:
			app: nginx
	template:
		metadata:
			labels:
				app: nginx
		spec:
			containers:
			- name: nginx
				image: nginx:1.17
```

- template 아래에 있는 metadata 와 spec 은 replicaSet 에 의해 만들어지는 파드의 스펙이다.
- replicaSet 의 아쉬운 점 중 하나는 image 의 버전을 관리해주지 않는다는 점이다.
    - 예를 들어 nginx:1.17 을 1.18 버전으로 변경해도, replicaSet 은 파드의 개수만 보기 때문에 버전까지 관리해주지는 않는다.

**ReplicaSet 의 부족한 부분을 보완해주는 Deployment**

<img width="619" alt="스크린샷 2023-11-14 오후 11 08 03" src="https://github.com/kihyuk-jeong/k8s/assets/39195377/a8f2498b-8915-44e0-916e-9752908d65d4">


Deployment 와 ReplicaSet 은 단독으로 사용되는 경우는 거의 없고, 함께 쓰인다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
	name: my-deployment
spec:
	replicas: 3
	strategy:
		type: RollingUpdate
		rollingUpdate:
			maxSurge: 1
			maxUnavailable: 1

// 아래는 replicaSet 설정
	selector:
		matchLabels:
			app: nginx
	template:
		metadata:
			labels:
				app: nginx
		spec:
			containers:
			- name: nginx
				image: nginx:1.17
```

**strategy : 파드를 업데이트 해야 하는 상황이 발생했을 때, 어떤 방식으로 업데이트를 할 것인가.**

- Recreate : 모든 파드를 한 번에 교체하는 전략
→ 파드가 전체 종료되면 서비스가 중단될 수 있음
- RollingUpdate : 각각의 파드를 순차적으로 업데이트 하는 전략
    - 무중단으로, 일반적인 서비스에서 선호되는 전략
    - readiness probe 로 pod 의 상태를 check 하면서 교체한다.
    - 단점으로는 파드가 순차적으로 교체되는 과정에서 클라이언트의 호출에 대한 결과값이 상이할 수 있다는 점. (예를 들어, 두 번 호출한다고 가정하면 첫 번째 호출은 이전 버전, 두 번째 호출은 최신 버전)
- maxSurge: 업데이트 하는 동안 파드가 얼마나 더 생성될 수 있는지
- maxUnavailable: 업데이트 하는 동안 파드가 얼마나 줄어들 수 있는지

→ 즉, 롤링업데이트 되는 동안 파드의 총 수는 `replicas - maxUnavailable ~ replcias + maxSurge`

