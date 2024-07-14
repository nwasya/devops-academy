``` yaml 
version: '3'
volumes:
  esdata: null
services:
  logstash:
    image: docker.elastic.co/logstash/logstash-oss:6.2.2
    restart: always
    volumes:
      - ./logstash/pipelines.yaml:/usr/share/logstash/config/pipelines.yml:ro
      - ./logstash/pipeline:/usr/share/logstash/pipeline:ro
      - ./logstash-data:/usr/share/logstash/data/
    ports:
      - '24254:24254/tcp'
      - '24254:24254/udp'
    environment:
      LS_JAVA_OPTS: "-Xmx256m -Xms256m"
    depends_on:
      - elasticsearch
    links:
      - elasticsearch
    logging:
      driver: "json-file"
      options:
         max-size: "5000m"
  elasticsearch:
    container_name: elastic
    image: 'elasticsearch:7.17.0'
    restart: always
    expose:
      - 9205
    environment:
      - discovery.type=single-node
      - http.port=9205
      - "ES_JAVA_OPTS=-Xms2g -Xmx2g"
      - ELASTIC_PASSWORD=mypass
      - xpack.security.enabled=true
    volumes:
      - 'esdata:/usr/share/elasticsearch/data'
    logging:
      driver: "json-file"
      options:
         max-size: "5000m"
  kibana:
    image: 'kibana:7.17.0'
    restart: always
    links:
      - elasticsearch
    depends_on:
      - elasticsearch
    ports:
       - '5660:5660'
    environment:
      - 'ELASTICSEARCH_HOSTS=http://elastic:9205'
      - 'ELASTICSEARCH_USERNAME=elastic'
      - 'ELASTICSEARCH_PASSWORD=mypass'
      - 'SERVER_PUBLICBASEURL=http://server-ip:5660'
      - 'SERVER_COMPRESSION_ENABLED=true'
      - 'SERVER_PORT=5660'
    logging:
      driver: "json-file"
      options:
         max-size: "5000m"
```

logstash/pipelines.yaml
``` conf 
- pipeline.id: ourapp
  path.config: "/usr/share/logstash/pipeline/ourapp.conf"
  queue.type: persisted

```
logstash/pipeline/ourapp.conf
```conf
input {
  tcp {
    port => 24254
    codec => json
  }
  udp {
    port => 24254
    codec => json
  }
}

output {
  stdout { codec => rubydebug }
  elasticsearch {
      hosts => [ "elastic:9205" ]
      user => 'elastic'
      password => 'mypass'
      index => "myindex"
      id => "myapp"
  }
}
```



``` python 
from flask import Flask
app = Flask(__name__)
import mysql.connector
import random
import logging
import logstash
import sys
@app.route('/')
def hello_world():

    host = 'your-server-ip'

    test_logger = logging.getLogger('python-logstash-logger')
    test_logger.setLevel(logging.INFO)
    test_logger.addHandler(logstash.LogstashHandler(host, 24254, version=1))
# test_logger.addHandler(logstash.TCPLogstashHandler(host, 5959, version=1))

    test_logger.error('python-logstash: test logstash error message.')
    test_logger.info('python-logstash: test logstash info message.')
    test_logger.warning('python-logstash: test logstash warning message.')

# add extra field to logstash message
    extra = {
        'test_string': 'python version: ' + repr(sys.version_info),
        'test_boolean': True,
        'test_dict': {'a': 1, 'b': 'c'},
        'test_float': 1.23,
        'test_integer': 123,
        'test_list': [1, 2, '3'],
    }
    test_logger.info('python-logstash: test extra fields', extra=extra)

    return "logs sent"

if __name__ == '__main__':
        app.run(host='0.0.0.0', port=8000)

```

```
python-logstash==0.4.8
```