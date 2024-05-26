### 1\. Выбор проекта
Для ДЗ было выбрано хранилище метрик Thanos c MiniO.

### 2\. Правки из ДЗ 1.

Для удобства Prometheus был перенесен из локальной среды в докер.
На его основе сделаны два инстанса с лейблами «prod1» и «prod2» соотвественно.  

### 4\. Результаты

Реализован `thanos/docker-compose.yml` включающий следующий набор из образа quay.io/thanos/thanos:v0.31.0:
* thanos-query-frontend
* thanos-querier
* thanos-store-gateway
* thanos-compactor
* thanos-ruler
* thanos-bucket-web
* и по thanos-sidecar на каждый Prometheus.

Дополинительно развернут minio, куда будут складываться метрики. Его конфигурация для Thanos
представлена в `thanos/bucket_config.yaml`

Согласно заданию необходимо хранение метрик 14 дней:

Для сервиса Compactor выставлено:
```clickhouse
            - "--retention.resolution-raw=15d"
            - "--retention.resolution-5m=15d"
            - "--retention.resolution-1h=15d"
```