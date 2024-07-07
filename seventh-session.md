
an example of traefik labels.

``` yaml
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=proxy"
      - "traefik.http.routers.test.entrypoints=web"
      - "traefik.http.routers.test.rule=(Host(`your-domain`))"
      - "traefik.http.services.test.loadbalancer.server.port=80"
```


lets test minio:
```bash
curl https://dl.min.io/client/mc/release/linux-amd64/mc \
  --create-dirs \
  -o $HOME/minio-binaries/mc

chmod +x $HOME/minio-binaries/mc

export PATH=$PATH:$HOME/minio-binaries/
mc --help
```

add your alias:
```bash
mc alias set myminio http://localhost:9000  your-access-key your-secret-key
```

```bash
mc cp --recursive ./your-file  myminio/your-bucket/
```

Get backup routine:
``` bash
0 * * * * bash /usr/bin/backup.sh >> /var/log/backup/$(date --iso-8601).log
```


```bash 

#!/bin/bash
mkdir /tmp/db
cd /tmp/db
file=$(date --iso-8601=minutes)-app.sql
docker exec -e MYSQL_PWD='pass' container_name  mysqldump --all-databases > ${file}
#ls -al /tmp/db

echo "start ziping file ..."
gzip ${file}

echo "uploading to storage ..."
/home/deployer/minio-binaries/mc cp ${file}.gz myminio/backup/app/${file}.gz
echo "remove files ..."
rm ${file}.gz
echo "remove backup files older than 3 days"
/home/deployer/minio-binaries/mc rm --recursive --force --older-than 7d myminio/backup/app

echo "done"


```