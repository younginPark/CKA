# Core Concepts

## Kubernetes Architecture
- Kubernetes: CRI (Container Runtime Interface), Docker는 dockershim 씀
- Master: Nodes를 관리, 플랜, 스케줄, 모니터링 함
  - ETCD Cluster: key value 형태로 클러스터에 필요한 정보들 저장
  - Kube controller manager: 각 노드들에 있는 컨테이너들을 관리
  - kube-apiserver: 각 노드들과 통신하는 서버 
  - kube-scheduler: 어느 노드에 어떤 컨테이너 올릴지 스케줄링
- Worker Node: 각 노드들, 실제 컨테이너들이 올라옴
  - kubelet: kube-api server와 통신하며 마스터가 시키는 일 수행
  - kube-proxy: 각 노드들 간에 통신하며 읽을 수 있게 해줌
- Container Runtime Engine: Docker, containerD, rkt (로켓)

- 명령어
  - containerD: ctr (디버깅용으로 운영에서 비추), nerdctl (앞으로 계속 많이 사용될 것), containerD를 위해 사용
  - kubernetes: crictl  (쿠버네티스에서 공식을 낸걸로 CRI 호환가능한 런타임에서 지원)

- Kube-api Server
  - 사용자 인가
  - Request가 유효한지 체크
  - ETCD로부터 직접 Data 가져오고 업데이트
  - Scheduler와 통신
  - Kubelet과 통신

- Kube Controller Manager
  - 상태 확인
  - 상황 해결 위한 행동을 함
  - 컨트롤러는 쿠버네티스 시스템 안에 있는 다양한 컴포넌트들의 상태를 모니터링 함
    - Node Controller는 kube-apiserver 통해 각 워커 노드들의 상태를 파악함, ex. 5초에 한번씩 헬스체크
    - Replication Controller는 ReplicaSet의 개수가 요구하는대로 잘 있는지 체크하고 죽어있으면 하나 새로 띄움
    - 이외에 컨트롤러는 아주 많음, 쿠버네티스의 브래인, 커스터마이징 가능
    - 무언가 컨트롤러가 작동이 안되면 Kube Controller Manager의 설정부터 보는게 좋음

- Kube Scheduler
  - 스케줄러는 그저 어떤 Node에 어떤 Pod을 띄울지만 정하지 실제로 Pod을 위치 시키는 것은 아님 (이건 Kubelet 역할)
  - 2단계로 Pod를 어떤 Node에 띄울지 정해짐
    - Filter Nodes: Pod의 프로필에 맞는 Node를 찾음 (충분한 CPU나 메모리 자원이 있는가)
    - Rank Nodes: 1~10 중의 숫자로 Node들의 순위를 매김, 예를들면 pod을 띄우고 나서도 free한 리소스의 양이 많으면 점수를 더 줌
  - 위에 Rank Nodes를 커스터마이징해서 자신만의 스케줄러를 만들 수도 있음 (Resource Requirements and Limits, Taints and Tolerations, Node Selector/Affinity)

- Kubelet
  - master Node의 컨택 포인트
  - Scheduler가 시키는대로 하게 됨
  - 정기적으로 Node와 Pod의 상태를 모니터링해서 Kube-api server에 전달함
  - Kubelet 통해 Node 등록
  - Kube Scheduler -> Kube-api server -> Kubelet -> Runtime Engine(Docker) 
  - Kubeadm으로 cluster 구성하면 Kubelet이 자동으로 설치되지 않음, 수동으로 다운받아서 설치해야함

- Kube Proxy
  - pod 네트워크는 Internal virtual network
  - Service로 Pod간 통신이 가능해지는데 Service는 실재하는 것이 아닌 쿠버네티스 메모리에만 저장된거라 내부에 프로세스가 돌지 않아서 Pod 네트워크에 조인되진 못함, 그래서 클러스터 통해서 노드로 접근함 => Kube Porxy
  - Kube Proxy는 각 Node에서 실쟁되고 있는 프로세스 (daemonset)
  - iptables rules 같은 방법으로 서비스IP가 내부의 Pod IP와 연결되어 있음 

- Pod
  - container를 감싸는 object
  - 보통 application의 single instance로 one-to-one이지만 Pod 하나에 Multi Containers가 들어가기도 함
  - Multi Container에 들어가는 또 다른 pod은 helper Containers로 localhost 주소로 각 컨테이너들이 연결됨 

- yaml file
  - top level fields: apiVersion, kind, metadata, spec
    - apiVersion: kind에 따라 Version이 달라짐
    - kind: 셍성할 object 종류 
    - metadata: object의 name, labels 등, dictionary 형태로 저장됨, labels는 다른것과 다르게 쿠버네티스스에서 정해진대로 넣는게 아닌 자유롭게 넣을 수 있음
    - spec: spec 아래 containers가 들어가는데 이건 List/Array 형태, containers 아래에는 name과 image가 들어감

- Kube Controller
  - Replica Controller -> ReplicaSets
    - ReplicaSets로 바뀌면서 각 Pod들을 모니터링하는 역할까지 추가됨
    - 그래서 만약 replicas가 3이여서 pod이 3개 떠 있어야 하는데 없으면 template의 pod spec을 보고 띄울 수 있게 되고,
    - ReplicaSets에서는 selector를 통해 떠있는 여러 pod들 중 특정 label의 pod이 잘 떠있는지 모니터링 할 수 있게 됨
    - 명령어
      - k create -f replicaset.yml
      - k get replicaset
      - k delete replicaset myapp-replicaset
      - k replace -f replicaset.yml
      - k scale --replicas=6 -f replicaset.yml 
      - k scale --replicas=6 replicaset myapp-replicaset (type과 name 순으로 작성, 이미 생성한 replicaset의 Pod의 개수를 3 -> 6으로 변경할 때 사용)
      - k edit replicaset new-replica-set  (yaml 파일 위치 몰라도 수정가능하고 자동 업데이트됨, 대신 이미 떠있는 pod들은 죽였다 살려야 image 변경같은 사항들이 새로 반영됨)

- Deployment
  - Deployment가 ReplicaSet (Replica)보다 큰 개념
  - ReplicaSet yml 파일과 kind 빼고는 동일
  - Deployment yml을 k create 했을 때 deploymnet가 생기고 거기서 난수가 더해진 replicaset이 생기고 거기서 난수가 또 더해진 pod이 생성됨
  - Deployment를 굳이 써서 얻을 수 있는 이점은 추후에..

- Services
  - Service를 통해 end-user가 pod에 접근할 수 있게 됨
  - Services Types
    - NodePort: node에서 port로 서비스에 대해 리스닝하여 pod으로 request 전달
      - Target Port: POD의 port
      - Port: Service에서 Target Port 바라보는 port (Service의 ip는 ClusterIP에서 말한 vip)
      - Node Port: Node 자체의 Port로 외부에서 접근할 때 필요 (range: 30,000~32,767)
    - CluserIP: 클러스터 내부에서 Virtual IP를 생성하여 클러스터 내부에서 통신 가능하게 함 (ex. 각 svc간 통신 가능)
      - 여러개의 pod의 single interface를 제공
      - targetPort는 연결된 Pod의 포트이고, 그냥 port는 서비스의 Port
    - LoadBalancer: 클라우드에서 제공해주는 기능으로 Node의 로드밸랜서 역할
  - yml 작성 시 spec에 type과 ports를 적어주고(ports는 array라 - 가 들어감) selector도 (pod > metadata > labels에 들어가 있던 app, type 이런거 적어줌) 작성해줌
  - 노드 하나에 pod이 여러개 떠 있고 port와 label 정보가 같을 경우 내부 랜덤 알고리즘에 의해 pod으로 요청이 전송됨
  - 여러 노드가 클러스터링되어 있을 때 서비스를 하나 생성하면 특별한 작업 없이 전체 노드 전반에 걸쳐 서비스가 생성되고 포트가 설정되어 적절한 pod으로 접근 가능

- Namespaces
  - default, kube-system, kube-public은 기본적으로 생성되는 namespace
  - 관리자가 실수로 삭제, 변경하는 것을 막기 위해 namespace를 새로 생성해서 자원들을 관리할 수 있음 
  - namespace별 자원양 limit 가능
    - ResourceQuota 생성
  - mysql connect 예시
    - default namespace: mysql.connect("db-service")
    - dev namespace: mysql.connect("db-service.dev.svc.cluaster.local")
  - yaml 파일로 자원 생성 시에 namespace 지정을 원하면 metadata에 namespace 설정 가능
  - namespace를 새로 생성하고 싶을 때 apiVersion은 v1이고 kind에 Namespace, Metadata의 name에 dev 와 같은 원하는 namespace명 적어주면 됨
  - default namespace에서 dev namespace를 기본으로 바라보게 하고 싶으면 
    - kubectl config set-context $(kubectl config current-context) --namespace=dev
  - k get pods --all-namespaces  or k get pods -A
  - k get ns

- Imperative(명령형) vs Declarative(선언형)
  - 시험에서는 빨리 하는게 중요하니까 Imperative 추천, 하지만 실제 업무환경에서 특히 협업할 경우는 Declarative 권장
  - Imperative: run, create, delete, expose, edit 과 같은 명령들을 한줄한줄 command line 통해 명령
    - Creative Objects
      - k run --image=nginx nginx
      - k create deployment --image=nginx nginx
      - k expose deployment nginx --port=80
    - Update Objects
      - k edit deployment nginx
      - k scale deployment nginx --replicas=5
      - k set image deployment nginx nginx=nginx:1.18
      - Imperative 방식으로 관리 시에 파일 변경이 recorded 되지 않으므로 설정파일을 변경하고 k replace -f nginx.yaml 과 같이 replace를 통해 원본 설정 파일을 바꿔주는 것이 좋음
      - 설정 변경 후 강제로 object들을 replace 하고 싶을 경우 k replcae --force -f nginx.yaml과 같이 --force 명령어를 사용해 줌
  - Declarative: configuration file에 딱 원하는 결과 (destination)을 적어서 apply command 통해 생성 및 관리
    - 생성 및 수정 후 k apply -f configutation.yaml