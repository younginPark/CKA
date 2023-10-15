 - all namespace shortkut : -A
 
 ```
$ kubectl get deploy nginx -n nginx-demo -o yaml > nginx-demo.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2023-05-22T16:36:25Z"
  generation: 1
  labels:
    app: nginx
  name: nginx
  ```


$ k run sample --image=nginx --dry-run=client -o yaml > sample-pod.yaml

- Runnuing 중인 Pod을 수정하고 싶을 때
  - $ kubectl get pod webapp -o yaml > my-new-pod.yaml
  - vi 로 해당 파일 수정
  - 기존 pod 삭제
  - 수정했던 yaml 파일로 pod 다시 실행
- Running 중인 deployment 수정하고 싶을 때
  - $ kubectl edit deployment my-deployment

- 삭제 대신 replace
  - k replace --force -f asdadsa.yaml

- DaemonSet 생성 (deployment와 유사하기 때문에 deployment를 create 후 수정)
```
An easy way to create a DaemonSet is to first generate a YAML file for a Deployment with the command kubectl create deployment elasticsearch --image=registry.k8s.io/fluentd-elasticsearch:1.20 -n kube-system --dry-run=client -o yaml > fluentd.yaml. Next, remove the replicas, strategy and status fields from the YAML file using a text editor. Also, change the kind from Deployment to DaemonSet.

Finally, create the Daemonset by running kubectl create -f fluentd.yaml
```