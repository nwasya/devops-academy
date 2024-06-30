## lets connect
```python 
from flask import Flask
app = Flask(__name__)
import mysql.connector
import random
@app.route('/')
def hello_world():

    mydb = mysql.connector.connect(
    host="some-mysql",
    port="3306",
    user="root",
    password="pass",
    database="devops"
    )


    mycursor = mydb.cursor()

    mycursor.execute(f"INSERT INTO customers (name, number) VALUES ('devops' , 2)")
    mydb.commit()

    return "done"

if __name__ == '__main__':
        app.run(host='0.0.0.0', port=8000)
```



## docker compose files used to test volume and network functionality.


``` yaml 
services:
  flask-app:
    build:
      context: .
    # flask requires SIGINT to stop gracefully
    # (default stop signal from Compose is SIGTERM)
    stop_signal: SIGINT
    ports:
      - '3030:8000'
    networks:
      - app
networks:
  app:
    external: true
```


``` yaml 
services:
  mysqlapp:
    image: mysql:8.0
    volumes:
      - devops-mysql:/var/lib/mysql
    container_name: some-mysql
    networks:
      - app
volumes:
  devops-mysql:
    name: devops-mysql
networks:
  app:
    external: true

```