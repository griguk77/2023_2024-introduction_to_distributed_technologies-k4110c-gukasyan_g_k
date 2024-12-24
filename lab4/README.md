# Лабораторная работа №4 "Сети связи в Minikube, CNI и CoreDNS"

University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)

Year: 2023/2024

Group: K4110с

Author: Gukasyan Grigory Kamoevich

Lab: Lab4

Date of create: 24.12.2024

Date of finished: 

**1. Установка плагина и режима работы Multi-Node Clusters**
```
$ minikube start --network-plugin=cni --cni=calico --nodes 2 -p multinode-demo
```
**2. Проверка работы Calico**

```
kubectl get pods -l k8s-app=calico-node -A
kubectl get nodes
```


**3. Пометка нод** 

--overwrite=true - если уже помечали nods до этого и хотите rename

```
$ kubectl label nodes multinode-m02 rack=1 --overwrite=true
$ kubectl label nodes multinode rack=0 --overwrite=true 
```
![3](https://github.com/user-attachments/assets/c8a4e8a6-1031-401c-a899-5b7a5aeecb11)



**4. Манифест для Calico**

```
apiVersion: crd.projectcalico.org/v1
kind: IPPool
metadata:
  name: rack-0
spec:
  cidr: 192.168.30.0/24
  ipipMode: Always
  natOutgoing: true
  nodeSelector: rack == "0"
---
apiVersion: crd.projectcalico.org/v1
kind: IPPool
metadata:
  name: rack-1
spec:
  cidr: 192.168.40.0/24
  ipipMode: Always
  natOutgoing: true
  nodeSelector: rack == "1"
```

**5. Скачивание конфигурационного файла**

**(перед командой необходимо [скачать конфигурационный файл)](https://github.com/projectcalico/calico/blob/master/manifests/calicoctl.yaml)**
```
$ kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml
```

**6. Создание deployment**
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment
  labels:
    app: lab4
spec:
  replicas: 2
  selector: 
    matchLabels:
      app: lab4
  template:
    metadata:
      labels:
        app: lab4
    spec:
      containers:
      - name: lab4
        image: ifilyaninitmo/itdt-contained-frontend:master
        env:
        - name: REACT_APP_USERNAME
          value: gukasyan
        - name: REACT_APP_COMPANY_NAME
          value: itmo
        ports:
        - containerPort:  3000

```

**7. Создание service**
```
apiVersion: v1
kind: Service
metadata:
  name: service-itmo-lab-4
spec:
  type: NodePort
  selector:
    app: lab4
  ports:
    - port: 3000
      protocol: TCP
      name: http
```

**8. Обновление deployment и service**
```
$ kubectl apply -f deployment.yaml
$ kubectl apply -f service.yaml
```

**9. Вход в веб браузер**
```
localhost:3000
```


![9(1)](https://github.com/user-attachments/assets/0506365c-04e5-4c0b-b7a6-e95dcb5ac2b8)

![9(2)](https://github.com/user-attachments/assets/c0a87561-e8cf-4ca1-b401-5149ff96a149)


**10. Узнать информацию о FQDN имена подов**
```
$ kubectl get pods -o wide
NAME                          READY   STATUS    RESTARTS   AGE     IP               NODE                 NOMINATED NODE   READINESS GATES
deployment-5d4bf8f7f6-hdn6g   1/1     Running   0          3m52s   10.244.239.2     multinode-demo-m02   <none>           <none>
deployment-5d4bf8f7f6-wgmgs   1/1     Running   0          3m52s   10.244.113.195   multinode-demo       <none>           <none>


$ kubectl exec deployment-5d4bf8f7f6-hdn6g nslookup 10.244.239.2
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
Server:  10.96.0.10
Address: 10.96.0.10:53

2.239.244.10.in-addr.arpa name = 10-244-239-2.service-itmo-lab-4.default.svc.cluster.local


$ kubectl exec deployment-5d4bf8f7f6-wgmgs nslookup 10.244.113.195
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
Server:  10.96.0.10
Address: 10.96.0.10:53

195.113.244.10.in-addr.arpa name = 10-244-113-195.service-itmo-lab-4.default.svc.cluster.local
```

![10(1)](https://github.com/user-attachments/assets/e665e956-e8a6-4794-bc79-9b197bc64022)
![10(2)](https://github.com/user-attachments/assets/7f8cd4df-c8f0-41e6-afc6-95c116126ba9)
![10(3)](https://github.com/user-attachments/assets/73233bf9-5a8a-4615-8aef-f06936153dfc)



**11. Пинг подов**
```
$ kubectl exec -it deployment-5d4bf8f7f6-hdn6g sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
/frontend # ping 10-244-113-195.service-itmo-lab-4.default.svc.cluster.local
PING 10-244-113-195.service-itmo-lab-4.default.svc.cluster.local (10.244.113.195): 56 data bytes
64 bytes from 10.244.113.195: seq=0 ttl=62 time=0.364 ms
64 bytes from 10.244.113.195: seq=1 ttl=62 time=0.859 ms
64 bytes from 10.244.113.195: seq=2 ttl=62 time=0.508 ms
64 bytes from 10.244.113.195: seq=3 ttl=62 time=0.649 ms
64 bytes from 10.244.113.195: seq=4 ttl=62 time=0.297 ms

$ kubectl exec -it deployment-5d4bf8f7f6-wgmgs sh 
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
/frontend # ping 10-244-239-2.service-itmo-lab-4.default.svc.cluster.local
PING 10-244-239-2.service-itmo-lab-4.default.svc.cluster.local (10.244.239.2): 56 data bytes
64 bytes from 10.244.239.2: seq=0 ttl=62 time=0.407 ms
64 bytes from 10.244.239.2: seq=1 ttl=62 time=0.307 ms
64 bytes from 10.244.239.2: seq=2 ttl=62 time=0.560 ms
64 bytes from 10.244.239.2: seq=3 ttl=62 time=0.501 ms
64 bytes from 10.244.239.2: seq=4 ttl=62 time=0.228 ms
64 bytes from 10.244.239.2: seq=5 ttl=62 time=0.473 ms
64 bytes from 10.244.239.2: seq=6 ttl=62 time=0.286 ms
```
![11(1)](https://github.com/user-attachments/assets/34ec0ca3-f5de-4ebc-9cb0-4da6163da81e)
![11(2)](https://github.com/user-attachments/assets/310e6203-f893-4314-aec7-a8411626f460)
