# This allows you build your image up on another base image
# If this is exist in local it will run. If it's not Docker will download it from Docker Hub
FROM node:12

WORKDIR /app

# Tell Docker which file that live here on our local machine should go into the image.

# Host file system
# First '.' is outside of the image where the files leave that should be copied into the image. It means that is the same folder that contains the Dockerfile (All folders and subfolders)

# Image/cointainer file system
# Second '.' is where those files hould be stored.
# Actually here it is a good idea to not use the root folder in your Docker container but some sub folder whici is totally up to you.
# COPY . ./
COPY . /app

# We need to run NPM install (or another environment). Because we had to do outside of the container as well. For node applications we had to run `npm install` in order to install all the dependencies of our node application
# By default all those commands will be executed in the working directory of your Docker container and image. and by default, that working directory is the root folder in that container file system. Since I'm copying my code into the /app folder I actually want to run `npm install` inside of the app folder as well. And a convenient way of telling Docker that all commands should be executed in that folder is taht you set another instruction before you copy everything. Thats a `WORKDIR` instruction for a setting the working directory of the Docker container and setting that to /app.
# After you set WORKDIR you can set your COPY to `COPY . ./`
RUN npm install

# Docker container is isolated from our local environment. And as a result, it also has its own internal network. And when we listen to port 80 when we node application inside of our container, the container does not expose that port to our local machine. so we won't be able to listen on the port just because somethings listening inside of a container
EXPOSE 3000

# Now we want to start our server.
# The difference between `RUN` and `CMD` is RUN will execute when image is created but CMD will execute when a container is started based on the image.
# CMD should always be the last instruction in our Dockerfile
CMD ["node", "server.js"]