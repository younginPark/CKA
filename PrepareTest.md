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


$ k run sample --image=nginx --dry-run=client -o yaml > sample-pod.yaml
