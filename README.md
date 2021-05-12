# Assumptions
You:
* Have an internet connection
* Know how to use the command line interface
* Have either:
    * an e-mail address to use when signing up for a Docker account or,
    * a Docker account
* Have done application development and are familiar with editing text files
* Are comfortable with Git

# Understand
## What is Docker?
Docker provides a way to portably run software in packages called containers. Containers are isolated from one another and bundle their own software, libraries and configuration files.

Features of Docker:
* Identical configuration of images eliminates unknown environmental differences
    * After initial setup of getting 'any container' to run it eliminates "it works on my machine" for all containers.
* Can run on Linux or Windows.
* Can run Linux or Windows containers.
    * Linux is by far the more popular kernel to use due to cross-compatability, and due to Windows container size.
        |              | Windows container | Linux container |
        |--------------|-------------------|-----------------|
        | Windows host | ✓                 | ✓              |
        | Linux host   | X                 | ✓              |

## What Docker is not
Docker is not:
* Virtualization/VMs
    * caveat: "Docker Toolbox" is a VM that has Docker installed on it, but Docker by itself is not "virtualization".
* Microservices
    * Microservices could be run on anything, but Docker is the most popular ways to run microservices. It's safe to assume Docker as a seperate but prerequisite technology for microservices.
* For desktop applications
    * Docker does not provide a GUI interface and is not suitable for desktop applications.
        * caveat: X11 can be used for edge cases but has not seen mainstream use.

## Introdoctury terms
* Image
    * The template from which containers are created. An image is built from the Dockerfile. An image does not have state and it never changes. Synonomous with a "golden image" VM template or a software class.
* Tag
    * Specific version of an image. Images do not change, but new images can be created with new tags. Synonomous with "version"
* Container
    * A running instance of an image. Containers store changes to filesystem temporarily.
* Registry
    * Storage for images.
* Dockerfile
    * This file contains instructions to create a Docker image when built/"compiled".

## How to use Docker
Docker can be used in two scenerios, software development and production use.

For production use Docker is often paired with an "orchestrator" to manage the running and co-ordination of Docker containers. Production usage and orchestration is outside the scope of this documentation.

For software development the workflow usually looks like the below example:
### Initial setup:
0. Configure Dockerfile

### Application development:
0. Make code changes
0. Test the changes via unit tests
0. Test the changes in Docker by:
    0. Building the Docker image
    0. Run the container
    0. Test the code changes
    * Although outside the scope of this documentation, Docker can automatically update a local running container with changes (file saves or compile events) to allow rapid testing which would eliminate the need to build and run the container after every code change through "hot-loading".
0. Repeat the above steps until all desired changes are implemented
0. Submit a pull request.

# Implement
## Getting Docker
Docker can be installed locally on your computer. This process can involve more than installing software. For the sake of brevity we will use (labs.play-with-docker.com)[https://labs.play-with-docker.com/] which behaves similarly to having Docker installed on your computer.

### Play with Docker
The following steps with get you access to an online CLI that can run Docker

0. Visit (labs.play-with-docker.com)[https://labs.play-with-docker.com/]
0. Click on the "Login" button
0. Sign in.
    * If needed go through the "sign up" process. Note that this login may be used to upload any images you create to Docker Hub, the main Docker repository.
0. Click "Start"
0. Click "+ Add new instance"

#### Docker run
Run `docker run alpine`. This will check the local repository and download the image if it isn't found. Since this is a new environment it will not find the alpine image and will automatically pull the alpine image from Docker Hub. After pulling the image it will then spawn an instance of the image as a container.

Now that the image is downloaded try running `docker run alpine` again. Notice that there is no console output. This is because the first time the command was run the image pull process was creating console output. This time Docker ran the alpine container to completion but the alpine image doesn't create any output on it's own. This is not an uncommon behaviour as most docker containers are focused on providing a service not providing console feedback.

##### Console output
Now try running `docker run hello-world` twice. This container is designed to output to the console so everytime it runs there should be some visible output.

Up to this point all containers that have been run weren't setup to provide a service so they ran to completion then terminated. What about running a container that provides a service? Run the command `docker run httpd` and wait for the image to be automatically pulled. And wait. And wait. The console hasn't been returned to us. You can hit enter as many times as you want or try entering commands but it won't do anything useful. This is because the Docker run command is faithfully outputting the console logs from httpd and httpd hasn't terminated. This isn't very useful if you wanted to start the container then use the CLI to do something else or run multiple containers. To regain control press `ctrl+c`. However there is a solitution to this; Daemonization.

##### Daemonization
To run a container and return control to the console pass the '-d' (daemonize) switch. Run `docker run -d httpd`. Notice in the string of characters that are logged to the console? That is the container's ID. Container IDs will be covered in greater detail later.

##### Interactive mode
The above examples have been non-interactive once executed. What if you wanted to run CLI commands on the container? To do this you can either run the container interactively, attach to the container (docker attach), or run a single command to the container from your host (docker exec). The simplest way is to run the container interactively with the -i (interactive) and -t (terminal) switches. We are also going to name this container with the `--name` parameter. Run `docker run -it --name MyContainer alpine`. From here you can run commands inside the container. Try running a few commands on the container. When done, run `echo "I did it!" > firstname_lastname.txt` to write to a file on the container then run `ls` to confirm the file was written.

To exit the interactive mode run `exit`

### Docker ps
Running the command `docker ps` will show all running containers. There should be a single container shown here, the httpd container. Note that no other containers are shown. If a container has been started then stopped it will not be visible unless you use the '-a' (all) switch `docker ps -a`. Running that command will show you all the containers that have been created so far.

### Docker start
To start a container that is stopped you can run `docker start $ID_or_name`. To attach an interactive console to the container when you start it use the '-a' (attach) and -i (interactive) switch. Run `docker start -ai MyContainer` to launch the container you created the file in earlier. Lets check the file that was created with `ls`. The file is still there. However if you create another container on the host then it won't have the file. This is because it created a new container from the base image and is utterly unaware of "MyContainer".

### Port forwarding
You can expose container ports with `-p $hostPort:$containerPort` syntax. Do this now by running `docker start -d -p 80:80 --name MyWeb httpd`

### Docker logs
To find the console output of containers you can use `docker log`. This can be used to recall old information, or poll the console output of a running container like httpd which may log to the console. Run `curl localhost` to establish an HTTP connection to the webserver on the MyWeb container. After this run `docker logs MyWeb` and you will see that the container logged the incoming connection. This can be useful for getting information out of a container if you get an error during testing.

### Environmental configs
Everything covered so far has been working with non-customized containers. What if you needed to run a container that connected to another service, but the dev service was different from the production service? You wouldn't want to test dev work on production services so you shouldn't hard-code production locations into the image. Docker allows us to pass environmental variables to the container through the `-e '$Key=$Param'` syntax. There is more to using environmental parameters but that is outside the scope of this document. 

### Dockerfile
#### Dockerfile syntax
Up to this point we've only been using pre-defined images. If we want to create our own image (step 0 in the "initial setup" of the software development workflow) then we must build the image. The Dockerfile is what defines a Docker image. Below is an example Dockerfile:
```
# Alpine Linux with OpenJDK JRE
FROM openjdk:8-jre-alpine

# copy WAR into image
COPY spring-boot-app-0.0.1-SNAPSHOT.war /app.war

# run application with this command line 
CMD ["/usr/bin/java", "-jar", "-Dspring.profiles.active=default", "/app.war"]
```

Lets cover the syntax used here.

##### FROM
This specified the base image. In this case we are using a prebuilt container, and wrapping it with more functionality.

##### COPY
This command will copy a file or directory from the directory containing the Dockerfile to the image. 

In this case it copies the "spring-boot-app-0.0.1-SNAPSHOT.war" file next to the Dockerfile and places it in the root directory (/) of the image as "app.war".

##### CMD
This is an array of elements that specify what command to automatically run, and what parameters to pass to it. In this case it runs Java and passes it the /app.war file thus starting a SpringBoot application.

#### Building the image
`docker build -t myimage .` will build the Dockerfile in the local directory (.) and name the image "myimage".

### Docker image
`docker image` is used to manage images. This includes building, removing, and uploading images.

### Docker rm
`docker rm $container` will delete a stopped container. This is how you clean up the output of `docker ps -a`.

#### Docker kill
`docker kill $ID_or_name` stops the specified running container.

#### Docker pull
`docker pull $image` will manually pull an image. This is done automatically by `docker run` if needed.

# Stretch goals
* Create a Docker image of your own and run it.
* How do you run a specific version of an image?
* How do you persist data between containers?
* How do you remove an interactive container on exit, without having to manually remove it?
* Install Docker on your computer.

# Resources
* http://training.play-with-docker.com/
* https://www.docker.com/get-started