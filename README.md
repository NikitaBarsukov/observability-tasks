### 1\. Выбор проекта
Для ДЗ была выбрана сборка CMS: nginx, php и mysql. Вся работа просиходит с иcпользованием Docker.
 ```bash
$ git clone https://github.com/nanoninja/docker-nginx-php-mysql.git
```
### 2\. Включение метрик
В исходные конфигурации проекта необходимо внести изменения для работы метрик т.к. изначально у образов они выключены.
    
  * #### 2.1\. Nginx 

В конфигурации `etc/nginx/default.conf`
  Включаю API метрик Nginx:

  ```clickhouse
  location /nginx_status {
      stub_status on;
      access_log off;
      allow 127.0.0.1;
      allow 172.16.0.0/12;
      deny all;
  }
  ```
* #### 2.2\. MySql
  Добавляем привилегии при инициализации MySql.
  В файле `docker-compose.yaml`
  к сервису `mysqldb` добавил скрипт иницализации:
  
  ```dockerfile
   volumes:
      - ./data/db/mysql:/var/lib/mysql
      - ./db_scripts:/docker-entrypoint-initdb.d
  ```
  
  Скрипт инициализации: `db_scripts/init.sql`
  ```sql
  CREATE USER 'exporter'@'172.16.0.0/255.240.0.0' IDENTIFIED BY 'exporterpassword' WITH MAX_USER_CONNECTIONS 3;
  GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'172.16.0.0/255.240.0.0';
  FLUSH PRIVILEGES;
  ```
### 3\. Запуск

Используя docker-compose необходимо запустить СMS:
В директории `docker-nginx-php-mysql` 
```bash
sudo docker-compose up -d
```
А так же стек экспортеров в директории `exporters`
```bash
sudo docker-compose up -d
```
Экспортеры используют дефолтную сеть СMS докера, поэтому CMS должен быть включен первым.

### 4\. Результаты

Реализован `exporters/docker-compose.yml` включающий следующий набор экспортеров:
* prom/node-exporter
* nginx/nginx-prometheus-exporter:0.4.2
* hipages/php-fpm_exporter
* prom/mysqld-exporter:v0.14.0
* prom/blackbox-exporter:latest

Prometheus не входит в docker-compose тк я поставил его локально вне докера. Его конфиг представлен в 
`GAP-1/prometheus.yaml`

Директория `updated-cms-configs` содержит конфиги из п. 2.




