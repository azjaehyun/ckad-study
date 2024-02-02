k apply -f- <<EOF
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: web
  name: web
  namespace: broken
spec:
  containers:
  - name: liveness
    image: nginx
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /
        port: 8080
      initialDelaySeconds: 3
      periodSeconds: 3
EOF
