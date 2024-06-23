
# 💠 Install Docker

``` bash 
sudo apt remove --yes docker docker-engine docker.io containerd runc || true
sudo apt update
sudo apt --yes --no-install-recommends install apt-transport-https ca-certificates
wget --quiet --output-document=- https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository --yes "deb [arch=$(dpkg --print-architecture)] https://download.docker.com/linux/ubuntu $(lsb_release --codename --short) stable"
sudo apt update
sudo apt --yes --no-install-recommends install docker-ce docker-ce-cli containerd.io
sudo usermod --append --groups docker "$USER"
sudo systemctl enable docker
docker --version
```

# 💠 Install Docker Compose
``` bash

sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose --version
```


# 💠 Install Nginx
``` bash
 docker run --name some-nginx -p 9090:80 -v ./index.html:/usr/share/nginx/html/index.html -d nginx
```


# 💠 Install MySQL
``` bash
 docker run --name some-mysql  -v mydata:/var/lib/mysql -d -e MYSQL_ROOT_PASSWORD=my-secret-pw mysql:8.0 
```




# 💠 Install Flask App
``` python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, Docker!'

if __name__ == '__main__':
        app.run(host='0.0.0.0', port=8000)
```

### requirements.txt

```
blinker==1.6.2
click==8.1.6
colorama==0.4.6
Flask==2.3.2
itsdangerous==2.1.2
Jinja2==3.1.2
MarkupSafe==2.1.3
Werkzeug==2.3.6
```

### Dockerfile
``` docker
# syntax=docker/dockerfile:1.4
FROM python:3.11.4
WORKDIR /app/devops
COPY ./requirements.txt .
COPY . .
RUN pip3 install -r requirements.txt
EXPOSE 8000
ENTRYPOINT ["python3"]
CMD ["app.py"]
```


### docker-compose.yml
``` yaml
services:
  flast-app:
    build:
      context: .
    # flask requires SIGINT to stop gracefully
    # (default stop signal from Compose is SIGTERM)
    stop_signal: SIGINT
    ports:
      - '8000:8000'
```
