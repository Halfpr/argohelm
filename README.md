Для запуска нужно:
1. Поднять свой кубер с ингресс. Так же нужен melallb для того чтобы ингресс получил свой external-ip. На этот айпи настроть A запись в днс. 
У меня это kubertest.secretgroup.dev(*.secretgroup.dev)  Серт предназначен для локального тестирования ингресс и не смотрит во внешнею сеть.
Можно и без ингресс, но тогда придётся переделывать service манифест c cluster ip на  load balancer.
2. Images можно использовать только те, которые есть в harbor. Внутри есть подключение к нему, через мою AD учётку.
Например ваш логин и  пароль в AD test1 и Tt123456789, то в поле "auth" надо записать test1:Tt123456789 в base64 формат(смотреть файл values.yaml)
3. Так же нужно поднять свой NFS сервер. Отдельная машина с двумя дисками. Я сделал наспех для проверки вот так
sdb           8:16   0   25G  0 disk 
└─vg01-lv01 253:0    0   25G  0 lvm  /mnt/vg01-lv01     - это для базы
sdc           8:32   0   25G  0 disk 
└─vg02-lv02 253:1    0   25G  0 lvm  /opt/sce-data      - это для файлов
Можете точно так же создать и тогда вам остаётся только сменить IP адрес в Persistent volum на ваш NFS сервер.
4.  Псоле того как скопировали хелмы к себе, то под тем же пользователей, через который подключаетесь к куберу, запускаете helm install zp ./kuber
Для этого устанавливаете сам helm. ./kuber Это просто папка где лежат чарты.
5. Псоле запуска, Pod базы может падать, потому что max_connections там 100. Для этого нужно проделать следующее:
Шаги

#kubectl get pods
#Подключитесь к Pod-у

#kubectl exec -it zp-psql-7b98649c6d-9r22x -- /bin/sh
#Измените конфигурационный файл PostgreSQL
#После подключения к Pod-у, отредактируйте конфигурационный файл postgresql.conf. Обычно он находится в директории /var/lib/postgresql/data/.


#vi /var/lib/postgresql/data/postgresql.conf
#Найдите строку с max_connections и измените её значение на 1000.:

#max_connections = 1000
#Перезапустите PostgreSQL
#После изменения конфигурационного файла, перезапустите PostgreSQL. В зависимости от того, как PostgreSQL запущен внутри контейнера, это может быть сделано разными способами. Один из способов — перезапустить контейнер:

#kubectl rollout restart deployment zp-psql

#Ждёте минуту и всё должно заработать =D