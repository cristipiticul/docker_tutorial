Course: Docker & Kubernetes: The Practical Guide [2022 Edition] from udemy

Cheat-sheet
-----------
## Terminal commands:

- `docker build .`
  - builds the Dockerfile in current dir and prints an id for it
  - optional parameter: `-t <name[:tag]>` gives the image a name and optionally a tag (version); the image name:tag can be used instead of id in the commands below
  - optional parameter: `--build-arg <arg_name>=<value>` specifies an argument 
- `docker tag <image_name> <new_image_name>`
  - creates a clone of an image with a different name/tag
- `docker run <image_id/name_of_dockerhub_image>`
  - creates a container based on an image, starts the container and exposes a port from container to host "-p" stands for publish
  - optional parameter: `--name <container_name>` give a name to the new container
  - optional parameter: `-p host_port:container_port` for exposing a port
    - example: `docker run -p 3000:80 a6aac062f7ed` - exports port 80 from container to port 3000 on host
  - optional parameter: `-it` start with an interactive shell (otherwise no STDIN input will work)
    - example: `docker run -it node:14` - runs the `node:14` image from docker hub
  - optional parameter: `-d` run in detached mode (don't listen to container output)
  - optional parameter: `--rm` deletes the container after it stops
  - optional parameter: `-v "<volume_name/absolute_path>:<container_path>"`
    - if first argument is `volume_name` (relative), it creates a named volume: the `container_path` is stored on the host system in a folder controlled by docker. Usually needed for persistent storage that isn't also changed on host (e.g. database files)
    - if it's an `absolute_path` (you can use `$(pwd)` for path to current dir), it creates a bind mount, a mapping from host filesystem to container filesystem. Usually needed for source code (while developing) or other files that change frequently on host and need to update automatically inside containers.
      - WARNING! It overrides all files in `container_path` (if in `Dockerfile` you installed something there, like `npm install` -> `node_modules`, it won't be there). Use anonymous volumes for that case!!! Add a `-v /app/node_modules` so that it's stored separately. If multiple volumes match a path (clash), the one with longer path wins.
      - on MacOS, you might need to add the folder in Docker GUI -> Resources -> File Sharing
  - Common usage: `docker run -d --rm -p 3000:80 --name container_name -v persistent_volume:/app/path_to_persistent_folder -v "$(pwd)/source_code_path:/app" -v /app/image_path_we_dont_want_to_override image_name`
  - optional parameter: `--env <var_name>=<value>` -- there can be multiple of them
  - optional parameter: `--env-file ./<filename>` (usually filename is .env)
    - CAREFUL! Do not commit it to git if it has security sensitive information
  - optional parameter: `--network <network_name>` uses this network for container-to-container communication (you need to first create the network)
- `docker network create <network_name>` creates a network for container-to-container communication
- `docker start <container_name>`
  - starts a container (you can find stopped container names with `docker ps -a`)
  - does not block the terminal
  - optional parameter: `-a` starts in attached mode
  - optional parameter: `-i` allows STDIN input
- `docker logs <container_name>`
  - see the output of a container
  - optional parameter: `-f` keep showing the output
- `docker container attach <container_name>`
  - starts displaying output from that container
- `docker ps`
  - shows running containers
  - optional parameter: `-a` shows also exited containers
- `docker images`
  - shows all images we have
- `docker stop <image_name>`
  - stops a running container
- `docker container prune`
  - removes all stopped containers
- `docker image prune`
  - removes all unused & untagged images
  - optional parameter: `-a` also removes tagged images (and officially installed)
- `docker rm <container_name_1> <container_name_2>`
  - deletes stopped containers
- `docker rmi <image_id_1> <image_id_2>`
  - deletes images (only if they have no container running or stopped) 
- `docker image inspect <image_id>`
  - shows information (OS, run command, layers, etc.) of an image
- `docker cp <source> <target>`
  - copies files between host and container, if either source or target is a container, use `container_name:/path_in_container`
  - to copy a folder's contents, use `folder_name/.` as source
- `docker volume ls`
  - lists all volumes
- `docker volume prune`
  - removes all volumes that aren't mapped to any container

hint: instead of `<image_id>` I can put only the first chars of the id

hint2: each docker command 

## Dockerfile
- `FROM <image_name>[:<image_version>]`
  - start from a docker hub image
- `WORKDIR <docker_dir>`
  - the working directory
- `COPY <host_dir> <docker_dir>`
  - example: "COPY . /app" copies current dir into the image "app" folder
  - obs: we could also use "COPY . ." since the working directory is "app"
  - obs: the COPY command ignores all files specified in `.dockerignore` (e.g. for `.git` folder or `Dockerfile` or `node_modules` if you have it installed locally)
- `RUN <command>`
  - example: "RUN npm install"
  - obs: do NOT start the app using RUN, that's not part of the image, but the container's job
- `ENV <var_name> <default_value>`
  - sets an env variable;
  - it can also be used in the other commands using $ (e.g. `$PORT`)
  - NOT RECOMMENDED for security sensitive data
- `ARG <arg_name>[=<default_value>]`
  - declares a build time argument. Can be used in other commands (except CMD) in Dockerfile by putting a $ before the arg name.
  - obs: if you put it as late as possible, docker build will be faster in case of arg change
- `EXPOSE <port_number>`
  - example: "expose 80" - allow communication on that port with host machine
  - obs: it is optional, but best practice.
- `VOLUME [ "<path_in_container>" ]`
  - DON'T USE THIS (see `docker run -v` parameter)! Annonymous volumes have randomly assigned names. If you delete the container and create a new one, it won't map to the same volume.
- `CMD ["command", "param1", "param2"]`
  - example: CMD ["npm", "run", "server.js"]
  - obs: should be the last command
  - obs: tells what to run when a container is started
  - obs: if no CMD is specified, the CMD of the base image is used

## Sharing an image
- options: public or private registry (docker hub limits to 1 private repository)
- create a dockerhub account
- create a repository (= image) --> after creating the image, docker hub shows the command to push a new tag to the repo
- build/tag an image with the same name as the repo name in dockerhub (e.g. `docker tag <image_name> cristianmilitaru/test_repo:1 .` -- creates a clone)
- `docker login`
- `docker push cristianmilitaru/test_repo:1` --> pushes only the extra information, not also the base image contents 

Notes
-----
- containers = the thing you run - contains code + required tools/runtimes + is actually running the app
- image = template for containers - contains code + required tools/runtimes
  - obs: the containers don't copy stuff from image, so if we have 3 containers running on the same image, the environment + code is only stored in the image
- I can use the same image to create multiple similar containers.
- Docker Hub has many pre-built docker images. I can create my own image based on an pre-built image
- Each "docker run" command creates a new container based on an image
- Images are read-only! Changes in source code require a new "docker build ." to create a new image
- Images are layer-based! After each instruction, docker creates a cached version, so if nothing changed that far, "docker build ." will skip the step (see "Using cached" in "docker build ." output). After the first instruction that has changed inputs, all subsequent instructions are ran.
  - Optimization: do steps as late as possible, e.g. in a js app, first copy package.json, then npm install, then copy source code. Otherwise, if we copy everything and then npm install, each time the source code changes it would npm install again.
- Data storage:
  - Containers can store temporary data (files will be deleted if container is deleted)
  - Volumes can persist data of containers outside of containers, in the host filesystem. Like a mapped area. But the host cannot access that data (docker stores them somewhere). Useful for database files / persistent storage.
  - Bind Mounts are mappings between container and host machine. We can configure where the container path is on the host machine. Useful for source code during development, because otherwise, if we copy it in the image, it would take a lot of time after each code change.
    - Careful! See warning at cheatsheet for `docker run` --> `-v` parameter
    - For `nodejs`, to not need to restart the server on file changes, I can use `nodemon` to restart server when any file changes -- does not work on windows with WSL2 unless the source code is stored in WSL filesystem 
- Arguments can be accessed in the dockerfile only. They are like build-time variables. But you can also expose it to the application code by declaring an env variable with that value
- Environment variables can be accessed in both Dockerfile or application code. They are more like run-time variables.
- Networking
  - By default, containers can send requests and receive data from the internet
  - By default, containers cannot send requests to localhost ports. But if you replace `localhost` in app code URLs with `host.docker.internal` then it works, because it's replaced with the host IP address as seen from the container.
  - By default, containers can talk to other containers if you know their IP address (by using `docker container inspect` command). To make things easier, I can use the `--network <network_name>` parameter of `docker run` and specify the same network name for communicating nodes, then in the app code use container name as host name, for example `http://my_container:3000` talks to container named my_container

