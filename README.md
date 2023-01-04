# nginx

This is to test nginx conf with golang services as backend.
golang services - https://github.com/trsubramaniarao/docker-gs-ping

### Docker image, container, network for golang services.
```
docker build -t docker-gs-ping:multistage -f Dockerfile.multistage .

docker run --hostname goapp1 --name goapp1 -p -d docker-gs-ping:multistage
docker run --hostname goapp2 --name goapp2 -p -d docker-gs-ping:multistage
docker run --hostname goapp3 --name goapp3 -p -d docker-gs-ping:multistage

docker network connect raonginxnet nginx
docker network connect raonginxnet goapp1
docker network connect raonginxnet goapp2
docker network connect raonginxnet goapp3
```

### Run nginx with nginx.conf
```
docker run --name nginx --hostname ng1 -p 80:8080 -v /HOST_PATH/nginx.conf:/etc/nginx/nginx.conf nginx
docker network connect raonginxnet goapp1
docker start nginx
```
