# Useful Docker commands

[Official Docker installation instructions for Ubuntu 20.04 LTS](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04)

[Official Docker Compose installation instructions for Ubuntu 20.04 LTS](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04)

##### Run Docker image
`$ docker run <IMAGENAME>`

##### View Running Docker processes
`$ docker ps`

##### View Locally Available Docker images
`$ docker images`

##### Run In Background
`$ docker run -d <IMAGENAME>` or `$ docker run --detatched=true <IMAGENAME>`

##### Map open ports to host
 > if you want to, for example, be able to access a web server running within the Docker container from an outside browser, you can do this to make it available

`$ docker run -P <IMAGENAME>`
