# HAPROXY_NGINX



1. Создаем директории 

`` mkdir http1 http2 ``
2. Создаем index.html

```
echo "SERVER 1:8888" > http1/index.html
echo "SERVER 2:9999" > http2/index.html
```
3. Запускаем простой web - сервер 
```
cd http1
python3 -m http.server 9999 --bind 0.0.0.0

# В дугом окне
cd http2
python3 -m http.server 9999 --bind 0.0.0.0
```

 ### Тестируем (в 3м окне):

``curl localhost:8888``
получаем ``SERVER 1:8888``

``curl localhost:9999``
получаем ``SERVER 2:9999``
![alt text](img/web_python.png)

4. Установим NGINX
``` sudo apt install nginx ```
посмотрим что в конфиге  
`` cat /etc/nginx/nginx.conf ``

ищем строку include /etc/nginx/sites-enabled/*;
чтобы изменить сонфиг
``sudo nano /etc/nginx/sites-enabled/default``
проверка конфига

``nginx -t`` 

стартуем 
```systemctl start nginx```

проверяем
``curl localhost``


5. Создадим конфиг и настроим upstream

```
sudo echo "include /etc/nginx/include/upstream.inc;
server {
   listen       80;
   server_name  example-http.com;
   access_log   /var/log/nginx/example-http.com-acess.log;
   error_log    /var/log/nginx/example-http.com-error.log;
   location / {
                proxy_pass      http://example_app;

   }
}" > example-http.conf
```
копируем куда надо 

``sudo cp example-http.conf /etc/nginx/conf.d/``


создаем 
``mkdir -p /etc/nginx/include/``

Создадим файл upstream.inc

```
echo "upstream example_app {

        server 127.0.0.1:8888 weight=3;   # Больший вес указываем на основной сервер
        server 127.0.0.1:9999;

}" > upstream.inc
```

копируем
``sudo cp upstream.inc /etc/nginx/include/upstream.inc``

проверим 
``sudo nginx -t``

применяем конфиг
```
systemctl reload nginx
```
Проверяем
```
curl -H 'Host: example-http.com' http://localhost
```

![alt text](img/Round_robin.png)