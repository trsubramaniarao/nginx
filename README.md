# nginx

This is to test nginx conf with golang services as backend.
golang services - https://github.com/trsubramaniarao/docker-gs-ping

### Clone the repo
```
git clone git@github.com:trsubramaniarao/docker-gs-ping.git
git checkout uploads
```

### Docker image, container, network for golang services.
```
docker build -t docker-gs-ping:multistage -f Dockerfile.multistage .

docker container run --hostname goapp1 --name goapp1 -d docker-gs-ping:multistage
docker container run --hostname goapp2 --name goapp2 -d docker-gs-ping:multistage
docker container run --hostname goapp3 --name goapp3 -d docker-gs-ping:multistage

docker network create lbnetnginx
docker network connect lbnetnginx goapp1
docker network connect lbnetnginx goapp2
docker network connect lbnetnginx goapp3
```

### Run nginx with nginx.conf
```
# This will fail because nginx is not in the lbnetnginx
docker container run --name nginx --hostname ng1 -p 80:8080 -v /HOST_PATH/nginx.conf:/etc/nginx/nginx.conf -d nginx

# Add the nginx container to the network.
docker network connect lbnetnginx nginx

# Start.
docker start nginx
```

### Cleanup
```
docker container stop goapp1
docker container stop goapp2
docker container stop goapp3

docker container rm goapp1
docker container rm goapp2
docker container rm goapp3

docker image rm docker-gs-ping:multistage

# Enough to stop and remove container. No need to remove image because the image is from nginx.
docker container stop nginx
docker container rm nginx

docker network rm lbnetnginx
```

## curl commands to test
```
-- 1 MB limitation. Should succeed.
curl -v --location --request POST 'http://localhost/upload' \
--form 'files=@/HOST_PATH/v3test_attach1.txt'

-- 1 MB limitation. Should fail.
curl -v --location --request POST 'http://localhost/upload' \
--form 'files=@/HOST_PATH/size_above20MB.zip'

-- 100 MB limitation. Should succeed.
curl -v --location --request POST 'http://localhost/huge' \
--form 'files=@/HOST_PATH/v3test_attach1.txt'

-- 100 MB limitation. Should succeed. If there is no consumer then it may return response before consuming request and hence may result in 502.
curl -v --location --request POST 'http://localhost/huge' \
--form 'files=@/HOST_PATH/size_above20MB.zip'
```
