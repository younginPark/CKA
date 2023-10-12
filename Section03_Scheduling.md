# Scheduling

- Manual Scheduling
  - Pod의 설정파일에 nodeName으로 직접 어떤 노드에 띄으고 싶은지 설정 가능
  - Pod의 설정파일에 nodeName을 명시하지 않으면 자동으로 각 node들의 상태를 보고 띄움
  - 이미 존재하는 pod의 nodeName을 변경하고 싶을 경우 수정보다 pod의 node에 binding할 수 있는 설정파일을 생성하여 pod's binding api에 리퀘스트
    - 예시
        apiVersion: v1
        kind: Binding
        metadata:
            name: nginx
        target:
            apiVersion: v1
            kind: Node
            name: controlPlane
  - pod을 실행시켰는데 state가 pending이고 node에 none이라고 뜨면 kube-system 네임스페이스에서 scheduler가 잘 돌고있는지 확인 필요
  - delete + apply 대신 k replace --force -f nginx.yaml도 가능

- Labels and Selectors
  - Labels: 속성을 부여할 수 있음
    - Key, Value 형태로 metadata 아래에 저장
  - Selectors: 각 속성들을 필터링 할 수 있음
  - Annotations: Labels에 쓰지 않는 디테일들을 기록할 수 있음 

- Taints and Tolerations
  - 이 기능들은 보안을 위한게 아니라 그저 pod이 Node에 schedule 되는 것을 제한할 수 있을 뿐
  - Node에 taint 해두면, taint 와 동일한 toleration을 가진 pod만 해당 Node에 띄울 수 있음
  - 특정 Node에 대해 toleration을 가진 pod이 무조건 특정 Node에 뜨는 것은 아니고 스케줄이 되었을 때 뜰 수 있을 뿐임
  - Master Node에는 자동적으로 taints 되어 있어서 pod이 Master Node에는 스케줄되지 않음
  - Taints
    - k taint nodes node-name key=value:taint-effect (- untaint 하려면 - 만 붙이면 됨)
    - taint-effect 종류
      - NoSchedule
      - PreferNoSchedule
      - NoExecute -> 이미 실행되고 있는 pod도 쫓아냄, pod killed
  - Tolerations
    - pod yaml file에서 spec > container와 같은 레벨
    - 예시 (따옴표로 감싸줘야 함)
    tolerations:
        - key:"app"
          operator:"Equal"
          value:"blue"
          effect:"NoSchedule"

- Node Selectors
  - 특정 pod 서비스가 특정 Node에만 띄워져야 할 때 (ex. Node의 하드웨어 스펙이 중요) pod 설정파일에 Node를 지정할 수 있음
  - 방법은 Node의 labels에 key value 적어두고 pod의 nodeSelector에 해당 key value 적어주면 됨
  - Node Selectors만으로는 복잡하게 label 설정을 할 수 없음 => Node Affinity 필요
  - spec의 Containers와 같은 레벨
  - 예시
    nodeSelector:
        label-key:label-value
  - label Nodes 방법
    - k label nodes node-name label-key=label-value

- Node Affinity
  - Node Selectors 보다 더 복잡하게 Node를 선택할 수 있게 함
  - Node Affinity Types
    - Available:
      - requiredDuringSchedulingIgnoredDuringExecution
      - preferredDuringSchedlingIgnoredDuringExecution (조건에 매칭되게 최선을 다해라)
    - Planned:
      - requiredDuringSchedulingRequiredDuringExecution
  - 예시 (containers와 같은 레벨)
    affinity:
        nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                    - matchExpressions:
                        - key: size
                          operators: Exists


