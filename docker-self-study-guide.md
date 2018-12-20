# Docker self-study

The goal of this document is to provide you with a self-study path explaining some common and basic things about docker. I assume docker is already installed, If you still need to do that, please follow the basic installation instructions [here](installation.md).

# Learning the basics

### How to see which docker images are available

Typicaly images are available in a central repository. The public one is https://hub.docker.com.

It is possible to set up a local repository of your own. Most clients will do this for security reasons.

## Test with the alpine image

The first thing to do is getting a simple linux docker container and echo 'hello world' to the standard output. This means pulling the image and running

### Pull the image

To pull the alpine docker image to your local machine using the following command:

```bash
$ docker pull alpine

Using default tag: latest
latest: Pulling from library/alpine
4fe2ade4980c: Already exists
Digest: sha256:621c2f39f8133acb8e64023a94dbdf0d5ca81896102b9e57c0dc184cadaf5528
Status: Downloaded newer image for alpine:latest
```

You can see that, since we haven't specified which version tag of the image we wanted to use, it pulls the 'latest' image. By default, docker pull uses hub.docker.com as the source.

The long hash codes are the 'names' of docker image layers being pulled down. In the above example, because I've pulled an alpine image before, there was a correct version of the layer already existing and so it did not need to download it.

You can see which images are available locally using the following command: 

```bash
$ docker images

REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
jenkinsci/blueocean   latest              dfc2410d706e        2 months ago        468MB
alpine                latest              196d12cf6ab1        3 months ago        4.41MB
```

### Run the image with echo

To actually do something with the image, you can start it with the following command:

```bash
$ docker run alpine echo "Hello World"
Hello World
```

It starts the image and executes echo command inside it.
To check if the image is still running, use

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS                                              NAMES
c52fc5412337        alpine              "echo 'Hello World'"     50 seconds ago      Exited (0) 48 seconds ago                                                      sad_mirzakhani
614eb2ae1837        dfc2410d706e        "/sbin/tini -- /usr/…"   2 months ago        Exited (255) 2 months ago   0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp   myjenkins
```

The first command asks for the currently running containers, but there are none. The second command asks for running and recently run containers.

### Make a custom image extending the alpine image

In the previous section we ran the Hello World command using the commandline. Now we are going to make our own fancy Hello World image that does exactly the same.

Create a file called ```Dockerfile``` with your favorite editor and add the following:

```Dockerfile
FROM alpine:latest
CMD echo "Hello World"
```

We have to use [`vi`](https://pangea.stanford.edu/computing/unix/editing/viquickref.pdf) to create this file.

Lets see what is in there:

- FROM is used to specify the docker image to start from.
- CMD is used to specify the command to execute when starting this docker image. 

You can build and run this custom image in the following way:

```bash
$ docker build . -t fancy-Hello World:1

Sending build context to Docker daemon  731.6kB
Step 1/2 : FROM alpine:latest
 ---> 196d12cf6ab1
Step 2/2 : CMD echo "Hello World"
 ---> Running in 5142856aa0db
Removing intermediate container 5142856aa0db
 ---> 854ee762cd1e
Successfully built 854ee762cd1e
Successfully tagged fancy-hello-world:1

$ docker run fancy-hello-world:1

Hello World
```

# Create Hello World webpage

## Manually create the Hello World page

### Pull the Nginx image

```bash
$ docker pull nginx
```

### Run Nginx
```bash
$ docker run --rm --name nginx -d -p <http_port>:80 nginx
```

Nginx index page should be available at `<docker_host>:<http_port>`, for example http://localhost:1180

### Create the hello world page

```bash
$ docker exec -it nginx /bin/bash
```

Inside the container:

```bash
$ cd /usr/share/nginx/html
```

Since no editor is avaialbe in the nginx image we have to use a trick to add a new file.

```bash
$ cat > hello.html
```

Copy-paste the following snippet (with the last blank line) and then type `ctrl+c`.

```html
<html>
    <body>
        <h1>Hello World!</h1>
    </body>
</html>

```

Now try to access the page in your browser at `<docker_host>:<http_port>/hello.html`, for example http://locahost:1180/hello.html

### Stop the container

```bash
$ docker kill nginx
```

Now try to run the nginx container again.

```bash
$ docker run --rm --name nginx -d -p <http_port>:80 nginx
```

- What happens if you try to access the hello world page?
- Why?

## Repeatable creation of Hello World page

### Create a custom docker image

Create a `hello.html` with this content:

```html
<html>
    <body>
        <h1>Hello World!</h1>
    </body>
</html>
```

Create a `Dockerfile` with this content:

```Dockerfile
FROM nginx:latest

COPY hello.html /usr/share/nginx/html
```

Now we can build a new docker image

```bash
$ docker build . -t nginx-hello-world:2
```

```bash
$ docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
nginx-hello-world     2                   e821a5e73ef0        14 seconds ago      109MB
fancy-hello-world     1                   854ee762cd1e        7 minutes ago       4.41MB
nginx                 latest              568c4670fa80        3 weeks ago         109MB
jenkinsci/blueocean   latest              dfc2410d706e        2 months ago        468MB
alpine                latest              196d12cf6ab1        3 months ago        4.41MB
```

### Run the nginx hello world image 

```bash
$ docker run --rm --name nginx-hello-world -d -p <http_port>:80 nginx-hello-world
```

```bash
$ docker logs -f nginx-hello-world
```

### Stop the container

```bash
$ docker kill nginx-hello-world
```

```bash
$ docker run --rm --name nginx-hello-world -d -p <http_port>:80 nginx-hello-world
```

- What happen if you try to access the hello world page?
- Why?

# Docker-compose

[Docker Compose]() provides some nice support on top of the docker runtime to make starting and handling an environmnent with (multiple) docker(s) a bit more easy. Instead of using a lot of parameters on the commands, docker-compose uses a file to describe the whole environment in a file docker-compose.yml.

## Example of Nginx with docker-compose

Lets start with adding a file docker-compose.yml at the same location as the Dockerfile and hello.html file (example of Nginx above).

```docker-compose
version: '3'

services:
  nginx:
    build: .
    ports:
      - "8080:80"

```

- The first line indicates the version of compose we are using
- Services indicates which services we will declare, in our case only nginx.
- "build: ." indicates to start from a Dockerfile in the current directory.
- Ports indicate which ports to expose. 

Execute the following command:

```bash
$ docker-compose up
Creating network "compose-example_default" with the default driver
Building nginx
Step 1/2 : FROM cipadmin-prod.be.net.intra:8085/nginx:1.13.6
 ---> 40960efd7b8f
Step 2/2 : COPY hello.html /usr/share/nginx/html
 ---> 3d343140ca24
Successfully built 3d343140ca24
Successfully tagged compose-example_nginx:latest
WARNING: Image for service nginx was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Creating compose-example_nginx_1 ... done
Attaching to compose-example_nginx_1
```

Now you can find it at [http://localhost:8080/hello.html](http://localhost:8080/hello.html). 

To stop it, you can use Crtl-C.

### Some extra tricks

Start it in the background

```bash
$ docker-compose up -d
```

See what is running

```bash
$ docker ps
```

Normally stopping the service

```bash
$ docker-compose down
```

- Can you start the service again in background and stop it without using the docker-compose command?


### Nginx with volume

Until now we build the html page inside the docker. We can also mount data from our local computer inside the docker immediatly. 

To test this make a new directory, make the file docker-compose.yml in this directory and give it the following content.

```docker-compose
version: '3'

services:
  nginx:
    image: nginx
    ports:
      - "8080:80"
    volumes:
      - ./hello.html:/usr/share/nginx/html/hello.html
```

Additionally ensure there is a file hello.html is in the same directory. Then start again.

Questions:
- What happens if we change the text in hello.html? Why?

# References

- [Docker Commands](https://docs.docker.com/reference/)

# Credits

My colleague coaches at BNP Paribas Fortis wrote the original version of this document for a training there. I'm indebted to them for the use of some of their materials here:

[Nelis Boucke (Wemanity)](mailto:nelis.boucke@bnpparibasfortis.be) / [@nelisboucke](http://twitter.com/nelisboucke)<br>[Matteo Pierro (Wemanity)](mailto:matteo.pierro@bnpparibasfortis.be) / [@matteo_pierro](http://twitter.com/@matteo_pierro)