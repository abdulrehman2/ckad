# Docker storage

when we install docker on a host, the docker store it's files like this

- var/lib/docker
  - aufs
  - containers
  - image
  - volumes

All files related to a container will reside inside the `containers` folder, same for `images`.

## Layered architecture
When we create a docker image, from a docker file, each instruction in docker file acts a layer, where each instruction create a layer on top of the last layer.

```Dockerfile
From Ubuntu

RUN apt-get update && apt-get -y install python

RUN pip install flask flask-mysql

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask
```

```bash
docker build Dockerfile -t abdul/my-custom-app
```

Running above will create following layers

- Layer 1. Base ubunut layer (120 MB)
- Layer 2. Changes in apt packages (306 MB)
- Layer 3. Changes in pip packages (6.3 MB)
- Layer 4. Source code  (220 B)
- Layer 5. Update EntryPoint (0 B)


Now lets say we have another `app 2`, it has following docker file. 

```Dockerfile
From Ubuntu

RUN apt-get update && apt-get -y install python

RUN pip install flask flask-mysql

COPY app2.py /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app2.py flask
```
It will reuse the first 3 layers as they are same for app 1, so it will not build it and reuse it from other app. However it will build the layer 4 and 5 since source code is different.


> When we run a container based on image created using above docker file, the docker will create a new layer called `Container Layer` on top of the other layers. This layer will hold the container related data (logs etc.) and this layer will exist untill the docker container is destroyed.

The contents of image layer are not editable, if we want to edit a file that belongs to the image layer, docker will first create a copy and then change/write on it. This is called `COPY-ON-WRITE`

What if we want to preserve the changes that are done in the `Container Layer` ? We need to create a volume :)

```bash
docker volume create data_volume
```

the above command will create a new volume and create a directory inside

- var/lib/docker
  - volumes
     - data_volume


## 1. Storage drivers


## 2. Volume drivers