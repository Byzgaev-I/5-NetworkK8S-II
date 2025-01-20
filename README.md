# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 2»

## Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к двум приложениям снаружи кластера по разным путям.

## Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (MicroK8S)
2. Установленный локальный kubectl
3. Редактор YAML-файлов с подключённым git-репозиторием

## Задание 1. Создать Deployment приложений backend и frontend

### 1. Создание Deployment

#### Frontend Deployment

Манифест [frontend-deployment.yaml](task1/frontend-deployment.yaml):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

#### Backend Deployment

Манифест [backend-deployment.yaml](task1/backend-deployment.yaml):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  labels:
    app: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 80
```

### 2. Создание Services

#### Frontend Service

Манифест [frontend-service.yaml](task1/frontend-service.yaml):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

#### Backend Service

Манифест [backend-service.yaml](task1/backend-service.yaml):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### 3. Результаты применения манифестов

Проверка подов:
```bash
kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
backend-7d87b45648-gwwcd           1/1     Running   0          2m42s
frontend-65d978c499-c6wk2          1/1     Running   0          2m42s
frontend-65d978c499-d4xbr          1/1     Running   0          2m42s
frontend-65d978c499-pc8kl          1/1     Running   0          2m42s
```

![image](https://github.com/Byzgaev-I/5-NetworkK8S-II/blob/main/5-1.png)

Проверка сервисов:
```bash
kubectl get services
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
backend-svc    ClusterIP   10.152.183.83    <none>        80/TCP    3m10s
frontend-svc   ClusterIP   10.152.183.73    <none>        80/TCP    3m10s
```
![image](https://github.com/Byzgaev-I/5-NetworkK8S-II/blob/main/5-2.png)


![image](https://github.com/Byzgaev-I/5-NetworkK8S-II/blob/main/5-1-1.png)

## Задание 2. Создать Ingress и обеспечить доступ к приложениям снаружи кластера

### 1. Включение Ingress в MicroK8S

```bash
sudo microk8s enable ingress
Addon core/ingress is already enabled
```

### 2. Создание Ingress

Манифест [ingress.yaml](task2/ingress.yaml):
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-svc
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-svc
            port:
              number: 80
```

### 3. Проверка работы Ingress

```bash
kubectl get ingress
NAME         CLASS    HOSTS   ADDRESS   PORTS   AGE
my-ingress   public   *                 80      15s
```
![image](https://github.com/Byzgaev-I/5-NetworkK8S-II/blob/main/5-3.png)


![image](https://github.com/Byzgaev-I/5-NetworkK8S-II/blob/main/5-2-1.png)








