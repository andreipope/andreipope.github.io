---
title: How to Containerize a JavaScript application
description: This tutorial shows how you can build a Docker image that contains a JavaScript application.
layout: tutorial
permalink: /tutorials/containerize-javascript-application
---


This tutorial shows how you can build a Docker image that contains a JavaScript application.

You will learn to:

* Create a bare-bones JavaScript application with NuxtJS
* Configure a Dockerfile for your NuxtJS application
* Build a Docker image
* Run your application as a container
* Push your image to Docker Hub
* Run your image from Docker Hub

## Prerequisites

* {% include prerequisites-node-js.md %}
* {% include prerequisites-docker.md %}
* {% include prerequisites-docker-hub.md %}

## Create a JavaScript application with NuxtJS

1. Open a new terminal window, and create a working folder. In the working folder, enter the following command to create a new project:

    ```Bash
    npx create-nuxt-app nuxtjs-helloworld
    ```

    You will be prompted to answer a few questions:

    ```
    create-nuxt-app v2.15.0
    ✨  Generating Nuxt.js project in nuxtjs-helloworld
    ? Project name nuxtjs-helloworld
    ? Project description My awesome Nuxt.js project
    ? Author name Andrei Popescu
    ? Choose programming language JavaScript
    ? Choose the package manager Npm
    ? Choose UI framework None
    ? Choose custom server framework None (Recommended)
    ? Choose Nuxt.js modules (Press <space> to select, <a> to toggle all, <i> to invert selection)
    ? Choose linting tools ESLint
    ? Choose test framework None
    ? Choose rendering mode Universal (SSR)
    ? Choose development tools (Press <space> to select, <a> to toggle all, <i> to invert selection)

    🎉  Successfully created project nuxtjs-helloworld

      To get started:

        cd nuxtjs-helloworld
        npm run dev

      To build & start for production:

        cd nuxtjs-helloworld
        npm run build
        npm run start
    ```

    >Note that the above output was truncated for brevity.

    The `npx create-nuxt-app` installs the dependencies, and then creates the default directory structure needed by a NuxtJS application:

    ```Bash
    tree nuxtjs-helloworld -L 1
    ```

    ```
    nuxtjs-helloworld
    |-- README.md
    |-- assets
    |-- components
    |-- layouts
    |-- middleware
    |-- node_modules
    |-- nuxt.config.js
    |-- package-lock.json
    |-- package.json
    |-- pages
    |-- plugins
    |-- static
    `-- store

    9 directories, 4 files
    ```

2. Use the following command to start your application:

    ```bash
    cd nuxtjs-helloworld && npm run dev
    ```

    ```
    > nuxtjs-helloworld@1.0.0 dev /Users/andrei/Documents/test/nuxtjs-helloworld
    > nuxt


       ╭─────────────────────────────────────────────╮
       │                                             │
       │   Nuxt.js v2.12.2                           │
       │   Running in development mode (universal)   │
       │                                             │
       │   Listening on: http://localhost:3000/      │
       │                                             │
       ╰─────────────────────────────────────────────╯

    ℹ Preparing project for development                                              19:45:18
    ℹ Initial build may take a while                                                 19:45:18
    ✔ Builder initialized                                                            19:45:18
    ✔ Nuxt files generated                                                           19:45:19

    ✔ Client
      Compiled successfully in 9.76s

    ✔ Server
      Compiled successfully in 8.58s

    ℹ Waiting for file changes                                                       19:45:30
    ℹ Memory usage: 178 MB (RSS: 272 MB)                                             19:45:30
    ℹ Listening on: http://localhost:3000/                                           19:45:30
    ```

3. Open a browser and visit [http://localhost:3000](http://localhost:3000). You should see a page similar to that shown below:

![NuxtJS Hello World](/assets/img/nuxtjs-hello-world.png)

## Create a Dockerfile for your JavaScript application

1.  Use a plain-text editor to create a file named `Dockerfile` and copy in the following content:

    ```Dockerfile
    FROM node:10
    WORKDIR /usr/src/app
    COPY . /usr/src/app/
    RUN npm install
    EXPOSE 3000
    CMD [ "npm", "run", "dev" ]
    ```

    This file provides the instructions the Docker engine needs to create a container image. The way this works is that the Docker engine creates a layer for each instruction found in the `Dockerfile` and places it atop of the previous layers.

    The following list explains each line:

    * `FROM node:10` specifies the base image (`node 10`)
    * `WORKDIR /usr/src/app` sets `/usr/src/app` as the working directory for the `COPY`, `RUN`, and `CMD` commands
    * `COPY . /usr/src/app/` is used to copy files from the host to the container
    * `RUN npm install` installs the dependencies
    * `EXPOSE 3000` specifies that your container listens on port `3000`
    * `CMD [ "npm", "run", "dev" ]` sets the default command (`nmp run dev`) that'll be run when an instance of your container image is deployed

2. To specify the list of files the Docker engine should ignore when generating  the build context, create a file called `.dockerignore` with the following content:

    ```
    node_modules
    npm-debug*
    .nuxt
    ```

## Build an image

1.  Build an image for your NuxtJS application. Enter the `docker build` command, and use the `-t` flag to specify the name of the image.
    >Note that the name of the image is in the following format `<YOUR-DOCKER-HUB-USERNAME>/<IMAGE_NAME>`.

    The following example command builds an image named `andreipopescu12/nuxtjs-helloworld`, and sets the path to the context to the current directory:

    ```Bash
    docker build -t andreipopescu12/nuxtjs-helloworld .
    ```

    ```
    Sending build context to Docker daemon  125.3MB
    Step 1/6 : FROM node:10
    10: Pulling from library/node
    56da78ce36e9: Pull complete
    fbfe0f13ac45: Pull complete
    6254ff6d0e60: Pull complete
    e0e1e13bd9f6: Pull complete
    b86b38b40a24: Pull complete
    e357e1a6c1b2: Pull complete
    26ead3dd6706: Pull complete
    2a074406f86d: Pull complete
    2bb91d5c5247: Pull complete
    Digest: sha256:816cfaee24dc2cea534e21d7f9c55f3b22c8bc6af61d8445f8d0178168ef3b28
    Status: Downloaded newer image for node:10
     ---> 01b816051d34
    Step 2/6 : WORKDIR /usr/src/app
     ---> Running in 2480e9e38114
    Removing intermediate container 2480e9e38114
     ---> a8d4d9e1b7ac
    Step 3/6 : COPY . /usr/src/app/
     ---> fad0616f58b3
    Step 4/6 : RUN npm install
     ---> Running in f1bfc0494664

    audited 21125 packages in 21.523s

    49 packages are looking for funding
      run `npm fund` for details

    found 0 vulnerabilities

    Removing intermediate container f1bfc0494664
     ---> 374518fec47e
    Step 5/6 : EXPOSE 3000
     ---> Running in c8c78265d768
    Removing intermediate container c8c78265d768
     ---> de05a90681ca
    Step 6/6 : CMD [ "npm", "run", "dev" ]
     ---> Running in b092f80697d3
    Removing intermediate container b092f80697d3
     ---> 61c719d0055b
    Successfully built 61c719d0055b
    Successfully tagged andreipopescu12/nuxtjs-helloworld:latest
    ```

2. You can list the Docker images available on your computer as follows:

    ```Bash
    docker images
    ```

    ```
    REPOSITORY                          TAG                 IMAGE ID            CREATED             SIZE
    andreipopescu12/nuxtjs-helloworld   latest              61c719d0055b        4 minutes ago       1.02GB
    node                                10                  01b816051d34        5 days ago          911MB
    ```

## Run your application as a container

1. To run the `andreipopescu12/nuxtjs-helloworld` image, enter the `docker run` command, and pass it the following arguments:

    * `-p` with the port on the host (3000) that’ll be forwarded to the container (3000), separated by `:`
    * `-d` to specify that the container must be run in the background
    * The name of the image (this example uses `andreipopescu12/nuxtjs-helloworld`, but yours will be different)

    ```Bash
    docker run -p 3000:3000 -d andreipopescu12/nuxtjs-helloworld
    ```

    ```
    84154f4301b26abad077865ea903e620d826d81ce1f4f74e1b197cca8d2cdcb5
    ```

2. Display the list of containers running on your computer with the following command:

    ```Bash
    docker ps
    ```

    ```
    CONTAINER ID        IMAGE                               COMMAND                  CREATED             STATUS              PORTS                    NAMES
    84154f4301b2        andreipopescu12/nuxtjs-helloworld   "docker-entrypoint.s…"   56 seconds ago      Up 55 seconds       0.0.0.0:3000->3000/tcp   festive_nash
    ```

3. You can see the logs by entering the `docker logs` command followed by the `id` of your container (this example uses `84154f4301b2`, but yours will be different):

    ```Bash
    docker logs 84154f4301b2
    ```

    ```
    > nuxtjs-helloworld@1.0.0 dev /usr/src/app
    > nuxt

    ℹ Listening on: http://localhost:3000/
    ℹ Preparing project for development
    ℹ Initial build may take a while
    ✔ Builder initialized
    ✔ Nuxt files generated
    ℹ Compiling Client
    ℹ Compiling Server
    ✔ Server: Compiled successfully in 10.40s
    ✔ Client: Compiled successfully in 11.33s
    ℹ Waiting for file changes
    ℹ Memory usage: 175 MB (RSS: 270 MB)
    ℹ Listening on: http://localhost:3000/
    ```

4. You can use the `docker top` command to see the process running inside your container:

    ```Bash
    docker top 84154f4301b2
    ```

    ```
    PID                 USER                TIME                COMMAND
    2746                root                0:00                npm
    2791                root                0:00                sh -c nuxt
    2792                root                0:20                node /usr/src/app/node_modules/.bin/nuxt
    ```

5. To see your application in action, point your browser to [http://localhost:3000](http://localhost:3000):

    ![NuxtJS Hello World](/assets/img/nuxtjs-hello-world.png)


6. Now that everything works as expected, use the `docker kill` command to stop your container:

    ```Bash
    docker kill 84154f4301b2
    ```

    ```
    84154f4301b2
    ```
7. Delete the container by entering the `docker rm` command, and passing it the `id` of your container (this example uses `84154f4301b2`, but yours will be different):

    ```Bash
    docker rm 84154f4301b2
    ```

    ```
    84154f4301b2
    ```

## Push your image to Docker Hub


1. Log in to Docker Hub. Run the `docker login` command specifying the `--username` flag with your Docker Hub user name ( this example uses `andreipopescu12`, but your user name will be different):

    ```Bash
    docker login --username andreipopescu12
    ```

    You will be prompted to enter your Docker Hub password:

    ```
    Password:
    Login Succeeded
    ```

2. Push your image to Docker. Type the `docker push` command followed by the name of your image (this example uses `andreipopescu12/nuxtjs-helloworld`, but yours will be different):

    ```Bash
    docker push andreipopescu12/nuxtjs-helloworld
    ```

    ```
    The push refers to repository [docker.io/andreipopescu12/nuxtjs-helloworld]
    d3ff542cca46: Pushed
    9e5f854f4ab1: Pushed
    88f011bd5729: Pushed
    2f942335d8fb: Mounted from library/node
    2144a19162ed: Mounted from library/node
    f1cac044aca7: Mounted from library/node
    45ac74adb5b4: Mounted from library/node
    d485cbbe6a5e: Mounted from library/node
    391c89959588: Mounted from library/node
    588545a7a2a3: Mounted from library/node
    8452468a5e50: Mounted from library/node
    55b19a5e648f: Mounted from library/node
    latest: digest: sha256:3b90e66f215cf6ff5a718dd8a3576c2e67f226b790a312e9741be9a45a044353 size: 2844
    ```

3. Now you can delete the image from your computer:

    ```Bash
    docker rmi andreipopescu12/nuxtjs-helloworld --force
    ```

    ```
    Untagged: andreipopescu12/nuxtjs-helloworld:latest
    Untagged: andreipopescu12/nuxtjs-helloworld@sha256:3b90e66f215cf6ff5a718dd8a3576c2e67f226b790a312e9741be9a45a044353
    Deleted: sha256:61c719d0055b319856d0d885952a5c9604032c8e74efdfe79c7cc2c82b05715d
    Deleted: sha256:de05a90681ca38f2c3c80da9429b76314edec9352ba34c93e6383f34d76081ee
    Deleted: sha256:374518fec47eeb2095d1424f594fb06f20106b6ad18e38abc79d90a85cec93a2
    Deleted: sha256:fad0616f58b3d990ed5c552bf21e5eb815667b92a273fbfe16c2a6e286bcf1f7
    Deleted: sha256:a8d4d9e1b7ac3c1b2da0c2d5cddb863895c9246f618bac467cced4b02c2cfbbf
    ```


## Run your image from Docker Hub

1. The following `docker run` commands downloads the `andreipopescu12/nuxtjs-helloworld` image to your computer and then runs it:

    ```Bash
    docker run -p 3000:3000 -d andreipopescu12/nuxtjs-helloworld
    ```

    ```
    Unable to find image 'andreipopescu12/nuxtjs-helloworld:latest' locally
    latest: Pulling from andreipopescu12/nuxtjs-helloworld
    56da78ce36e9: Already exists
    fbfe0f13ac45: Already exists
    6254ff6d0e60: Already exists
    e0e1e13bd9f6: Already exists
    b86b38b40a24: Already exists
    e357e1a6c1b2: Already exists
    26ead3dd6706: Already exists
    2a074406f86d: Already exists
    2bb91d5c5247: Already exists
    f69924c128cf: Already exists
    c25d72ae961f: Already exists
    6b358121e530: Already exists
    Digest: sha256:3b90e66f215cf6ff5a718dd8a3576c2e67f226b790a312e9741be9a45a044353
    Status: Downloaded newer image for andreipopescu12/nuxtjs-helloworld:latest
    e1e7fa715378243963dac8c9cb3d47d60b3d73fa717c14beaa2b223deabc5e99
    ```


2. Use the `docker ps` command to make sure that your application is running:

    ```Bash
    docker ps
    ```

    ```
    CONTAINER ID        IMAGE                               COMMAND                  CREATED             STATUS              PORTS                    NAMES
    e1e7fa715378        andreipopescu12/nuxtjs-helloworld   "docker-entrypoint.s…"   57 seconds ago      Up 56 seconds       0.0.0.0:3000->3000/tcp   beautiful_pare
    ```

3. You can now point your browser to [http://localhost:3000](http://localhost:3000):

    ![NuxtJS Hello World](/assets/img/nuxtjs-hello-world.png)


4. Use the `docker kill` command to stop your container:

    ```Bash
    docker kill e1e7fa715378
    ```

    ```
    e1e7fa715378
    ```

5. Use the `docker rm` command to delete your container:

    ```Bash
    docker rm e1e7fa715378
    ```

    ```Bash
    e1e7fa715378
    ```

## Clean up

Images can be large, and you usually want to delete the ones created while developing your application.

1. List the images available on your computer:

    ```Bash
    docker images
    ```

    ```
    REPOSITORY                          TAG                 IMAGE ID            CREATED             SIZE
    andreipopescu12/nuxtjs-helloworld   latest              61c719d0055b        About an hour ago   1.02GB
    node                                10                  01b816051d34        5 days ago          911MB
    ```

2. Delete your image (this example uses `andreipopescu12/nuxtjs-helloworld`, but yours will be different):

    ```Bash
    docker rmi andreipopescu12/nuxtjs-helloworld --force
    ```

    ```
    Untagged: andreipopescu12/nuxtjs-helloworld:latest
    Untagged: andreipopescu12/nuxtjs-helloworld@sha256:3b90e66f215cf6ff5a718dd8a3576c2e67f226b790a312e9741be9a45a044353
    Deleted: sha256:61c719d0055b319856d0d885952a5c9604032c8e74efdfe79c7cc2c82b05715d
    ```

3. You can also delete the base image (`node:10`):

    ```Bash
    docker rmi node:10
    ```

    ```
    Untagged: node:10
    Untagged: node@sha256:816cfaee24dc2cea534e21d7f9c55f3b22c8bc6af61d8445f8d0178168ef3b28
    Deleted: sha256:01b816051d343a5aaa3b33e165f6b6d620b7e899961bf6e8955a7577a0129873
    ```

---

In this tutorial, you learned how to build a Docker image that contains a JavaScript application.

Thanks for reading!
