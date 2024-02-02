### [학습 목표](https://velog.io/@pinion7/Kubernetes-%EB%A6%AC%EC%86%8C%EC%8A%A4-Secret%EC%97%90-%EB%8C%80%ED%95%B4-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B3%A0-%EC%8B%A4%EC%8A%B5%ED%95%B4%EB%B3%B4%EA%B8%B0) 


k -n moon create secret generic secret1 --from-literal user=test --from-literal pass=pwd
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-myapp
  template:
    metadata:
      labels:
        app: nginx-myapp
    spec:
      containers:
      - name: nginx-myapp
        image: nginx:latest
        env:
        - name: MY_USERNAME
          valueFrom:
            secretKeyRef:
              name: secret1
              key: user
        - name: MY_PASSWORD
          valueFrom:
            secretKeyRef:
              name: secret1
              key: pass
```              


apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-myapp
  template:
    metadata:
      labels:
        app: nginx-myapp
    spec:
      containers:
      - name: nginx-myapp
        image: nginx-myapp:latest
        envFrom:
        - secretRef:
            name: secret1