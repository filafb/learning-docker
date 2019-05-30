#Learning Docker => https://www.linkedin.com/learning/learning-docker-2/run-processes-in-containers

Docker != VM

- Image: Executable package
- Container: A self-contained sealed unit of software. A runtime instance of an image
  - code
  - configs
  - processes
  - networking
  - dependencies
  - OS

`docker info`
`docker run -ti --rm ubuntu bash`
-ti: terminal interactive // --rm: remove when it's done // ubuntu: image name // bash: interatice shell

## The Docker flow

  Image === `docker run` ====> Running Container* ===> Stopped Container === `docker commit` ===> New Image
  * Two different containeres made from the same image are their own container (doesnt share files, programs, when installed in the container)

  `docker ps -l` see latest container to exit

  `docker ps -a` see all container running or stopped

  Making a new image, from a container: `docker commit <current_container_name> <target_image_name>` // This does commit & tag

### To see how info is displayed in the terminal, change config in `~/.docker/config.json`

## Running things in docker

`docker run`
  - containers have a main process
  - the container stops when that process stops
  - containers have names

  commands:
   `--rm` means, delete this container when exits
   `-d` for detached, start and run in the background - to see it: `docker attach <container_name>`
   `control-p control-q` detach when start in attach mode
   `docker exec -ti <container_name> <process>` - It will execute the process in a container already running.

## Looking at container output
  - `docker logs`

## Stopping and removing containers
  - listing contaniners: `docker container ls -a`
  `docker kill container_name` - container goes to stop mode
  `docker rm container_name` - remove container
  - remove all containers: `docker rm $(docker ps -a -q)`

## Private container network - Containers talking to each other
  - You decide whom talking to whom
  - `docker run --rm -ti -p 45678:45678 -p 45679:45679 --name echo-server ubuntu:14.04 bash`
  `nc -lp 45678 | nc -lp 45679`. We can run a different container and connect to these ports
  We can also pass only the inside port.

## Connecting between containers
  - A more straightforward way to connect containers - has its pitfals:
  `docker run --rm -ti --name server ubuntu:14.04 bash` -- Give a name to the container
  `docker run --rm -ti --link server --name client ubuntu:14.04 bash` -- *Link* the container by name
  then, connect both:
    - on the server: `nc -lp 1234`
    - on the client: `nc server 1234`
  - Can be risk if the containers are not running sync, and in the same machine

## Docker private networks
  - `docker network create <name>`
  eg: `docker network create example`, then:
  - `docker run --rm -ti --net=example --name server ubuntu:14.04 bash`
  - `docker run --r -ti --link server --net=example --name client ubuntu:14.04 bash`

## Images
  - `docker images` - list the images I have downloaded
  - Getting images: `docker pull`, which is ran automatically by `docker run`
  - removing images: `docker rmi <image_id>` or `docker rmi <repository>`
  - removing all images: `docker rmi $(docker images -q)`
  - removing dangling images <none>:<none> when listed through `docker images`:
    - `docker rmi $(docker images -f "dangling=true" -q)`
    These are images crated on top of other images that were updated. Dangling iamges: Intermerdiate unused images. Need to be prune

## Volumes
  - Storing and share data
  - Two types:
    - Persistent
    - Ephemeral
  - Not part of the images
  _Sharing folders with the host:_
  - `docker run -ti -v <absolute_path>:/<yourfoldername> ubuntu bash`
  `docker run -ti -v /Users/fila/projects/learning-docker/example:/shared-folder ubuntu bash`
    - *the colon is important!*
  - create a file inside <yourfoldername> and it will be available in your local
  Sharing files is the same. File needs to exist, otherwise it will assume it's a folder

  _Sharing Data between containers:_ ephemeral
`volumes-from`
- start a docker sharing data:
`docker run -ti -v /shared-data ubuntu bash`
- start a docker using it:
`docker run -ti --volumes-from <container_name> ubuntu bash`
The data will be persistent and can be accessed from a third container, even when the first one is gone. However, data is gone when all containers using is are gone

## Dockerfile
  - A small "program" to create an image // A portable image
  `docker build -t <image_name> .`
    -t = tag
    . = place for the dockerfile
  When finishes, the result will be in your local docker registry, able to run with `docker run`

- Each steps produce its own image
- The previous image is unchanged
- It does not edit the state from the previous line -> eg. if you donwload a file, modified it, and delete it, do in one line. Otherwise, the size of your final image will be huge.
- Docker caches results. Put the code that change most in the end of the program.
- Processes you start on one line will not be running on the next line
- ENV: environment variables are common in different lines

*Documentation:* https://docs.docker.com/

###Docker Statements
  - *FROM*
    - Which image to download and start from
    - Must be the first command
    - It's ok to put multiples on the dockerfile => It means that the Dockerfile produces more than one image

  - *MAINTAINER*
    - Defines the author of this Dockerfile

  - *RUN*
    - Runs the command line, waits for it to finish, and saves the result

  - *ADD*
    - Adds local files
    - Adds the contents of tar archives (.tar.gz)
    - Works with URLs

  - *ENV*
    - Sets environment variables
    - Both during the build and when running the result

  - *ENTRYPOINT* and *CMD*
    - Entrypoint: specifies the start of the command to run
    - CMD: Specifies the whole commmand to run

  - *EXPOSE*
    - Maps a port into the container

  - *VOLUME*
    - Defines shared or ephemeral volumes
    Two arguments - shared:
    `VOLUME ["/host/path/" "/container/path/]`
    One argument - ephemeral crate a file that can be inherited by later containers:
    `VOLUME ["/shared-data"]`
    - Avoid defining shared folders on Dockerfile -> It means it will run only on your computer

  - *WORKDIR*
    - Sets the directory the container starts in

  - *USER*
    - Sets which user the container will run as

##Multi-Stage builds
  - in the example @ ./multi-state, we create a image from Ubuntu, but we don't need all the functionalities. So, later, we copy what we need from builder to the new image, then run what we need

## App Hierarchy
  - stack (interactions of all the services)
  - Services (define how containers behave in production)
  - Containers

### Created a python app on ./docker-docs
  To build the image, based on Dockerfile:
  - `docker build --tag=friendlyhello . `
  To run to image (create a container):
  - `docker run -p 4000:80 friendlyhello`
    - To run the app in the background: `-d`
  - Visit localhost:4000
  - To stop it:
  - `docker stop <container_id>`

## Share your image
  - Registry: a collection of repositories
  - Repository: collection of images

  * Tag the image
  `docker tag <image_name> <username/repository:tag>`
  * Publish the image
  `docker push <username/repository:tag>`
  * Run the image from the remote repository. If the image isn't available locally, Docker pulls it
  `docker run -p 4000:80 <username/repository:tag>`
