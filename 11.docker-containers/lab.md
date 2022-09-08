
# Lab

Containers with Docker

## Objectives 

1. Install Docker
2. Write a `Dockerfile` and build a Docker image
3. Run a Docker container with multiple options
4. Share your Docker container with a classmate
5. Apply your knowledge to create your own docker image.

## Useful links

- [Dockerfile best practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

## 1. Install Docker

Before you can start the lab, you have to:
1. Install [Docker Desktop](https://www.docker.com/get-started) following the instructions depending on your OS.
2. Make sure your docker installation is working properly by running the following command in a terminal. You can run it from whichever location. Read the displayed text and try to understand what happened.
   ```
   docker run hello-world
   ```

## 2. Write a Dockerfile and build a Docker image

1. Open [`assets/hello-python`](assets/hello-python) directory and check out the `hello_world.py` and `Dockerfile` files
2. Check out the explanations for each line in the Dockerfile from [the documentation](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#dockerfile-instructions) 
3. Build the docker container   
  1. Open a terminal (CMD or PowerShell for Windows)
  2. Navigate to the [`assets/hello-python`](assets/hello-python) directory in the cloned repository
  3. Run the following command:
     ```
     docker build -t hello-python .
     ```
     - Don't forget the `.` at the end of the command. It is here to tell Docker it should look for the `Dockerfile` in the current directory. 
     - `-t` tag - to build container with the name you want (here `hello-python`)
4. Check if your Docker container appears in the local Docker images:
   ```
   docker images
   ```
 5. Run the container with: 
    ```
    docker run hello-python
    ```
 6. Check all the running containers: `docker ps`. Do you see yours? Why?
 7. Run `docker ps -a` to list all existing containers. Compare restarting one with `docker start <name-of-container>` or `docker start -a <name-of-container>`.    

## 3. Run a Docker container with multiple options

1. Inspect and run the `example1-self-contained`, where you will learn how to map the port on the hos and container.
2. Inspect and run the `example2-data-science-project`, where you will learn how to create a mount point. Read about [Docker Volumes](https://docs.docker.com/storage/volumes/).   

## 4. Share your Docker container with a classmate

1. Modify the message printed in the `hello-python` (you can add your name for example).
2. Rebuild the Docker container (with a different name) with this modified code and see if you can run it.
3. Register on [Docker Hub](https://hub.docker.com/).
4. Tag your container with the following command:
   ```
   docker tag hello-python-dsti <DOCKER_ACCOUNT_NAME>/<CUSTOM_IMAGE_NAME>
   ```
   where `DOCKER_ACCOUNT_NAME` - is your account on Docker Hub, `CUSTOM_IMAGE_NAME` - the custom name of the image.
5. Log in to Docker Hub from your terminal:
   ```
   docker login
   ```
6. Push the docker image to Docker Hub:
   ```
   docker push <DOCKER_ACCOUNT_NAME>/<CUSTOM_IMAGE_NAME>
   ```
7. See if you can find the image in your [repositories](https://hub.docker.com/repositories) in the Docker Hub
8. Ask a classmate to retrieve your Docker container and run it:
   ```
   docker pull <DOCKER_ACCOUNT_NAME>/<CUSTOM_IMAGE_NAME>
   docker run <DOCKER_ACCOUNT_NAME>/<CUSTOM_IMAGE_NAME>
   ```

## 5. Exercise

1. Try to build and run a docker container executing the pytests from the unit testing lab (module 3).
2. Explore [Docker Tensorflow](https://www.tensorflow.org/install/docker).
