## Docker images

#### Basics of a DockerFile !

 - Dockerfile is constructed in `Instruction` and `Argument` format, every Dockerfile start with the `FROM` instruction

- Every docker image is based on some other base image

- When the image is run as a container, the `ENTRYPOINT` command will be executed, hence we need to specify this command in order to run a application.

-- Docker build images in a layer structure, every instruction will create a new layer

```DockerFile
FROM Ubuntu

RUN ap-get update
RUN apt-get install python

RUN pip install flask
RUN pip install flask-mysql

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

```bash
# how to build an image
docker build Dockerfile -t abdulrehman/my-custom-app

#how to push the image
docker push abdulrehman/my-custom-app 
```