# Section 1: Getting Started
# Docker
## What is Docker? And Why?
Docker is a **container** technology: A tool for creating and managing containers.
___
## Container
* A standardized unit of software.
* A Package of code **and** dependencies to run that code (e.g. NodeJs code + NodeJs runtime)
 
* The same container always yields the **exact same application and execution behavior!** No matter where or by whom it might be executed. 
* Support for Containers **is built into** modern operating systems!
* **Docker simplifies** the creation and management of such containers.
___

## Why Containers
Why would we want <span style="color:orange"> independent, standardized "application packages"</span>?

* Different Devewlopment & Production Environments
  * We want to build and test in exactly (!) the same envionment as we later run our app in.
* Different Development Environments Within a Team / Company
  * Every team member should have the exactly (!) same environment when working on the same project.
* Clashing Tools / Versions Between Different Projects
  * When switching between projects, tools used in project A should not clash with tools used in project B.
___
## Solution: Virtual Machines / Virtual Operating Systems
Pros:
* Separated environments
* Environment -specific configurations are possible.
* Environment configurations can be shared and reproduced reliably

Cons:
* Redundant dublication, waste of space.
* Performance can be slow, boot times can be long
* Reproducing on another computer server is possible but may still stricky.
___
## Containers vs Virtual Machines
Docker Containers
* Low impact on OS, very fast, minimal disk space usage.
* Sharing, re-building and distribution is easy.
* Encapsulate apps/environments instead of **whole machines**.

Virtual Machines
* Bigger impact on OS, slower, higher disk space usage.
* Sharing, re-building and distribution can be challenging.
* Encapsulate **whole machines** instead of just apps/environments.
___
## Docker Tools & Building Blocks
* Docker Engine
* Docker Desktop (incl. Deamon & CLI)
* Docker Hub
* Docker Compose
* Kubernetes
___
 ## Create First Container
 ```Dockerfile
FROM node:14

WORKDIR 001

COPY package.json .

RUN npm install

COPY . .

EXPOSE 3000

CMD ["node", "app.mjs"]
```
___

This builds the image the Dockerfile it finds in the directory in which you run this. 
```bash
$ docker build .
```

Then we should run the docker
```bash
$ docker run -p 3000:3000 31c1acdc6e7a
```

If you want to stop docker run: `docker ps` and select your image. After run `docker stop <name>`

`docker ps ` shows running without `-a` you see the running proccesses.
___
# Course Outline
Getting Started & Overview

* Foundation
  * Images & Containers
    * How you can build your own images
    * How you can use existing images
    * How you can run and configure containers based on images
  * Data & Volumes (in containers)
    * How you can manage data in containers
    * How you can ensure that data persists so that is not lost if a container is shut down and restarted and removed in between for example.
  * Containers & Networking
    * How multiple Containers can talk to each other. For example you might be building an application where you have Node REST API in one Container and have a react JS front-end in another Container
___
* "Real Life"
  * Multi-Container Projects
    * What should we careful about and how to manage these projects
  * Using Docker-Compase
    * Managing containerized applications much more easier
  * "Utility Containers"
  * Deployin Docker Containers
___
* Kubernetes
  * Kubernetes Introduction & Basics
  * Kubernetes: Data & Volumes
  * Kubernetes: Networking
  * Deploying a Kubernetes Cluster
___

# Section 2: Docker Images & Containers: The Core Building Blocks
* Two Core Concepts: Images & Containers
  * How images are reletad to Container and how you need both to work with Docker
* Using Pre-Built & Custom Images
* Creating & Managing Containers
___
## Images & Continers
* Images
  * Templates/Blueprints for containers
  * Copntains code + required tools/runtimes
* Containers
  * The running **unit of software**
  * Multiple containers can be created based on one image
___
## Finding / Creating Images
We need an Image
* Use an **existing, pre-built Image**
  * Best source is [Docker Hub](https://hub.docker.com)
  * `docker run node` : this command will use find node on Docker Hub and utilize it to create so called container based on this image
  * For interactive session : `docker run -it node`
  * For list your containers : `docker ps -a`
* Create your **own, custom Image**
  * Write your own Dockerfile (based on another Image)
___
## Dockerfile
This allows you build your image up on another base image

If this is exist in local it will run. If it's not Docker will download it from Docker Hub

```Dockerfile
FROM node
```
___
Tell Docker which file that live here on our local machine should go into the image.

This is **host file system**

First '.' is outside of the image where the files leave that should be copied into the image. It means that is the same folder that contains the Dockerfile (All folders and subfolders)

And the other part is **Image/cointainer file system**

Second '.' is where those files hould be stored.

Actually here it is a good idea to not use the root folder in your Docker container but some sub folder whici is totally up to you.

```Dockerfile
# COPY . ./
COPY . /app
```
___
We need to run NPM install (or another environment). Because we had to do outside of the container as well. For node applications we had to run `npm install` in order to install all the dependencies of our node application.

By default all those commands will be executed in the working directory of your Docker container and image. and by default, that working directory is the root folder in that container file system. Since I'm copying my code into the **/app** folder I actually want to run `npm install` inside of the app folder as well. 

A convenient way of telling Docker that all commands should be executed in that folder is taht you set another instruction before you copy everything. Thats a `WORKDIR` instruction for a setting the working directory of the Docker container and setting that to **/app**.

After you set `WORKDIR` you can set your `COPY` to `COPY . ./`
```Dockerfile
RUN npm install
```
___
This command is **optional**.

Now we want to start our server.

Docker container is isolated from our local environment. And as a result, it also has its own internal network. And when we listen to, for example port 80 when we node application inside of our container, the container does not expose that port to our local machine. so we won't be able to listen on the port just because somethings listening inside of a container.
```Dockerfile
EXPOSE 80
```
___
The difference between `RUN` and `CMD` is `RUN` will execute when image is created but `CMD` will execute when a container is started based on the image.

`CMD` should always be the last instruction in our Dockerfile.
```Dockerfile
CMD ["node", "server.js"]
```
___

## Turn Dockerfile Into An Image and Then Into A Container

Create an image based on the instructions in specified Dockerfile

```bash
$ docker build .
```

For start container with local accessable port
```bash
$ docker run -p 3000:80 <container-id>
```

Note: Command contains port 80 because we **expose** port 80 in our Dockerfile
___
## Images are Read-Only!
If you change your source code you don't see that change in application.Because images doesn't work that way.

In Dockerfile we `COPY` everything inside of project folder including source code into the **image filesystem** into the **/app** folder.

Therefore we do one important thing. We **copy** our **source code** into the **image** and we basically take a snapshot of our source code at the point of time we copied.

If I thereafter edit my source code, **this change is not part** of the source code in the image. We need to **rebuild** our image to copy our updated source code into a new image. That is very **crucial**.

But we don't have to build image everytime we change our code. There is more elegant and faster way of picking up changes in our code later.

But the core takeaway here is important and will always be true.

### Images are basically locked and finished once you built them. 

### Everything in the images is read only then, and you can't edit from the outside by simply updating your code just because you copied that code in the past.

### The images doesn't care about the past. Once `COPY` operation is done, you can change your outside code however you want. You can even delete it and the image will not be affacted.

### You need to rebuild to pick up external changes and basically copy all the updated code into the image.
___
## Image Layers
Every instruction in Dockerfile represents a layer. And an image is simply built up from multiple layers based on these different instructions.

Images are layer based and every instruction creates a layer and these layers are cached. If you then run a container based on an image that container basically adds a new extra layer on top of the image which is that running application that running code.

In Dockerfile, all the instructions before the final instruction are already part of the image though as separate layers, and if nothing changes all these layers can be used from cache.

Whenever one layer changes, I said that all layers **after the changing layer are reubilt**

If you have dependencies, everytime you rebuild that dependencies are reinstalling. You can avoid by running`COPY package.json /app` insturction before changing layers.

Example:
```Dockerfile
FROM node

WORKDIR /app

COPY package.json /app

RUN npm install

COPY . /app

EXPOSE 80

CMD ["node", "server.js"]
```

In that way, if you change your source code, you just need to build

```Dockerfile
COPY . /app

EXPOSE 80

CMD ["node", "server.js"]
```
parts. Because the older layers didn't change (unless you add new dependencies) and they are **cached**.
___
## Managing Images & Containers
Images
* Can be **tagged** (named)

  `$ docker tag`

* Can be **listed**

  `$ docker images`

* Can be **analyzed**

  `$ docker image inspect`

* Can be **removed**
  
  `$ docker rmi`

  `$ docker prune`

Containers

* Can be **named**
  
  `--name`

* Can be **configured in detail**

  see `--help`

* Can be **listed**

  `$ docker ps`

* Can be **removed**

  `$ docker rm`
___
## Stopping & Restarting Containers
You can always **restart** your stopped containers that you see in `$ docker ps -a`

`$ docker run` always creates **new** container.

If nothing changed in our dependencies or our source code and our image didnt change, there is no need to create a new brand container. We can just **restart** an existing container. For that:

Search for stopped containers with

`$ docker ps -a`

and pick the container name under `NAMES`, or you can select container id under `CONTAINER_ID`. And then you can **restart** your stopped container by following command:

`$ docker start <container-name>`

or

`$ docker start <container-id>`

Docker starts containers but in different mode. For that we need to understand attached and deattached containers.
___
## Attached & Deatached Containers
`$ docker run` is attached mode

`$ docker start` is detached mode

but which one do we want and why does this even matter?

* With attached mode we can see the terminal outputs. If you want `$ docker run` in detached mode you must follow

  `$ docker run -p <port>:<port> -d <image-id>`

* If you want attach yourself again to a running container

  `$ docker attach <name>`

* Also you can see the logs without attach a container with

  `$ docker logs <name>`
___
## Entering Interactive Mode
*folder 003*

* You can use interactive run with
  
  `$ docker run -i <name>`

  or

  `$ docker run --interactive <name>`

* Or you can use it with `--tty` or `-t` and this means it creates a terminal and if you combine `-i` and `-t` flag we are able to input something so the container will listen for our input and we'll also get a terminal exposed by the container which is actually the device where we enter the input.

  `$ docker run -it <name>`

* If you want to restart your container with interactive mode

  `$ docker start -a <name>` (this is only read mode)

  or

  `$ docker start -a -i <name>` (interactive mode)
  ___
## Deleting Images & Containers

* First you need to stop the container you want to remove if its up and running. Then 

    `$ docker rm <name> <name2> ..`

* Alternatively, you can also run
  
  `$ docker container prune`

  to remove **all stopped containers at once**

* If you want to remove images you need to list all images with

  `$ docker images`

  and remove them with

  `$ docker rmi <image-id>`

  You can't remove images belonging to any container rather is running or stopped. You need to remove it first.

  For remove **all** the images

  `$ docker image prune`

  For remove **ALL** images including **tagged images**

  `$ docker image prune -a`
___
## Removing Stopped Containers Automatically

You don't need to manually clean all stopped containers from time to time

`$ docker run --rm <name>`
___
## Inspecting Images
`$ docker image inspect <image-id>`
___
## Copying Files Into & From A Container
Add something to container or extract something from container while it is already running

* Copy files or folders into a running container or out of a running container.

  `$ docker cp <path-you-want-to-copy> <container-name>:<output-destination>`

* If you want to copy file to out of container

  `$ docker cp <container-name>:<path-you-want-to-copy> <output-destination>`

  Now our output destination is local folder.

This commands allows you to add something to a container without restarting the container and rebuilding the image. 

When your source code change normally you need to rebuild the image because of that and restart the container.

But copy something in our container is not good solution because it's also not really possible to replace files which are currently executed like the server.js file.

So copying a file into a container is not a good solution.

We will instead learn about a better way of updating code in a container without rebuilding the image later.
___
## Naming & Tagging Containers and Images
* Set a **container name**

  `docker run --name <container-name>`

* Set an **image tag**

  `$ docker build -t <name>:<tag>`

  * Image tags consists of two parts:
    
    * **name** also called **repository** and then a **tag** separated by a colon like

      `name:tag`
      * **name** : Defines a **group of, possible more specialized**, images

        Example: "node"
      * **tag** : Defines a **specialized image within a group of images**

        Example: "14"

    **Combined**: A **unique identifier**
___
## Sharing Images - Overview
* We want to have the **exatct same environment for development and production** -> This ensures thatit works exactly as tested.

* It should be easy to **share a common development environment** / setup with (new) employees and colleagues.

* We **don't want to uninstall and re-install** local dependencies and runtimes all the time.

There are two ways to share an image:

* Share a **Dockerfile**
  
  * Simply run `$ docker build .`

  * **Important**: The Dockerfile instructions **might need surrounding files** / folders (e.g. source code)

* Share a **Built Image**

  * **Download** an image, **run a container** based on it.

  * **No build step** required, **everything is included in the image** already!
___
## Pushing Images to DuckerHub
There are two main places where we can push our images to:

* Docker Hub

  `$ docker push <image-name>`
  
  * Official Docker Image Registry

  * Public, private and "official" Images

* Private Registry

  `$ docker push <host>:<image-name>`

  * Any provider / registry you want to use

  * Only your own (or team) Images)

If you want to **pull** an image:
  * For Docker Hub

    `$ docker pull <image-name>`

  * For any other registry

    `$ docker pull <host>:<image-name>`

**Important Note**

For pull or push an image, your Docker Hub repository name needs to match with your local image. If your local image name is different you can **rename** it with

`$ docker tag <old-name>:<tag> <new-name>:<tag>`

Add *tags* if there are any

When you rename an image, you don't get rid of the old image instead, you kind of create a **clone** of the **old image**.

But of course you can't push pictures just because you renamed tags. If you rae not logged in, your request going to be denied. For login and logout:

`$ docker login`

`$ docker logout`
___
## Pulling & Using Shared Images
For pulling images:
  
  `$ docker pull <image-name>`

If you want to use any public docker image you can just simply:

  `$ docker run <image-name>`

**Notes:**

  `$ docker pull` command will always pull the latest image of that name from your container registry. So if in the meantime, you will rebuild the image and push an updated image to the registry the next time you will execute `$ docker pull` will fetch that latest image.

  You could also run `$ docker run` without running `$ docker pull` first. If `$ docker run` doesn't find an image locally on your machine, it will automatically reach out to the container history your image name uses (Docker Hub), and it will check for the image there. And if it finds an image of that name there it will use that and pull that automatically.

  What `$ docker run` **will not** do though, and that's important, is automatically **check for updates**. If you have an image locally, because you pulled or ran it before, then if you run a container again based on an image, `$ docker run` will not check if the image you have locally on your system is the latest version of that image.

  So if in the meantime, you updated the image and pushed it again to Docker Hub, just using `$ docker run` there after **will not** give you that **latest updated image**. You instead manually need to run `$ docker pull` and then the image name first to ensure that you have that latest image version and you then execute `$ docker run` again.

  So it automatically pulls images but not necessarily the latest version if you already have that image locally on your system, because you pulled it before already.


[Cheat Sheet](images/Cheat-Sheet-Images-Containers.pdf)

[Cheat Sheet with Slides](images/slides-images-containers.pdf)
___
# Section 3: Managing Data & Working with Volumes
*Writing, Reading & Persisting Data*

We will see how we can manage our data inside of images and containers.

* Understanding Different Kinds of Data

* Images, Containers & Volumes

* Using Arguments & Environment Variables
___
## Understanding Data Categories / Different Kinds of Data

Data Types

* Application
  * (Source Code + Environment)
  * Written & provided by you (= the developer)
  * Added to image and container in build phase
  * "Fixed": Cant' be change once image is built because images are **read-only**
  * **Read-only**, hence stored in images: 
    * Application and environment **should be** read-only and that is why we store it in an image.

* Temporary App Data (e.g. entered user input)
  * Fetched / Produced in running container
  * Stored in memory or temporary files
  * Dynamic and changing, but cleared regularly
  * **Read + write temporary, hence stored in containers**

* Permanent App Data (e.g. user accounts)
  * Fetched / Produced in running container
  * Stored in files or database
  * Must not be lost if container stops / restarts:
    * The data should actually even **survive the removal (deletion) of a container
  * **Read + write, permanent, stored with Containers & Volumes**
___
## Analyzing a Real App
*folder 005*

If you remove container or use `--rm` flag before you run it, your changes will lost. Because you removed your container and everything in it.

If you stop container with `$ docker stop` and restart it with `$ docker start` your changes will still be there because you didn't remove anything. You've just stopped your container.

But sometimes we want to make sure some data won't be deleted (like databases) even if container deleted. What will we do?
___
## Introducing Volumes
Volumes are **folders on your host machine** hard drive which are **mounted** ("made available", mapped) **into containers**

* Volumes **persist if a container shuts down**. If a container (re-)starts and mounts a volume, any data inside of that volume is **available in the container**.

* A container **can write** data into a volume **and read** data from it.

Dockerfile example:
```Dockerfile
FROM node:14

WORKDIR /app

COPY package.json /app

RUN npm install

COPY . /app

EXPOSE 80

VOLUME ["/app/feedback"]

CMD ["node", "server.js"]
```
___
## Named Volumes to the Rescue
Two Types of External Data Storages

* Volumes (Managed by Docker)
  * Anonymous Volumes
  * Named Volumes

  Either way, Docker sets up a folder / path on your host machine, exact location is unknown to you (=dev). Managed via `$ docker volume` commands.

* Bind Mounts (Managed by you)
  * You define a folder / path on your host machine (not docker)
  * Great for persistent, editable (by you) data (e.g. source code)

A defined path in the container is mapped to the created volume / mount. e.g /some-path on your hosting machine is mapped to /app/data.

Named volumes are great for data which should be persistent but which you don't need to edit directly.

`$ docker run -v <volume-name>:<workdir-app>/<path>`

For example of command above (look at the *folder 005*):

`$ docker run -v feedback:/app/feedback`

The commands above lets us to create anonymous volumes. The key difference is named volumes will not deleted by docker when the container shuts down. Anonymous volumes are deleted because they are recreated whenever a container is created. And therefore keepin them around after a container was removed makes no sense.

Anonymous volumes are closely attached to one specific container. Named volumes are not attached to a container.
___
## Removing Anonymous Volumes
We saw, that anonymous volumes are **removed automatically**, when a container is removed.

This happens when you start / run a container with the `--rm` option.

If you start a container **without that option**, the anonymous volume would **NOT be removed**, even if you remove the container (with `$ docker rm`).

Still, if you then re-create and re-run the container (i.e. you run `$ docker run` again), a **new anonymous volume will be created**. So even though the anonymous volume wasn't removed automatically, it'll also **not be helpful** because a different anonymous volume is attached the next time the container starts(i.e you removed the old container and run a new one).

Now you just start **pilling up a bunch of unused anonymous volumes** - you can **clear them** via `$ docker volume rm <volume-name>` or `$ docker volume prune`
___
## Getting Started With Bind Mounts (Code Sharing)
When you create image you can't change anything. But we need to always change, delete or add something new to our source code. With Bind Mounts we don't need to build a new image everytime we change our code.

* You define a folder / path on your host machine (not docker)
* Great for persistent, editable (by you) data (e.g. source code)

`$ docker run -v <local-file>:<workdir-folder>`

**Important**: Your local-file path must be **absolute path** not relative. And of course this can be folder, not just file.
___
## Bind Mounts - Shortcuts
Just a quick note: If you odn't alwyas want to copy and use the full path, you can use these **shortcuts**:

macOS / Linux: `-v $(pwd):/app

Windows: `-v "%cd%":/app`
___
## Combining & Merging Different Volumes
If we have a container, and we don't have a volume, and a bind mount, we can mount both into the contaier with a `-v` flag.

That means that some folders inside of the container are mounted, ar are connected to folders on the host machine. Now let's say we already had files inside of the container. Inthat case, they also now exist in the outside volume, and if you write a new file, it's also added in the folder on the host machine. If the container then stands up, and it finds files in the volume, and it doesn't have any internal files yet, it loads the files from the volume. That's actually what we utilize with the bind mount. Here we don't have any files inside of the container, let's say, but we have files on the local host machine. In taht case these files are basically also usable inside of the container. But now we have kind of both things happening. We have files inside of the container in the /app folder.
___
## Volumes & Bind Mounts - Quick Overview
* Anonymous Volume

  `$ docker run -v /app/data ...`

* Named Volume

  `$ docker run -v data:/app/data ...`

* Bind Mount

  `$ docker run -v /path/to/code:/app/code ...`

### Volumes - Comparison

* Anonymous Volumes

  * Created specifically for a single container.
  * Survives container shutdown / restart unless `--r` is used
  * Can not be shared across containers
  * Since it's anonymous, it can't be re-used (even on same image)

    Example of **temp** volume (anonymous volume):
    ```Dockerfile
    FROM node:14

    WORKDIR /app

    COPY package.json /app

    RUN npm install

    COPY . /app

    VOLUME ["/app/temp"]

    EXPOSE 80

    CMD ["npm", "start"]
    ```
* Named Volumes
  * Created in general - not tied to any specific container
  * Survives container shutdown / restart - removal via Docker CLI
  * Can be shared accross containers
  * Can be re-used for same container (across restarts)

* Bind Mounts
  * Location on host file, not tied to any specific container
  * Survives container shutdown / restart - removal on host file system
  * Can be shared across containers
  * Can be re-used for same container (across restarts)
___
## A Look at Read-Only Volumes
Default volumes are read + write. But when we change something in our source code we don't want to make this change in our container. But you can restrict that by adding an extra colon and then type ro (read only)

`$ docker run -v <path>:<workdir-folder>:ro`

Example:

`$ docker run -v "/home/cc/Desktop/docker_course/containers/005:/app:ro"`

Still, this is not all we need to do in this scenario, because you have to keep in mind that we bind this entire project folder as a bind mount.
___
## Managing Volumes
* List all the currently active volumes

  `$ docker volume ls`

* Create volume (docker will create for you. You don't need to do that unless you have good reason)

  `$ docker volume create <volume-name>`

* Inspecting a volume

  `$ docker volume inspect <volume-name>`
  
* Removing a volume

  `$ docker volume rm <volume-name>`

* Remove all unused volumes

  `$ docker volume prune`
___
## Using COPY vs Bind Mounts
If our bind mounts contains every file in our working directory, we can delete `COPY` line in our Dockerfile. That wouldn't change anything. **But** we use bind mounts when we developing, not while publish. When we publish our code, we definitely won't publish with our bind mount. So it's good to store snapshot of our working directory with `COPY` line in our Dockerfile.
___
## Don't COPY Everything: Using .dockerignore Files
`COPY . /app` instruction in Dockerfile copies everything in the folder where this Dockerfile lives, because we have `/app` there which means everything.

We can also restrict what gets copied here. And we do this by adding a `.dockerignore` file.

With .dockerignore we can specify which folders and files, should not be copied by a copy instruction.

### Adding More to the .dockerignore File
You can add more "to-be-ignored" files and folders to your `.dockerignore` file.

For example, consider adding the following to entries:

> Dockerfile

> .git 

this would ignore the `Dockerfile` itself as well as a potentially existing `.git` folder (if you are using Git in your project).

In general, you want to add anything which isn't required by your application to execute correctly.
___
## ARGuments & ENVironment Variables
Docker supports build-time **ARG**uments and runtime **ENV**ironment variables.

* ARG
  * Available inside of Dockerfile, NOT accessible in CMD or any application code
  * Set on image build (**docker build** via `--build-arg`)

* ENV
  * Available inside of Dockerfile & in application code
  * Set via ENV in Dockerfile or via `--env` on `$ docker run`

Dockerfile Example:
```Dockerfile
FROM node:14

WORKDIR /app

COPY package.json /app

RUN npm install

COPY . /app

ENV PORT 80

EXPOSE $PORT

CMD ["npm", "start"]
```

* Your only option is not hard code in the Dockerfile:

  `$ docker run --env PORT=8000`

  or

  `$ docker run -e PORT=8000`

* For multiple env:

  `$ docker run -e PORT=8000 -e APP=app.py`

* Read environment variables from file (e.g. `.env` file):

  `$ docker run --env-file .env`
___
## Environment Variables & Security
One important note about **environment variables and security**: Depending on which kind of data you're storing in your environment variables, you might not want to include the secure data directly in your `Dockerfile`.

Instead, go for a separate environment variables file which is then only used at runtime (i.e. when you run your container wiht `$ docker run`).

Otherwise, the values are "baked into the image" and everyone can read these values via `$ docker history <image>`.

For some values, this might not matter but for credentials, private keys etc. you definitely want to avoid that!

If you use a separate file, the values are not part of the image since you point at that file when you run `$ docker run`. But make sure you don't commit that separate file as part of your source control repository, if you're using source control (Git).
___
### Using Build Arguments (ARG)
With build time arguments, we can actually plug in different values into our Dockerfile, or into our image when we build that image without having to hard code these values into the Dockerfile.

* In Dockerfile example:
  ```Dockerfile
  FROM node:14

  WORKDIR /app

  COPY package.json /app

  RUN npm install

  COPY . /app

  ARG DEFAULT_PORT=80

  ENV PORT $DEFAULT_PORT

  EXPOSE $PORT

  CMD ["npm", "start"]
  ```

* In build example:

  `$ docker build feedback-node:dev --build-arg DEFAULT_PORT=8000`
___
## Module Summary
Containers can read + write data. **Volumes** can help with data storage, **Bind mounts** can help with direct container interaction.

* **Containers can read + write data**, but written **data is lost** if the container is removed

* **Volumes** are folders on the host machine, managed by Docker, which are mounted into the Container

* **Named Volumes** survive container removal and can therefore be used to store persistent data

* **Anonymous Volumes** are attached to a container - they can be used to save (temporary) data inside the container

* **Bind Mounts** are folders on the host machine which are specified by the user and mounted into containers - like **Named Volumes**

* **Build ARGuments** and **Runtime ENVironment** variables can be used to make images and containers more **dynamic / configurable**

[Slides Data Volumes](/images/slides-data-volumes.pdf)
[Cheat Sheet Data Volumes](/images/Cheat-Sheet-Data-Volumes.pdf)
___
# Section 4: Networking: (Cross-)Container Communication
* Contaienrs & External Networks

* Connectiong Containers with Networks
___
## Case 1: Container to WWW Communication
*folder 006*
Sending HTTP request to a website or web API
___
## Case 2: Container to Local Host Machine Communication
It could be web service or a database etc. and we wanna talk to that from iside our dockerized app.

Use `host.docker.internal` instead `localhost`
___
## Case 3: Container to Container Communication
Your application running inside of your container (e.g. mongodb) wants to talk to another container (NodeJS).

For `localhost` equivalent `$ docker container inspect <container-name>`

After that under the `"NetworkSettings"` there is `"IpAddress"`. Which is most time `"172.17.0.2"`

What you'll find is the **ip address** of your **container**.
___
## Creating Container Networks
With docker, you can create so-called **container networks**, also called just **networks**.

Networks:

You have multiple containers and you want to allow communication between these containers. In docker you can add containers in the same network by adding `--network-- option on the `$ docker run` command.

`$ docker run --network <network-name> ...`

Then all containers in network, will be able to talk to each other.

Within a docker network, all containers can communicate with each other and IPs are automatically resolved.

Unlike volumes, docker doesn't create networks. You have to create them on your own:

`$ docker network create <network-name>`

When containers in same network you have to use container name instaed of `localhost` in your code:

`http://<container-name>:<port>/<path>`

For example:

```javascript
mongoose.connect(
  'mongodb://mongodb:27017/swfavorites
)
```

in this case `mongodb` is **container name**.
___
## Docker Network Drivers

Docker Networks actually support different kinds of "**Drivers**" which influence the behavior of the Network.

The default driver is the "**bridge**" driver - it provides the behavior shown in this module (i.e. Containers can find each other by name if they are in the same Network).

the driver can be set when a Network is created, simply by adding the `--driver` option.

`$ docker network create --driver bridge my-net`

*Of course, if you want to use the "bridge" driver, you can simply omit the entire option since "bridge" is the default anyways*.

Docker also supports these alternative drivers - though you will use the "bridge" driver in most cases:

  * **host**: For standalone containers, isolation between container and host system is removed(i.e. they share localhost as a network)

  * **overlay**: Multiple Docker deamons (i.e. Docker running on different machiens) are able to connect with each other. Only works in "Swarm" mode which is a dated / almost deprecated way of connecting multiple containers.

  * **macvian**: You can set a custom MAC address to a container - this address can then be used for communication with that container.

  * **none**: All networking is disabled.

  * **Third-party plugins**: You can install third-party plugins which then may add all kinds of behaviors and functionalities.

As mentioned, the "**bridge**" driver makes most sense in the vast majority of scenarios.

[Slides Networking](images/slides-networking.pdf)
[Cheat Sheet Networks Requests](images/Cheat-Sheet-Networks-Requests.pdf)
___
# Section 5: Building Multi-Container Applications with Docker
* Combining Multiple Services To One App
* Working with Multiple Containers
___
## Demo App
### Dockerizing MongoDB Server

Create a mongodb server:
  
  `$ docker run --name mongodb --rm -d -p 27017:27017 mongo:latest`
___
### Dockerizing the Node App
* Dockerfile in */backend folder*

  ```Dockerfile
  FROM node:latest

  WORKDIR /app

  COPY package.json .

  RUN npm install

  COPY . .

  EXPOSE 80

  CMD ["node", "app.js"]
  ```

* Connect to Mongo
  `$ docker run --name goals-backend --net=host -p 80:80 --rm goals-node:localhost3000`
___
## Adding Docker Networks for Efficient Cross-Container Communication
___
## Adding Data Persistence to MongoDB with Volumes
* Learn `-v data:/data/db`

  `$ docker run --name mongodb -v data:/data/db --rm -d --network goals-net mongo`

* MongoDB Security in Docker

  `$ docker run --name mongodb -v data:/data/db --rm -d --network goals-net -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_ROOT_PASSWORD=secret mongo`
___
## Volumes, Bind Mounts & Polishing for the NodeJS Container
* Adding Volumes

  `$ docker run --name goals-backend -v /home/cc/Desktop/docker_course/containers/007/backend:/app -v logs:/app/logs -v /app/node_modules -d --rm -p 80:80 --network goals-net goals-node`

* When we start our container we run app.js with the node comand and that basically logs in the code at the point of time this containers starts.

* The node process loads all the code, and then starts that code, so to say.

* So if the code changes their author, `CMD ["node", "app.js"]` in Dockerfile, has no impact on the already running node server. We don't want that of course.

* We want the node server to restart whenever the code changes. Yes we could stop and restart our container, that would do the trick, but even better than that, we can add the extra dependency to this project, which then actually will restart the server automatically
for us when the code changes.

* For that we need to use `nodemon` in this case. Delete `package-lock.json` file and add 
  ```json
    "devDependencies": {
      "nodemon": "^2.0.4"
    }
  ```
  to package.json file

  and update "scripts" to this:
  ```json
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1",
      "start": "nodemon app.js"
  ``` 
    then update Dockerfile CMD instruction to CMD ["npm", "start"]
___
## Live Source Code Updates for the React Container (with Bind Mounts)
`$ docker run -v /home/cc/Desktop/docker_course/containers/007/frontend/src:/app/src --name goals-frontend --rm -p 3000:3000 -it goals-react`
___
## Module Summary
**Important**: This module is development only

[Slides Multi Container](/images/slides-multi-container.pdf)
___
# Section 6: Docker Compose: Elegant Multi-container Orchestration
Docker Compose makes managing multi-containers setups easier, it helps you automate the setup process and it allows you to bring up entire setup, with all the different containers and their individual and configurations with just one command. And also you can tear everything down with just one command.

* What is Docker Compose?
* Using Docker Compose
___
## Docker Compose: What & Why?

Docker Compose is a tool that allows us to use multiple `$ docker build` and multiple `$ docker run` commands with just one configuration file and then a set of orchestration commands to start all those services, all these containers at once and build all necessary images, if it should be required and you then also can use one command to stop everything and bring everything down and that will be great. Because with docker compose, you will have one configuration file using a clearly defined language which you can share anyone and then it's just one command, no command copy and pasting just one command which will leverage this configuration file and which will then bring up or bring down your entire multicontianer application.

What Docker Compose is **NOT**
* Docker Compose does **NOT replace Dockerfiles** for custom Images
* Docker Compose does **NOT replace Images or Containers**
* Docker Compose is **NOT suited** for managing multiple containers on **different hosts (machines)**

Writing Docker Compose Files
* Services (Containers)
  * PUblished Ports
  * Environment Variables
  * Volumes
  * Networks

    Basically everything you can do with docker command in the terminal
___
## Creating a Compose File
We need to create `docker-compose.yaml` file.

`version` : Docker configurationg version you want to use

`services` : Basically containers
___
## Diving into the Compose File Configuration
Notes:
* You don't need to use `--rm` and `-d` options for containers because by default when you use Docker Compose, when you bring your services down they will be removed and regarding the detached mode.

* You don't need to use `--network` anymore. Docker Compose automatically add everything under `services:` to the same network.
___
## Installing Docker Compose on Linux
on macOS and Windows, you should already have Docker Compose installed - it's set up together with Docker there.

On linux machines, you need to install it separately.

These steps should get you there:

1. `$ sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`

2. `$ sudo chmod +x /usr/local/bin/docker-compose`

3. `$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose`

4. to verify: `$ docker-compose --version`

or just: `$ sudo apt install docker-compose`
___
## Docker compose Up & Down
* Detached Mode
  
  `$ docker-compose up -d`

* Stop all services and remove all containers (it does **not** delete **volumes** though)
  
  `$ docker-compose down`

* Delete volumes

  `$ docker-compose down -v`
___
## Working with Multiple Containers

* Build
  
  `build: <Dockerfile-path>`

* `depends_on:` Option

  With Docker Compose where we create and launch multiple services, so multiple containers at the same time, sometimes one ocntainer might depend on another container to be up and running already.

  In *folder 007* example: `backend` actually depends on `mongodb` being up and running already because `backend` container wants to connect to MongoDB.

* Interactive Mode
  * `stdin_open: true`
    * In here we are telling docker this container needs input from outside
  * `tty: true`
    * In here we are telling docker this container should be **interactive**
___
## Building Images & Understanding Container Names

* Build needed images and then start:

  `$ docker-compose up --build`

* Just build images

  `$ docker-compose build`

* Signing your own container name in `docker-compose.yaml` file:

  `container_name: <container-name>`
___
## yaml File Example
```yaml
version: "3.8"
services:
  mongodb:
    image: 'mongo'
    volumes: 
      - data:/data/db
    # environment: 
    #   MONGO_INITDB_ROOT_USERNAME: max
    #   MONGO_INITDB_ROOT_PASSWORD: secret
      # - MONGO_INITDB_ROOT_USERNAME=max
    env_file: 
      - ./env/mongo.env
  backend:
    build: ./backend
    # build:
    #   context: ./backend
    #   dockerfile: Dockerfile
    #   args:
    #     some-arg: 1
    ports:
      - '80:80'
    volumes: 
      - logs:/app/logs
      - ./backend:/app
      - /app/node_modules
    env_file: 
      - ./env/backend.env
    depends_on:
      - mongodb
  frontend:
    build: ./frontend
    ports: 
      - '3000:3000'
    volumes: 
      - ./frontend/src:/app/src
    stdin_open: true
    tty: true
    depends_on: 
      - backend

# Named volumes should be specified in here also
volumes: 
  data:
  logs:

```

[Slides Docker Compose](images/slides-docker-compose.pdf)
[Cheat Sheet Docker Compose](images/Cheat-Sheet-Docker-Compose.pdf)
___
# Section 7: Working with "Utility Containers" & Executing Commands in Containers
*folder 008*

Until now we use application containers

* Application Containers
  * Environment
  * My App
    
    and then 

  `$ docker run myapp` : Runs CMD and starts app
* Utility Containers (Which only have a certain environment in them (php, nodejs))

  `$ docker run <mynpm> init` : Executes / Appends custom command
___
## Utility Containers: Why would you use them?
Imagine you need to some files needs to created by a service but you don't have it.

For example for NodeJS, if you don't want to create and setup `package.json` file by yourself, you need to run `npm init` command. But you don't have `npm` on your local machine. That's why we need utility containers.
___
## Different Ways of Running Commands in Containers
The `$ docker exec` command allows you to execute certain commands inside of a running container besides the default command, this container executes. So besides the command that might've been specified in a Dockerfile, that command still continues running. So the application still continues to run but you can't run addititonal commands inside of a container. That's something which sometimes could be useful.

For example:

  `$ docker exec -it <container-name> npm init`

    or
  
  `$ docker run -it node npm init`
___
## Building a First Utility Container
Build image with : `$ docker build -t node-util .`

And then `$ docker run -it node-util npm init`

By default this command would run in the app folder inside of the container. Now we want to mirror it to our local folder so that what I create in the container is also available here on my host machine. So that I can create a project on my host machine with help of a container.

That's the idea behind having a utility container, we can use it to execute something which has an effect on the host machine without having to install all the extra tools on the host machine.

We can use a feature for that, which we already used many times before in the course, we can use a **bind mount** with `-v` in command.

`$ docker run -it -v /home/cc/Desktop/docker_course/containers/008:/app node-util npm init`

This command will create us `package.json` file in our `/app` folder. Now we can completely delete this container and it's image node-util because now we can run `npm init` in our Dockerfile.
___
## Utilizing ENTRYPOINT
When we built our first utility container we use bind mounts. If we accidently do something wrong in those commands, our folder would be messed up. So we need to restrict to commands we can run to protect ourselves.

For this we can use extra instuction in Dockerfile and that's the `ENTRYPOINT` instruction.

It is quite similar to the command instruction, but it has one key difference:

If we had a command after the image name on docker run, then `npm ...` command, overrites the `CMD` in Dockerfile (if there is one).

With the `ENTRYPOINT` that's different. For the `ENTRYPOINT`, whatever you enter after your image name on `$ docker run` is **appended** after the `ENTRYPOINT`. That means that we could specify `npm`. And now we could apppend any `npm` command after our image name once we rebuild this image.

`$ docker run -it -v /home/cc/Desktop/docker_course/containers/008:/app mynpm init`
___
## Using docker-compose with Utility Containers
`$ docker-compose run` allows us to run as single service from `yaml` file in case we had multiple services, here we have only one, but if we had multiple services, we could asl target a single service by the service name. And then, with any command that should be upended after our entry point of our choice, so for example:

`$ docker-compose run npm init`

If you want to remove container after it shuts down:

`$ docker-compose run --rm npm init`
___
## Utility Containers, Permissions & Linux
When working with "Utility Containers" on Linux, take a closer look at this very helpful [thread](https://www.udemy.com/course/docker-kubernetes-the-practical-guide/learn/#questions/12977214/).

This thread discusses user permissions as set by Docker when working with "Utility Containers" and how you should tweak them.

[Slides Utility Containers](images/slides-utility-containers.pdf)
___
# Section 8: A More Complex Setup: A Laravel & PHP Dockerized Project
[Slides Laravel](images/slides-laravel.pdf)
___
# Section 9: Deploying Docker Containers

Build your image and push it to Docker Hub

`$ docker push <image-name>`

## From Development to Production

Things to Watch Out For:
* **Bind Mounts shouldn't** be used in Production
* Containerized apps **might need a build step** (some applications (e.g. react apps))
* **Multi-Container projects** might need to be **split** (or should be split) across multiple hosts / remote machines
* **Trade-offs** between **control** and **responsibility** be worth it!
___
## Deployment Process & Providers
A Basic First Example : Standalone NodeJS App
Hosting Providers: There are lot of docker hosting providers. Pick one. AWS, Azure, Google Cloud -> These are 3 big hosting providers.
___
## Getting Started With an Example
## Bind Mounts In Production
Bind Mounts, Volumes & COPY

* In Development
  * Containers should encapsulate the runtime environment but not necessarily the code.
  * Use "Bind Mounts" to provide your local host project files to the running container
  * Allows for instant updates without restarting the container
* In Production
  * Image / Container is the **single source of truth**. A container should really work standalone, you should NOT have source code on your remote machine
  * Use COPY to copy a code snapshot into the image
  * Ensures that every image runs without any extra, surrounding configuration or code
___
