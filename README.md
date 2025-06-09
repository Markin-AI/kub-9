# Домашнее задание к занятию "`Конфигурация приложений`" - `Маркин Алексей`

### Цель задания

В тестовой среде Kubernetes нужно предоставить ограниченный доступ пользователю.

### Задание 1. Создайте конфигурацию для подключения пользователя

1. Создайте и подпишите SSL-сертификат для подключения к кластеру.
2. Настройте конфигурационный файл kubectl для подключения.
3. Создайте роли и все необходимые настройки для пользователя.
4. Предусмотрите права пользователя. Пользователь может просматривать логи подов и их конфигурацию (`kubectl logs pod <pod_id>`, `kubectl describe pod <pod_id>`).
5. Предоставьте манифесты и скриншоты и/или вывод необходимых команд.

### Решение

Генерируем ключ и запрос на выпуск сертификата
```sh
openssl genrsa -out user1.key 2048
```
```sh
openssl req -new -key user1.key -out user1.csr -subj "/CN=user1"
```

Подписываем запрос корневым сертификатом
```sh
openssl x509 -req -in user1.csr -CA /var/snap/microk8s/current/certs/ca.crt \
  -CAkey /var/snap/microk8s/current/certs/ca.key -CAcreateserial \
  -out user1.crt -days 365
```

Создаем пользователя и контекст в конфигурацию ```kubectl```
```sh
kubectl config set-credentials user1 --client-certificate user1.crt --client-key user1.key --embed-certs=true
```
```sh
kubectl config set-context user1 --cluster=microk8s-cluster --user=user1
```

![1](https://github.com/Markin-AI/kub-9/blob/main/img/1.png)

Созадаем [роль](https://github.com/Markin-AI/kub-9/blob/main/role.yaml)

Созадаем [binding роли](https://github.com/Markin-AI/kub-9/blob/main/binding-role.yaml)

![2](https://github.com/Markin-AI/kub-9/blob/main/img/2.png)

Созадаем [deploy](https://github.com/Markin-AI/kub-9/blob/main/deploy.yaml) приложения 

Переключаем контекст и проверяем права
```sh
kubectl config use-context user1
```

![3](https://github.com/Markin-AI/kub-9/blob/main/img/3.png)

```sh
kubectl logs nginx-54c98b4f84-xpqtv
```

![4](https://github.com/Markin-AI/kub-9/blob/main/img/4.png)

```sh
kubectl describe nginx-54c98b4f84-xpqtv
```

![5](https://github.com/Markin-AI/kub-9/blob/main/img/5.png)

------

### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

------