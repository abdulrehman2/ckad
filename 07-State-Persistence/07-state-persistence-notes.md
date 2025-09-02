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

- Layer 1. Base Ubuntu layer (120 MB)
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


> When we run a container based on image created using above docker file, the docker will create a new layer called `Container Layer` on top of the other layers. This layer will hold the container related data (logs etc.) and this layer will exist until the docker container is destroyed.

The contents of image layer are not editable, if we want to edit a file that belongs to the image layer, docker will first create a copy and then change/write on it. This is called `COPY-ON-WRITE`

What if we want to preserve the changes that are done in the `Container Layer` ? We need to create a volume :)


## 1. Volume drivers

```bash
docker volume create data_volume
```

the above command will create a new volume and create a directory inside

- var/lib/docker
  - volumes
     - data_volume

if we want to mount this volume to a container , we can do so using command

```bash
# we create a container of mysql and mount the directory of container i.e. /var/lib/mysql to a volume we created earlier
docker run -v data_volume:/var/lib/mysql mysql
```
Now even if the container is destroyed, the data stored by mysql database will persist, because it lives outside the container and in the host.

If the volume does not exist and we run the above command, docker will automatically create a new volume for that, for example

```bash
# data_volume2 does not exist , but docker will create it automatically
docker run -v data_volume2:/var/lib/mysql mysql
```

the above binding is called `volume mount`, i.e. we bind the volume directory on docker host to a directory of container. What if we want to bind a different directory of host instead of /var/lib/docker/volumes ?
We can do it using the `bind mount`. this allow us to mount any directory on docker host to a directory in docker, for this to work, we need to provide complete path of the directory when creating the container.

```bash
# here we provide the complete path that we need to mount i.e. /data/mysql
docker run - v /data/mysql:/var/lib/mysql mysql 
```

`-v` is the old way of doing things, new syntax allow us to provide key value pair with `--mount` attribute

```bash
docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```

## 2. Storage drivers
So who does all the , maintaining a layered architecture, creating writable layers, moving files across layers, enable copy-write operations ? This is done by the underlying storage driver we use. Storage drivers depend upon the underlying OS. For Ubuntu we have AUFS.
Some of the common drivers are 

- AUFS
- ZFS
- Device Mapper
- Overlay

Keep in mind the volumes are not managed by storage drivers, instead these are managed by `volume driver plugin`. The default volume driver plugin is called `local`.

Some of the popular volume driver plugins are 
- Azure file storage
- Convoy
- Flocker
- RexRay
- DigitalOcean Block Storage
- VMware vSphere Storage