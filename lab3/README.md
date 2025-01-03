# Лабораторная работа №3 "Сертификаты и "секреты" в Minikube, безопасное хранение данных."

University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)

Year: 2023/2024

Group: K4110с

Author: Gukasyan Grigory Kamoevich

Lab: Lab3

Date of create: 21.12.2024

Date of finished:

**1. Развернуть minikube cluster**
```
$ minikube start
```
**2. Манифест файла для развертывания deployment-react**

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: react-env
data:
  react_app_username: "gukasyan"
  react_app_company_name: "itmo"

---

apiVersion: apps/v1
kind: ReplicaSet                                            
metadata:
  name: react-repset                   
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
              valueFrom:
                configMapKeyRef:
                  name: react-env
                  key: react_app_username
            - name: REACT_APP_COMPANY_NAME
              valueFrom:
                configMapKeyRef:
                  name: react-env
                  key: react_app_company_name

---

apiVersion: v1
kind: Service
metadata:
  name: react-service
spec:
  selector:
    app: react
  type: ClusterIP
  ports:
    - port: 3030
      name: react-port
      targetPort: react-port
      protocol: TCP

---

apiVersion: v1
kind: Secret
metadata:
  name: react-tls
type: kubernetes.io/tls
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUREVENDQWZXZ0F3SUJBZ0lVY2xHaWNzbFdFVTdWTTVvSzRHeVVFQmdrSy80d0RRWUpLb1pJaHZjTkFRRUwKQlFBd0ZqRVVNQklHQTFVRUF3d0xaM1ZyWVhONVlXNHViV1V3SGhjTk1qUXhNakl4TVRNek9UTTNXaGNOTWpVeApNakl4TVRNek9UTTNXakFXTVJRd0VnWURWUVFEREF0bmRXdGhjM2xoYmk1dFpUQ0NBU0l3RFFZSktvWklodmNOCkFRRUJCUUFEZ2dFUEFEQ0NBUW9DZ2dFQkFMOGFqcXNzR3J2WmJHcHVlM3BJZERwUVIxNjIxei8zUHRPL3N3bk8KQTdXa0svR0lLeitMcHhkdlE0R1B2MGlxdkxiZG15aENTemJhekZlRmM0a21qRnVpOGxoLzF2aTl1RzJVaWViNQpnb3M4UG91Y0RvcU9BQ2ZYT2t4aFlZeXlwWWNtZ2ZZbm5HejBXaG5LRnhGSEk1N0FJcWN3UkllZytaY0kyVE82CmZRMEpMRVNvMnBCUXY5TEdXdmxUL3lzUGxlYXhTSWRvRnhjSmlDYlg4V3NuQTZpc3htejJQaklSZ3IyOC84VmsKZmkza0E1dFlTWjZKWVBXeXNlaUpjeWlwZ3U2QVBoTTM4WTFqaVY1YzJ0VytlTnBReHVTa2VjRTIwWEhHQXBZVwpOTE1WNHZrY05ibVdLVkF2YXo3bVAzYm5oVjErdCtmZ3NLU0x2RDlqckUxUWoyY0NBd0VBQWFOVE1GRXdIUVlEClZSME9CQllFRk9ZVnd3bXhvbHIzeG9jOTNNNm9zYndneThnaE1COEdBMVVkSXdRWU1CYUFGT1lWd3dteG9scjMKeG9jOTNNNm9zYndneThnaE1BOEdBMVVkRXdFQi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQgpBRFlBTTBwbVBkQUl0b1ZRQWVNNSs2ZWpQOWNIa1lYOWNpYzd1VWtvTENrQThub3V0K3I4V01ucEtPV1dTUHRsCnhIckFkUENwek5JNDJSMVpMR293TjdwRmZqdCsyOEZ0Qkp0OEl4SXJKdEErSUZZTlN2V1lQd1dGSmU2dDg0ZGgKWkN5aWx4VE42Uy9HWFdITHFoUlZZSUNNbVExTEljM3R1MHZDUDNOSnpDR045ZGlZZDA1Zkx2R2k0T1ZFdjFCcApIamZqUENHd28xSDFVTHQwSmJsVFd0Rlg5dHpOTENMVHFIT1dPQzA5UUxOd2JzNFp2ZkFjUHV4NHpLK01RV2luCmVDLzhRSU9zV0xQZW9rY3FNczUvYUNTTjdJd1VRN05JUm8wb3J5a0pnVk9zcFFTNXNJZm41eHBqQTlwZFZ2K2UKK2dNWTI0eHpubkM2aDdzdk52eU83RW89Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
  tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2UUlCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQktjd2dnU2pBZ0VBQW9JQkFRQy9HbzZyTEJxNzJXeHEKYm50NlNIUTZVRWRldHRjLzl6N1R2N01KemdPMXBDdnhpQ3MvaTZjWGIwT0JqNzlJcXJ5MjNac29Ra3MyMnN4WApoWE9KSm94Ym92SllmOWI0dmJodGxJbm0rWUtMUEQ2TG5BNktqZ0FuMXpwTVlXR01zcVdISm9IMko1eHM5Rm9aCnloY1JSeU9ld0NLbk1FU0hvUG1YQ05renVuME5DU3hFcU5xUVVML1N4bHI1VS84ckQ1WG1zVWlIYUJjWENZZ20KMS9Gckp3T29yTVpzOWo0eUVZSzl2UC9GWkg0dDVBT2JXRW1laVdEMXNySG9pWE1vcVlMdWdENFROL0dOWTRsZQpYTnJWdm5qYVVNYmtwSG5CTnRGeHhnS1dGalN6RmVMNUhEVzVsaWxRTDJzKzVqOTI1NFZkZnJmbjRMQ2tpN3cvClk2eE5VSTluQWdNQkFBRUNnZ0VBRWdZS01LQ3ZRTXBIekYzeExWNUovL24wVVQyRFhaZ1BCOStMbmtBYzMzUEYKWlRsSGd1M0Q2NVRsMUFGZnRCWE9TSnpyOWtjU1d2RVYxcWRTZHp2NGZ1Z0dYVXhwVlBwbnU0WUgzNDNtdndVRwpqUnBCME5XRnREdzdWbHJVM1hVMzRXQkRYNWpxeXVmYzI1UFF5eU5mZTZoNVhEVlVNVXFBWDd0STZQVU11RDdjCmJJcitTRTdPbUxGOWIvRDJ2Slordk5DaWNhNTlvWWVoeVBtNm44a1V6eXNDcFQrMWJiQk1CZzFDdlY4akZnT3QKd0lLeFQzNXZkMVlFNFA5eXFNVFFrczk3QVg4VXpGQnZBeUxRbm1KSVd2WjF3M21EZnRxQWM3b3krOG1XSnlZawp4OVpNNUpFbDYzc1gyZWpUdzVQS01sUUpEY2xTektkZEdNRndRSmtENFFLQmdRRHJ1RmQ4c1Jmd1AzRDNBVFVwClpyMkVOVy9aY0RBeWtHenkyV1FLc2QrZ0d0c05hY2hYVTFLV3VVN21Dek9GTlA2bFhrdm54OHRTanZaNmord00KMk5VS1hObksvd1QrcEJXMDAzNjB2ZmVaZkxsMkJ4ZkcvcjZmbnVQTVBQRnRSY0dDUWUvU0VmclZPNVlSajZmawpnSUZqaC8vR2w3QUZkZ01zU0VJMWRXQkdkd0tCZ1FEUGk0NFhjejVTdzc4cUxKN3pzRTV6ekFtQ2l6QjNUc1V6ClUxcW5RRSt3YWdBeldTV1hJQjBUUkROcnY2blp2TVNUU1Q5L25EQmVtRFZ2Z2RwV3NFVm0wUVF2TW00VW04MkIKajJPd0szUkJnNmxDeVVqN2VSNmpUQ3J6OVBrZ3NUTS9OYk1oZWJiNW1XSFdaRXFyZGZ1SWxkUDAxMUduYWV0RgpIcFJGd3pnS2tRS0JnUUMrN3ZIR3FiZ0hQWXdtTjQ4MU91aFYxK2NDakxJdHN2amMrU1BrMmRHVzRVb0FJbWhKCis4OFJFWWNNSlpTVTJLbFBrQU1SK3E4Qjc1Vk5ENWtYaXVkOTNsbjM2UHZGdTJsdHNFYVk1cXRSWTByaWhMcEkKMFAzZFU1bVhUT0lPTGd1aGxBRksrbzlmKzBVQ2NvZC9PbXdVRUF3cGM2TDd1V3kyaU0xQWVoUUMzd0tCZ0d5RwpQOFdaV0VSMDROZ1B2d29UN2VIMUZoL3g0bVR3aG9OaEdhME5IdDVUZjBsYVd5S2NBemdZMkg2R0dTMm0zRzliCnhOMVljWjUxSHJQeEJaZUEwcm1Cb0J1QjFqZm1oRjR6K1YrY1NVMGNxSHdvdm9Yb2ZwSEsrVWJabVE5ME9TVGIKVXBDMWtXMFF2QzBjQWtPSURRQU53R2h0MHQra3JnWlZpQmkyak81UkFvR0FRc2o0NEttL3R3bmhCREhPbWNHVgpydlZvS3NDSGkydWF6c0xUeExEMk9GVStvV3NaVCttTzJHc1d6S091bGQyVEo4aWxlOU05azhSK3dhWDNyYThzClZhVXNzeHhLWlNZcmpLMEVJejNzVVZlOENPT0dEU0RoWVpZN1AxTHpBcXFLR2VoTWIyMy9leEFValJzdEJ4M0EKUGsrNGRndmp6bk1ZaXFYamxnY1U0ZFU9Ci0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0K

---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: react-ingress
spec:
  tls:
    - secretName: react-tls
      hosts:
        - gukasyan.me
  rules:
    - host: gukasyan.me
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: react-service
                port:
                  number: 3030
```


**3. Привязывание манифеста**
```
$ kubectl apply -f cert.yaml
```
**4. Включение Ingress**
```
$ minikube addons enable ingress
```

**5. Создание сертификата**
```
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=gukasyan.me"
```
**6. Вывести ключи**
```
$ cat tls.crt | base64
$ cat tls.key | base64

```

**7. Изменить вместо localhost добавить свое название**
```
$ sudo nano /etc/hosts
```

**8. Обновить манифест**
```
$ kubectl apply -f cert.yaml
```

**9. Запуск tunnel**
```
$ minikube tunnel
```
![3(1)](https://github.com/user-attachments/assets/df9930bf-e769-4f88-981f-f6dd7916f0a7)
![3(2)](https://github.com/user-attachments/assets/80357408-3b79-4bdb-873d-2951e6725466)
<img width="985" alt="3draw" src="https://github.com/user-attachments/assets/8dee5b95-a1e9-41af-8c92-e98e4f5d0f90" />
