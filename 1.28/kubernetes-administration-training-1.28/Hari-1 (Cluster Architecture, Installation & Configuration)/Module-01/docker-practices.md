# Docker
### install docker on master node
You can refers this link for latest version [docker on ubuntu](https://docs.docker.com/engine/install/ubuntu/)<br>
for development and testing environtment you can use convenience script
``` 
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
```

### Containerize an application
Download the image to local
```
docker pull nginx 
``` 
Running the container on port 80
```
docker run -d -p 80:80 --name ct-name nginx
``` 
Check container status
```
docker ps 
``` 

### Update the application
Execution shell on container
```
docker exec -it ct-name /bin/bash
```

Copy custom content to container
```
docker cp index.html ct-name:destination-path
``` 

### Build your own application
Create directory
```
mkdir my-image
cd my-image
mkdir apps
```

create index file
```
vim apps/index.html
```

add this
```
<title>Training Kubernetes</title>
<h1>Belajar Kubernetes Hari 1</h1>
```

Create Dockerfile
```
vim Dockerfile 
```

Build image
```
docker build -t image-name dockerfile-path
``` 

Check the image
```
docker images 
```

Running a own image
```
docker run -d -p 8080:80 --name ct-name image-name
``` 

### Share the application
Create account dockerhub, you can refers this link [Dockerhub](https://hub.docker.com/)

Login dockerhub account on docker
```
docker login 
```
Check info docker 
```
docker info 
``` 
Tag name image with repository name on dockerhub 
```
docker tag old-name-image repository/new-name-image
``` 
Push image to dockerhub 
```
docker push repository/new-name-image
``` 