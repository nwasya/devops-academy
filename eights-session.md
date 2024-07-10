## rabbit
``` yaml
version: '3.8'
services:
  
  rabbitmq:
    restart: always
    image: rabbitmq:3.11.24-management-alpine
    container_name: rabbit
    hostname: rabbit-host
    cpus: "1"
    mem_limit: "2000M"
    ports:
        - 5672:5672
        - 15672:15672
    volumes:
        - rabbit-data:/var/lib/rabbitmq/
  

volumes:
  rabbit-data:

```


## mongoDB

``` yaml 
version: '3.8'
services:
  mongodb:
    image: mongo:5.0.23
    ports:
      - '27017:27017'
    command: --auth

    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
    volumes:
      - dbdata6:/data/db


  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: root
      ME_CONFIG_MONGODB_ADMINPASSWORD: example
      ME_CONFIG_MONGODB_URL: mongodb://root:example@mongo:27017/
      ME_CONFIG_BASICAUTH: false
volumes:
  dbdata6:



```

``` bash

- "./mysql.cnf:/etc/mysql/conf.d/mysql.cnf"

CREATE USER 'replicator'@'%' IDENTIFIED WITH mysql_native_password BY 'password';GRANT REPLICATION SLAVE ON *.* TO 'replicator'@'%';

CHANGE MASTER TO MASTER_HOST='x.x.x.x', MASTER_USER='replicator', MASTER_PASSWORD='my-pass', MASTER_LOG_FILE='binlog.000003', MASTER_LOG_POS= 157,MASTER_PORT=4141;

```


``` conf
[mysqld]
relay_log = relay-bin
log_bin = ON
server_id = 7
read_only = 1
binlog_format = ROW
max_binlog_size = 500M
sync_binlog = 0
#expire-logs-days = 1
binlog_expire_logs_seconds = 20000
slow_query_log   = 1
slave_skip_errors = all
# Boost Performance
max_connections=3000
replica_parallel_workers = 48
innodb_flush_log_at_trx_commit = 2
innodb_write_io_threads = 16
innodb_read_io_threads = 16
binlog_cache_size = 9332768
innodb_buffer_pool_size=9134217728
innodb_buffer_pool_instances=2
```