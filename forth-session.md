## lets connect
```python 
from flask import Flask
app = Flask(__name__)
import mysql.connector
import random
@app.route('/')
def hello_world():


    # mydb = mysql.connector.connect(
    # host="ypurip",
    # port="3306",
    # user="root",
    # password="pass",
    # database="mydb"
    # )


    mycursor = mydb.cursor()

#    mycursor.execute("CREATE TABLE customers (name VARCHAR(255), number int)")


    # rnd = random.randrange(20, 50)
    # print(rnd)


    mycursor.execute(f"INSERT INTO customers (name, number) VALUES ('devops' , 1)")
    mydb.commit()

    return "done"

if __name__ == '__main__':
        app.run(host='0.0.0.0', port=8000)
```