# Домашнее задание к занятию Troubleshooting

### Цель задания

Устранить неисправности при деплое приложения.

### Чеклист готовности к домашнему заданию

1. Кластер K8s.

### Задание. При деплое приложение web-consumer не может подключиться к auth-db. Необходимо это исправить

1. Установить приложение по команде:
```shell
kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
```
2. Выявить проблему и описать.
3. Исправить проблему, описать, что сделано.
4. Продемонстрировать, что проблема решена.

Ссылка на манифест - https://github.com/deadwhitepunk/hw-k8s-12/blob/main/task.yaml

Первое что нас встречает это отсутствие namespace, создаем их.

![Create namespaces](https://github.com/deadwhitepunk/hw-k8s-12/blob/main/img/create_ns.png)

Встретили ошибку стягивание образа. Ошибка заключалась в том что image использует старый формат манифеста Schema 1, эта поддержка была удалена в containerd 2.1

![After deploy](https://github.com/deadwhitepunk/hw-k8s-12/blob/main/img/after_deploy.png)
![Error pull image](https://github.com/deadwhitepunk/hw-k8s-12/blob/main/img/describe_po_image_pull_err.png)

Берем новый образ: image: curlimages/curl:latest и деплоим

Поды стартанули и мы идем разбираться дальше что же там не так.

![Deploy with new image](https://github.com/deadwhitepunk/hw-k8s-12/blob/main/img/pod_started_after_image.png)

Подключившись к web-consumer мы пытаемся получить доступ к сервису auth через curl как это сделано в манифесте, но нас встречает таймаут. 

![Unsuccess](https://github.com/deadwhitepunk/hw-k8s-12/blob/main/img/unsuccess_curl.png)

И тут я обратил внимание что поды находятся в разных namespaces.

Web-consumer - namespace web

auth-db - namespace data

А кубер резолвит DNS имена в "укороченном" типе только в пределах одного namespace. Поэтому нам нужно делать curl FQDN в данном случае "curl auth-db.data.svc.cluster.local".

curl прошел удачно.

![Success curl](https://github.com/deadwhitepunk/hw-k8s-12/blob/main/img/success_curl.png)

Из этого выходит 2 решения:
1. Поместить поды в один namespace.
2. Поменять команду проверки в манифесте.
- while true; do curl auth-db; sleep 5; done меняем на - while true; do curl auth-db.data.svc.cluster.local; sleep 5; done

Проверяем логи контейнера web-consumer!

![Success curl in pod](https://github.com/deadwhitepunk/hw-k8s-12/blob/main/img/success_curl_in_pod.png)



### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержат  скриншоты вывода необходимых команд, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.