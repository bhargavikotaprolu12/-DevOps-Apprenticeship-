For EVERY command answer
1 Why did I run this?
2 What happened internally?
3 What files changed?
4 Did Docker create an Image or Container?

docker version

docker info

## docker images
You run docker images to see if the images stored locally on your machine. 
Docker queried the local image store and listed the downloaded or built images available for creating containers.
It displays only information and does not modify anything.
It does not create anything only displayed information.

## docker ps
It queries the docker daemon to display a snapshot of all current running containers on your system.
You can use this command to check process status such as active health, uptime and identity of your background applications environments. Port verification: you want to see which host ports are mapped to which container ports. ID retrieval.
What happened internally? Docker client formats your query into a REST API request. The dockerd recieves the request. 
No files changed. It is entirely read-only operations query, it writes zero data to your disk.

## docker ps -a
It queries the docker daemon to display a snapshot of all containers on your system regardless of whether they are running, paused or stopped. -a flag stands for all.
Why did you run it? To see every container that dockerd knows about
What happened internally? Your Docker client sent a request to the Docker daemon (dockerd), which returned the metadata for every container it manages 

## docker pull hello-world
docker pull hello-world downloads the official hello-world image from Docker Hub to your local machine. It does not run a container by itself; it only fetches the image so you can use it later.
What happened internally? 
Docker checks if hello-world image already present locally and downloads from docker hub. The daemon stored that image so it can be used later with docker run
What files changed?
Usually, no project files changed on your machine. Docker may have updated its local image storage and metadata under Docker’s own data directory, but your app/source files were not modified by docker pull.
Did Docker create an Image or Container?
Docker is not creating a brand new image it is downloading already published hello-world image

## docker run hello-world
 Why did I run this?
I run this command to check if the docker is installed and working fine. 
What happened internally?
Docker first checked your local machine for the hello-world image if not it downloads from docker hub creates a container run a tiny program inside it and  shows output back to the terminal.
What files changed?
Docker may have updated its local image store and created container metadata and writable-layer data under Docker’s own storage area, but your application files were not modified by this command.
Did Docker create an Image or Container?
Docker created a container from the hello-world image. 

## docker run ubuntu
Why I used this command?
I run this command to start a ubuntu-based container it creates the container from the ubuntu image and then tries to execute image's default command, so if the command exits the container will stop
What happened internally?
Docker checked if the ubuntu image is present locally. If not it pulled the image and creates a writable container layer, setup the container filesystem, network namespaces and process isolation and launched the default process inside it through the container runtime.
What files changed?
Docker downloads the image layers into local storage and created container metadata.
Did Docker create an Image or Container?
Docker created the container not a image

## docker run -it ubuntu bash
Why I used this command?
It will starts an ubuntu container and gives you an interactive bash shell inside it. The -t keeps standard input open -t gives you terminal and bash is the command that runs inside the container.
What happened internally?
Docker checked if the ubuntu is existed locally, pulled it and then created a new container from that image. It sets up the container filesystem, network and started bash as the main process so you could type commands interactively.
What files changed?
Docker downloaded image layers into local storage and created a writable container layers plus metadata for that container.
Did Docker create an Image or Container?
Docker created the container not a image. 

## docker run nginx
Why I used this command?
I  run docker run nginx command when I want a consistent nginx setup inside the container. 
What happened internally?
Since it is a web server docker run nginx does not open the browser page by itself it starts the NGINX process inside the container and nginx keeps running the container as the main process so the container stays active.
What files changed?
Docker stored image layers and container metadata in its own local storage
Did docker create image or container?
Docker created a container not a new image. 

## docker run httpd
Why I used this command?
Used docker run httpd command to start a Apache httpd server container from the official httpd image. I used to it to quickly run a web server and test Apache.
What happened internally?
Docker checked whether the httpd image was already on your machine. If not it pulled the image from docker hub created new container from it, set up the container's isolated environment and started the Apache process inside it. 
What files changed?
No files changed, docker download image layers into its local storage and create container metadata.
Did docker create image or container?
Docker created a container not a image. 

## docker run alpine
Why I used this command?
I used this command mainly to start a very small linux container
What happened internally?
Docker checks your local image cache for alpine. If missing it downloads the alpine from dockerhub creates a new container from it set it up the container's isolated filesystem and then started the default command inside it.
What files changed?

Did docker create image or container?
Downloaded image file and container metadata into its own local storage and container may have 

## docker inspect hello-world
Why I used this command?
Used to look at the details of the image or container metadata such as layers, config, entrypoint/CMD, environment, architecture and other low-level details.
What happened internally?
Docker looked up the object named hello-world 
What files changed?
No files changes, this command only reads docker metadata.
Did docker create image or container?
Does not create image or a container. It only shows information about an existing docker object.

## docker history nginx
Why I used this command?
Used this command to see how nginx image was built, layer by layer. Can see what commands, files and metadata were added to create that image.
What happened internally?
Docker read the local metadata from the nginx and displayed its history entries.
What files changed?
This command is read-only and only inspects the image history.
Did docker create image or container?
Does not create either. Only shows build history of an existing image
