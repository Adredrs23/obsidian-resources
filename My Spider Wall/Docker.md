Docker is used to containerize our apps for various reasons like managing environments, managing dependencies, independency for operating system biases, integration with CI/CD systems.

- Application has a dockerfile that generates the image
- an image is the blueprint that informs how the container will behave
- images will have the necessary commands to basically set up the containers dependency and environments
- a docker image will have a base image to begin with
- alpine version is the most lightweight and minimal base version to work with
- one pulls the base version from the dockerhub which is the registry to host and manage these images.
- images will always have a tag indicating their versions eg alpine
- images are first needed to be built
- dockerfile will contain the instructions to build the image
- while building the images all the individual instructions will be cached and optimized for further builds
- any change to an instruction in dockerfile will invalidate the commands cache below it
- it is better to always have a workdir for your app as to not have collisions with other already existing namespaces/folders/files
- commands like npm install will temporary spin up a container to add the dependencies to the image and disappear when done
- an image will always consist of command that indicates how the app should run for example npm run start in cmd directive. This cmd directive will be called when a container starts.
- dont forget to order commands in dockerfile to avoid redundant processes like npm install as one instruction cache invalidation can take a long maneuver of doint that process again. So always try to optimize your dockerfiles. For example, copying package.json first and installing it so that any code changes done shouldn't invalidate the npm install script as well as shown below.
```dockerfile

// unoptimized
FROM node:alpine
WORKDIR /usr/app
COPY . .
RUN npm install
CMD ["npm", "run", "start"]

// optimized
FROM node:alpine
WORKDIR /usr/app
COPY package.json .
RUN npm install
COPY . .
CMD ["npm", "run", "start"]
```
- build images using 
```zsh
docker build -t tagname
```
- view all images with docker images
- run them using 
```zsh
docker run -d --name container-name image-name
```
- -d indicates detach mode
- to list all containers
```zsh 
docker ps
```
- every running container generates a unique id
- use stop or kill to terminate containers
```zsh 
docker stop id
docker kill id
```
- docker stop is safe and happy path will all the cleanup, whereas docker kill is an abrupt termination
- an important step is to port forward your container to external machine otherwise app despite running in the container wont be accessible outside
- port forwarding - machine-port : container-port
- so the above command will look like: 
```zsh
docker run -d --name container-name -p machine-port : container-port  image-name
```
Testing this 