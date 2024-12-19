# Лабораторная работа №2 "Развертывание веб сервиса в Minikube, доступ к веб интерфейсу сервиса. Мониторинг сервиса."

University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)

Year: 2023/2024

Group: K4110

Author: Gukasyan Grigory Kamoevich

Lab: Lab1

Date of create: 17.12.2024

Date of finished: 

**1. Развернуть minikube cluster**
```
$ minikube start
```
**2. Манифест файла для развертывания deployment-react**

```
apiVersion: apps/v1
kind: Deployment                                            
metadata:
  name: deployment-react                         
spec:
  replicas: 2
  selector:
    matchLabels:
      app: react
  template:
    metadata:
      labels:
        app: react
    spec:                                      
      containers:
        - image: ifilyaninitmo/itdt-contained-frontend:master
          name: react                           
          ports:
          - name: react-port
            containerPort: 3000
          env:
            - name: REACT_APP_USERNAME
              value: 'Gukasyan'
            - name: REACT_APP_COMPANY_NAME
              value: 'itmo'
```

**replicas: 2** - определяет количество экземпляров приложения, которые будут запущены одновременно.

**selector:** - определяет, какие поды должны управляться этим деплойментом.

**3. Манифест файла для Service**
```
apiVersion: v1
kind: Service
metadata:
  name: front-service
spec:
  selector:
    app: react
  type: NodePort
  ports:
    - port: 3010
      name: react-port
      targetPort: react-port
      protocol: TCP
```
**4. Привязывание манифеста**
```
$ kubectl apply -f deployment.yaml
$ kubectl apply -f service.yaml
```
**5. Перенаправление запрос с Service на pods**
```
$ minikube kubectl -- port-forward service/frontend-service 3010:3010
```

**6. Написать в браузере url**
```
http://localhost:3010
```
**7. Создать новый терминал и найти логи**
```
$ minikube kubectl -- get pods

NAME                                READY   STATUS    RESTARTS   AGE
deployment-react-59c9dc96c7-k2bzb   1/1     Running   0          3m33s
deployment-react-59c9dc96c7-xn8mr   1/1     Running   0          3m33s
vault                               1/1     Running   0          17m


$ minikube kubectl -- logs deployment-react-59c9dc96c7-k2bzb
Builing frontend
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
build finished
Server started on port 3000

$ minikube kubectl -- logs deployment-react-59c9dc96c7-xn8mr
Builing frontend
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
build finished
Server started on port 3000
```
<img width="1920" alt="1" src="https://github.com/user-attachments/assets/752c3dfa-f3aa-4c4c-9a36-cdf99aa8f1f8" />
<img width="677" alt="2" src="https://github.com/user-attachments/assets/9ae8dbf5-5674-4262-a91e-d38c1623a608" />
<img width="539" alt="2" src="https://github.com/user-attachments/assets/53b2ba85-89aa-4413-a944-7f3147db06bf" />
