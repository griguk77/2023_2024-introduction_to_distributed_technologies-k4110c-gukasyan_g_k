# Лабораторная работа №1 "Установка Docker и Minikube, мой первый манифест."

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
**2. Манифест файла для развертывания vault**

```
apiVersion: v1
kind: Pod
metadata:
  name: vault
  labels:
    app: vault
spec:
  containers:
  - name: vault
    image: hashicorp/vault:latest
    ports:
    - containerPort: 8200
```
**apiVersion: v1** - версия API Kubernetes.

**kind: Pod** - тип создаваемого объекта.

**metadata:** - метаинформация о Pod.

**name: vault** - имя для идентификации.

**labels: app: vault** - метки для классификации Pod.

**spec:** - описывает спецификацию Pod.

**containers:** - контейнеры, запущенные в Pod.

**name: vault** - имя контейнера.

**image: hashicorp/vault:latest** - образ контейнера.

**ports:** - открытые порты.

**containerPort: 8200** - порт внутри контейнера, доступный для работы.

**3. Создание или обновление объекта в Pod**
```
$ kubectl apply -f vault.yaml
```
**4. Создаёт объект Service, который открывает доступ к Pod vault через порт 8200**
```
$ minikube kubectl -- expose pod vault --type=NodePort --port=8200
```
**5. Перенаправление локального порта 8200 на порт 8200 сервиса vault, чтобы получить доступ к сервису**
```
$ minikube kubectl -- port-forward service/vault 8200:8200
```

**6. Написать в браузере url**
```
http://localhost:8200/ui/vault/dashboard
```
**7. Создать новый терминал и найти Root Token**
```
$ minikube kubectl logs vault
```
<img width="1920" alt="1lab" src="https://github.com/user-attachments/assets/344d555b-bbfa-4a79-86e5-5ad72413f41f" />
<img width="561" alt="1" src="https://github.com/user-attachments/assets/cc6cb342-48bb-4110-b1be-eda8e3d2fe9d" />
