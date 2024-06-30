## lets run a gitlab instance!

``` yaml
version: '3.5'
services:
  gitlab:
    image: gitlab/gitlab-ce:16.11.0-ce.0
    container_name: gitlab
    restart: always
    hostname: academy.girt.ir
    networks:
      - web2
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://academy.girt.ir'
        nginx['listen_port'] = 80
        nginx['listen_https'] = false
        nginx['http2_enabled'] = false

        nginx['proxy_set_headers'] = {
          "Host" => "$$http_host",
          "X-Real-IP" => "$$remote_addr",
          "X-Forwarded-For" => "$$proxy_add_x_forwarded_for",
          "X-Forwarded-Proto" => "https",
          "X-Forwarded-Ssl" => "on"
        }

        gitlab_rails['gitlab_ssh_host'] = 'academy.girt.ir'

        gitlab_rails['smtp_enable'] = true
        gitlab_rails['smtp_address'] = "smtp.gmail.com"
        gitlab_rails['smtp_port'] = 587
        gitlab_rails['smtp_user_name'] = "test.tech1@gmail.com"
        gitlab_rails['smtp_password'] = "test.tech1@gmail.com"
        gitlab_rails['smtp_domain'] = "smtp.gmail.com"
        gitlab_rails['smtp_authentication'] = "login"
        gitlab_rails['smtp_enable_starttls_auto'] = true
        gitlab_rails['smtp_tls'] = false
        gitlab_rails['smtp_openssl_verify_mode'] = 'peer'

        registry_external_url 'https://academy-registry.girt.ir'
        registry_nginx['listen_port'] = 5100
        registry_nginx['listen_https'] = false

        #registry['storage'] = {
        #  's3' => {
        #    'accesskey' => 'NCISYXQ8RU96MSELKITP',
        #    'secretkey' => 'GJr4Me7Svv9qKiglALCLH2vNaJMo9i0dqCONaOcA',
        #    'bucket' => 'girt-registry',
        #    'region' => 'eu-central-1',
        #    'regionendpoint' => 'https://s3.eu-central-1.wasabisys.com',
        #    'path_style' => true
        #  }
        #}
        #        registry['storage'] = {
        #          's3' => {
        #            'accesskey' => '143f55111e904c879dbfefe2c23433b5',
        #            'secretkey' => 'b5d07b338e8941859326b6791ef71c24',
        #            'bucket' => 'girt-registry',
        #            'region' => 'DE',
        #            'regionendpoint' => 'https://s3.de.cloud.ovh.net',
        #            'path_style' => true
        #          }
        #        }
        #


        registry['storage'] = {
          's3' => {
            'accesskey' => 'access-key',
            'secretkey' => 'secret-key',
            'bucket' => 'bucket-name',
            'region' => 'us-east-1',
            'regionendpoint' => 'http://my-minio:9000',
            'path_style' => true
          }
        }



        registry_nginx['proxy_set_headers'] = {
          "Host" => "$$http_host",
          "X-Real-IP" => "$$remote_addr",
          "     X-Forwarded-For" => "$$proxy_add_x_forwarded_for",
          "X-Forwarded-Proto" => "https",
          "X-Forwarded-Ssl" => "on"
        }

        gitlab_rails['omniauth_providers'] = [
          {
            "name" => "gitlab",
            "app_id" => "6494ba44a0f2471ce75194259614a538adc532316895b746dc3f515bf40bf563",
            "app_secret" => "6c763c8b173e5798a7645c37cc0bc26af0f83cbe4b5b5342ad8970877ba64b38",
            "args" => { "scope" => "api" }
          }
        ]
        prometheus_monitoring['enable'] = false
        sidekiq['concurrency'] = 10
        postgresql['shared_buffers'] = "256MB"
        puma['worker_processes'] = 5

        pages_external_url 'http://keeps.ir'
        gitlab_pages['enable'] = true
        pages_nginx['listen_port'] = 5200
        pages_nginx['listen_https'] = false

        pages_nginx['proxy_set_headers'] = {
          "Host" => "$$http_host",
          "X-Real-IP" => "$$remote_addr",
          "X-Forwarded-For" => "$$proxy_add_x_forwarded_for",
        }

        gitlab_pages['inplace_chroot'] = true
        gitlab_pages['external_http'] = ['gitlab:5201']
        gitlab_pages['access_control'] = false
        gitlab_pages['internal_gitlab_server'] = "http://gitlab"


    volumes:
      - ./config:/etc/gitlab
      - ./logs:/var/log/gitlab
      - ./gitlab-data:/var/opt/gitlab
    ports:
      # Feel free to map this to a different port if that one is in use already
      - "22:22"
      - "5100:5100"
      - "7171:80"
      - "13456:80"
    labels:
      - traefik.enable=true
      - "traefik.docker.network=web2"

      # Gitlab Service
      - "traefik.http.routers.gitlab.entrypoints=web,websecure"
      - "traefik.http.routers.gitlab.rule=Host(`academy.girt.ir` )"
      - "traefik.http.routers.gitlab.service=gitlab"
      - "traefik.http.services.gitlab.loadbalancer.server.port=80"

      # Gitlab Registry service
      - traefik.http.routers.registry.rule=Host(`academy-registry.girt.ir`)
      - traefik.http.routers.registry.service=registry
      - traefik.http.routers.registry.entrypoints=web,websecure
      - traefik.http.services.registry.loadbalancer.server.port=5100

      # Gitlab Pages service
      - traefik.http.routers.pages.rule=HostRegexp(`{subdomain:[a-zA-Z0-9_.-]+}.keeps.ir`)
      - traefik.http.routers.pages.service=pages
      - traefik.http.routers.pages.entrypoints=web,websecure
      - traefik.http.services.pages.loadbalancer.server.port=5200
      - "traefik.http.routers.pages.middlewares=keeps@file"
networks:
  web2:
    external: true


```

## we need a storage!


``` yaml 
version: '3'

services:
  minio:
    container_name: minio
    image: minio/minio
    restart: always
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - ./data:/data
    environment:
      MINIO_ROOT_USER: git
      MINIO_ROOT_PASSWORD: hellomysecretpass
    command:  server --console-address ":9001" /data

```

## we need also a webserver!

``` yaml 
version: "3.9"

services:
  traefik:
    image: traefik:latest
    container_name: traefik
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    networks:
      - web2
    ports:
      - 80:80
      - 443:443
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/resolv.conf:/etc/resolv.conf
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik.yml:/traefik.yml:ro

networks:
  web2:
    external: true

```


``` yaml
entryPoints:
  web:
    address: ':80'
  websecure:
    address: ':443'
log:
  level: DEBUG
providers:
  docker:
    watch: true
  
```

### you will need this for configing your apps

``` yaml
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=proxy"
      - "traefik.http.routers.myapp.entrypoints=web"
      - "traefik.http.routers.myapp.rule=(Host(``))"
      - "traefik.http.services.myapp.loadbalancer.server.port=80"
```


## lets try our first job
``` yaml

test-job:
  image: docker:20
  script:
    - echo "hello world"
    - pwd
    - ls
  only:
    - main

```




## oops. no runner? lets have a runner.
``` yaml
version: '3.3'
services:
  gitlab-runner:
    image: gitlab/gitlab-runner:alpine
    restart: always
    container_name: gitlab-runner
    hostname: gitlab-runner
    volumes:
     - ./gitlab-runner:/etc/gitlab-runner
     - /var/run/docker.sock:/var/run/docker.sock
```



``` yaml

concurrent = 8
check_interval = 0
shutdown_timeout = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "gitlab-runner"
  url = "https://academy.girt.ir"
  id = 52
  token = "your-token"
  token_obtained_at = 2023-11-11T17:37:10Z
  token_expires_at = 0001-01-01T00:00:00Z
  executor = "docker"
  environment = ["DOCKER_DRIVER=overlay2", "DOCKER_TLS_CERTDIR="]
  pre_build_script = "export DOCKER_HOST=tcp://docker:2375"

  [runners.cache]
    MaxUploadedArchiveSize = 0
  [runners.docker]
    tls_verify = false
    image = "ruby:2.7"
    privileged = true
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0
    network_mtu = 0
```





