k create ns ckad00018
k run ckad00018-newpod --image=nginx -n ckad00018
k run www --image=nginx -n ckad00018
k run db --image=nginx -n ckad00018
k label pod www -n ckad00018 app=www
k label pod db -n ckad00018 app=db
k apply -f- <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-access
  namespace: ckad00018
spec:
  podSelector:
    matchLabels:
      app: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          db-access: "true"
EOF
k apply -f- <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: www-access
  namespace: ckad00018
spec:
  podSelector:
    matchLabels:
      app: www
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          www-access: "true"
EOF



```
Context
You need to make a pod communicate only with the other two pods, not with the other two pods.

Task
Update pod ckad00018-newpod in namespace ckad00018 to use a NetworkPolicy that only allows traffic between this Pod and Pod www and db.

node-1:~$ kubectl contig use-context nk8s
[환경구성]:~$ k create ns ckad00018
[환경구성]:~$ k run ckad00018-newpod --image=nginx -n ckad00018
[환경구성]:~$ k run www --image=nginx -n ckad00018
[환경구성]:~$ k run db --image=nginx -n ckad00018
[환경구성]:~$ k label pod www -n ckad00018 app=www
[환경구성]:~$ k label pod db -n ckad00018 app=db
[환경구성]:~$ 52 page 생성
node-1:~$ k get pod -n ckad00018 -o wide
NAME                          READY   STATUS     RESTARTS   AGE     IP                 NODE
ckad00018-newpod   1/1           Running      0                    51s     10.44.0.24    nk8s-worker-1
db                                1/1           Running      0                    51s     10.44.0.21    nk8s-worker-1   
www                            1/1           Running      0                    58s     10.44.0.19    nk8s-worker-0
node-1:~$ k -n ckad00018 exec ckad00018-newpod  -it -- sh      # 실제 시험에서는 시간상 Skip 해도 될듯.
# curl 4 10.44.0.21
(.. 실패 ..)
# curl 4 10.44.0.19
(.. 실패 ..)
node-1:~$ k get networkpolicy -n ckad00018
 NAME                                      POD  
db-access                                app=db   
www-access                            app=www
node-1:~$ k get networkpolicy db-access -n ckad00018 -o yaml
(... selector 확인 … db-access: “true”)
node-1:~$ k get networkpolicy www-access -n ckad00018 -o yaml
(... selector 확인 … www-access: “true”)
node-1:~$ k get pod -n ckad00018 --show-labels
NAME                          READY   STATUS     RESTARTS   LABEL
ckad00018-newpod   1/1          Running     0                    <none>
db                                1/1          Running      0                    app=db   
www                            1/1          Running      0                    app=www
node-1:~$ k label pod ckad00018-newpod -n ckad00018 db-access=true
node-1:~$ k label pod ckad00018-newpod -n ckad00018 www-access=true  
node-1:~$ k -n ckad00018 exec -n ckad00018-newpod  -- sh
# curl 4 10.44.0.21
(.. 성공 ..)
# curl 4 10.44.0.19
(.. 성공 ..)

```

---
번외 https://kmaster.tistory.com/9


```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
      version: v1
  template:
    metadata:
      labels:
        app: hello
        version: v1
    spec:
      containers:
      - name: hello
        image: ghcr.io/kmaster8/hello:v1
        ports:
        - containerPort: 5050
---
apiVersion: v1
kind: Service
metadata:
  name: prod-service
spec:
  type: ClusterIP
  selector:
    app: hello
    version: v1
  ports:
  - protocol: TCP
    port: 5000
    targetPort: 5000
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tomcat
spec:
  rules:
  - host: test.prod.10.60.200.121.sslip.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: prod-service
            port:
              number: 5000
---                  
```

신규 앱
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello
      version: v2
  template:
    metadata:
      labels:
        app: hello
        version: v2
    spec:
      containers:
      - name: hello
        image: ghcr.io/kmaster8/hello:v2
        ports:
        - containerPort: 5050

```