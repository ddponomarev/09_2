# Домашнее задание к занятию "`Кластеризация и балансировка нагрузки`" - `Пономарев Денис`

### Задание 1
- Запустите два simple python сервера на своей виртуальной машине на разных портах
- Установите и настройте HAProxy, воспользуйтесь материалами к лекции по [ссылке](2/)
- Настройте балансировку Round-robin на 4 уровне.
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy.

```
Конфигурация haproxy.cfg
listen stats  # веб-страница со статистикой
        bind                    :888
        mode                    http
        stats                   enable
        stats uri               /stats
        stats refresh           5s
        stats realm             Haproxy\ Statistics

frontend example  # секция фронтенд
        mode http
        bind :8088
        #default_backend web_servers
        acl ACL_example.com hdr(host) -i example.com
        use_backend web_servers if ACL_example.com

backend web_servers    # секция бэкенд
        mode http
        balance roundrobin
        option httpchk
        http-check send meth GET uri /index.html
        server s1 127.0.0.1:8888 check
        server s2 127.0.0.1:9999 check


listen web_tcp

        bind :1325
        mode tcp
        balance roundrobin
        server s1 127.0.0.1:8888 check inter 3s
        server s2 127.0.0.1:9999 check inter 3s
```


![Задание1](https://github.com/ddponomarev/09_1/blob/master/img/z1.png)

---

### Задание 2
- Запустите три simple python сервера на своей виртуальной машине на разных портах
- Настройте балансировку Weighted Round Robin на 7 уровне, чтобы первый сервер имел вес 2, второй - 3, а третий - 4
- HAproxy должен балансировать только тот http-трафик, который адресован домену example.local
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него.

```
Скрипт web.sh
#!/bin/bash

if nc -z localhost 80 && [ -f /var/www/html/index.nginx-debian.html ]; then
    exit 0
else
    exit 1
fi
```


```
Конфигурация keepalived.conf
#!/bin/bash

vrrp_script check_web {
        script "/etc/keepalived/web.sh"
        interval 3
}

vrrp_instance VI_1 {
        state MASTER
        interface ens3
        virtual_router_id 50
        priority 255
        advert_int 1

        virtual_ipaddress {
              192.168.0.50/24
        }

        track_script {
              check_web
        }
}
```

![Задание2.1](https://github.com/ddponomarev/09_1/blob/master/img/zad2_1.png)

![Задание2.2](https://github.com/ddponomarev/09_1/blob/master/img/zad2_2.png)
---

### Задание 3*
- Настройте связку HAProxy + Nginx как было показано на лекции.
- Настройте Nginx так, чтобы файлы .jpg выдавались самим Nginx (предварительно разместите несколько тестовых картинок в директории /var/www/), а остальные запросы переадресовывались на HAProxy, который в свою очередь переадресовывал их на два Simple Python server.
- На проверку направьте конфигурационные файлы nginx, HAProxy, скриншоты с запросами jpg картинок и других файлов на Simple Python Server, демонстрирующие корректную настройку.

```
Скрипт web.sh
#!/bin/bash

if nc -z localhost 80 && [ -f /var/www/html/index.nginx-debian.html ]; then
    exit 0
else
    exit 1
fi
```

![Задание2.1](https://github.com/ddponomarev/09_1/blob/master/img/zad2_1.png)

### Задание 4*
- Запустите 4 simple python сервера на разных портах.
- Первые два сервера будут выдавать страницу index.html вашего сайта example1.local (в файле index.html напишите example1.local)
- Вторые два сервера будут выдавать страницу index.html вашего сайта example2.local (в файле index.html напишите example2.local)
- Настройте два бэкенда HAProxy
- Настройте фронтенд HAProxy так, чтобы в зависимости от запрашиваемого сайта example1.local или example2.local запросы перенаправлялись на разные бэкенды HAProxy
- На проверку направьте конфигурационный файл HAProxy, скриншоты, демонстрирующие запросы к разным фронтендам и ответам от разных бэкендов.

```
Скрипт web.sh
#!/bin/bash

if nc -z localhost 80 && [ -f /var/www/html/index.nginx-debian.html ]; then
    exit 0
else
    exit 1
fi
```

![Задание2.1](https://github.com/ddponomarev/09_1/blob/master/img/zad2_1.png)