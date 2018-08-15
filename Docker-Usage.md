# Docker Usage

* \* is for advanced usage

## 1 Images

* List images

  * `-a` Show all images (default hides intermediate images)

    ```
    docker images [-a]
    ```

    ```
    docker images ls [-a]
    ```

* Build image (cwdx has `Dockerfile`)

    ```
    docker build -t image name
    ```

    ```
    docker build -t [username]/[repository]:[tag] [target directory]
    ```

* Remove image

    ```
    docker rmi [image name]
    ```

* Remove all images

  * `-q` Only show numeric image IDs 

    ```
    docker rm $(docker images -q)
    ```

* Remove all `<none>` images

    ```
    docker rmi $(docker images -f "dangling=true" -q --no-trunc)
    ```

## 2 Run on container, Dockerfile

* List all containers

  * `-a` Show all containers (default shows just running) 

    ```
    docker ps [-a]
    ```

* Run a container (load an image into container)

  * `-d` Run container in background and print container ID

  * `--name` Assign a name to the container 

  * `-p` Publish a container's port(s) to the host 

  * `-v` Mount a volume

  * `--rm` Automatically remove the container when it exits

    ```
    docker run [-d|--rm] --name [custom container name] -p [local port]:[docker port] -v [local dir]:[docker dir] [image name] [cmd]
    ```

    ```
    docker run [username]/[repository]:[tag]
    ```

* Stop a container

    ```
    docker stop [container ID|container name]
    ```

    ```
    docker kill [container ID|container name]
    ```

* Remove a container

    ```
    docker rm [container ID|container name]
    ```

* Remove all containers

  * `-q` Only display numeric container IDs 

    ```
    docker rm $(docker ps -a -q)
    ```

* Run a command in a running container

  * `-i` Keep STDIN open even if not attached 

  * `-t` Allocate a pseudo-TTY 

    ```
    docker exec -it [container ID|container name] [cmd]
    ```

* Attach local standard input, output, and error streams to a running container 

    ```
    docker attach [container ID|container name]
    ```

* Detach from a container and keep running in background

  * `Ctrl-p`+`Ctrl-q`

* Create a new image from a container's change

  * `-m` commit message

    ```
    docker commit [container ID|container name] -m [commit message] [image name]:[tag]
    ```

* Remove all exited containers

    ```
    docker rm $(docker ps -q -f status=exited)
    ```

## 3 Publish

* Login
  
    ```
    docker login
    ```
    
* Add a tag to an image

    ```
    docker tag [image name] [username]/[repository]:[tag]
    ```

    ```
    docker tag [source image]:[tag] [target image]:[tag]
    ```
    
* Push an image to a repository or a registry

    ```
    docker push [username]/[repository]:[tag]
    ```
    
* Pull an existed image
  
    ```bash
    docker pull [username]/[repository]:[tag]
    ```

## 4 *Replicas, CPU, memory (scaling), select image

* 
  ```
  docker swarm init
  ```
  
* 
  ```
  docker stack deploy -c [compose file] [app name]
  ```
  
* 
  ```
  docker stack ls
  ```
  
* 
  ```
  docker stack services [app name]
  ```
  
* 
  ```
  docker stack ps [app name]
  ```
  
* 
  ```
  docker stack rm [app name]
  ```

* Worker

    ```
    docker swarm leave
    ```

* Master (manager)

    ```
    docker swarm leave --force
    ```

## 5 *Swarms, more machines for capacity

* 
    ```
    docker-machine ls
    ```
    
* 
    ```
    docker-machine create --driver [virtrualbox|vmwarefusion] [vm name]
    ```
    
* 
    ```
    docker-machine start [vm name]
    ```
    
* 
    ```
    docker-machine stop [vm name]
    ```

* View vm information
  
    ```
    docker-machine env [vm name]
    ```
* 
    ```
    docker-machine ssh [vm name] [cmd]
    ```

* Worker join
  
    ```
    docker swarm join --token [token][ip]:[port]
    ```
* 
    ```
    docker-machine scp [local file][vm name]:[path]
    ```
* 
    ```
    docker node ls
    ```

## 6 Useful tips

* Fix some docker terminal display problems
  
    ```
    docker exec -it [container] bash -c "export TERM=xterm; exec bash"
    ```

* If GPU is required, use `nvidia-docker`

    ```
    nvidia-docker exec -it [container] bash
    ```