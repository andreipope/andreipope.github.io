---
title: Package an Express.JS application as a multi-container application with Docker compose
description: This tutorial is designed to show how you can package an Express.JS as a multi-container application with Docker compose.
layout: tutorial
permalink: tutorials/package-expressjs-as-a-multi-container-application-with-docker-compose
---

This tutorial is designed to show how you can package an ExpressJS application as a multi-container application with Docker compose.

You will learn to:

## Prerequisites

* {% include prerequisites-node-js.md %}
* {% include prerequisites-docker.md %}


## Initialize a new project and install the dependenices

1. Install the Ghost command-line tool globally by entering the following command:

    ```Bash
    npm init -y
    ```

    ```
    Wrote to /Users/andrei/Documents/test/docker-compose-tutorial/package.json:

    {
      "name": "docker-compose-tutorial",
      "version": "1.0.0",
      "description": "",
      "main": "index.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "keywords": [],
      "author": "",
      "license": "ISC"
    }
    ```

2. Install dependencies:

    ```Bash
    npm install express mysql --save
    ```

    ```
    npm notice created a lockfile as package-lock.json. You should commit this file.
    npm WARN docker-compose-tutorial@1.0.0 No description
    npm WARN docker-compose-tutorial@1.0.0 No repository field.

    + express@4.17.1
    + mysql@2.18.1
    added 59 packages from 48 contributors and audited 139 packages in 4.187s
    found 0 vulnerabilities
    ```

3. Create a a file called `app.js` with the following content:

    ```JavaScript
    const express = require('express')
    const app = express()
    const port = 3000
    var mysql = require('mysql')

    const con = mysql.createConnection({
      host: "localhost",
      user: "yourusername",
      password: "yourpassword",
      database: "yourdatabase"
    })

    con.connect(function(err) {
      if (err) throw err;
      console.log("Connected!");
    })

    const sql1 = 'CREATE TABLE IF NOT EXISTS `visitors` (`count` int)'
    con.query(sql1, function (err, result) {
      if (err) throw err;
    })

    const sql2 = "INSERT INTO visitors (count) VALUES ('0')";
      con.query(sql2, function (err, result) {
        if (err) throw err;
        console.log("1 record inserted");
      })


    app.get('/', (req, res) => {
      con.query("SELECT count FROM visitors", function (err, result, fields) {
        if (err) throw err;
        let count = result[0].count
        if (count === undefined) {
          count = 0
        } else {
          count ++
          let sql = `UPDATE visitors SET count = ${count}`
          con.query(sql, function (err, result) {
            if (err) throw err;
              console.log("1 record inserted");
          })
        }
        res.send(`You are visitor number ${count}`)
      });
    })

    app.listen(port, () => console.log(`Example app listening at http://localhost:${port}`))
    ```

4. Create a file called `Dockerfile` and paste in the following content:

    ```Dockerfile
    FROM node:10
    WORKDIR /usr/src/app
    COPY . /usr/src/app/
    RUN npm install
    EXPOSE 3000
    CMD [ "node", "app.js" ]
    ```


5. To specify the list of files the Docker engine should ignore when generating the build context, create a file called `.dockerignore` with the following content:

    ```
    node_modules
    npm-debug*
    ```

6. Build an image for your ExpressJS application. Enter the `docker build` command, and use the `-t` flag to specify the name of the image.
    >Note that the name of the image is in the following format `<YOUR-DOCKER-HUB-USERNAME>/<IMAGE_NAME>`.

    The following example command builds an image named `andreipopescu12/docker-compose-expressjs`, and sets the path to the context to the current directory:

    ```Bash
    docker build -t andreipopescu12/docker-compose-expressjs .
    ```

    ```
    Sending build context to Docker daemon  23.55kB
    Step 1/6 : FROM node:10
    10: Pulling from library/node
    7568c21980bd: Pull complete
    4a9f2207c812: Pull complete
    6fe350d2b140: Pull complete
    d95a2fdc8b3d: Pull complete
    6e9eb1fbe207: Pull complete
    d56555bcbc0a: Pull complete
    8ca024298cd0: Pull complete
    871d3382b483: Pull complete
    1102201760ac: Pull complete
    Digest: sha256:a5dbd2049787dd255be075ca081f3d0cf4dc0039bef76125aefa43eec25933d4
    Status: Downloaded newer image for node:10
    ---> bd837ed69497
    Step 2/6 : WORKDIR /usr/src/app
    ---> Running in b5f21f51505e
    Removing intermediate container b5f21f51505e
    ---> cd48cfc5a044
    Step 3/6 : COPY . /usr/src/app/
    ---> 58f2d49bc5c1
    Step 4/6 : RUN npm install
    ---> Running in 28b54252e691
    npm WARN docker-compose-tutorial@1.0.0 No description
    npm WARN docker-compose-tutorial@1.0.0 No repository field.

    added 59 packages from 48 contributors and audited 139 packages in 3.1s
    found 0 vulnerabilities

    Removing intermediate container 28b54252e691
    ---> a1225c2f71db
    Step 5/6 : EXPOSE 3000
    ---> Running in b21de6284b90
    Removing intermediate container b21de6284b90
    ---> 8499f6399322
    Step 6/6 : CMD [ "node", "app.js" ]
    ---> Running in e980cdba3c2f
    Removing intermediate container e980cdba3c2f
    ---> bfd59424cbdb
    Successfully built bfd59424cbdb
    Successfully tagged andreipopescu12/docker-compose-expressjs:latest
    ```

7. You can push the `docker-compose-expressjs` image to Docker Hub by running the following command:

    ```Bash
    docker push andreipopescu12/docker-compose-expressjs
    ```

    ```
    The push refers to repository [docker.io/andreipopescu12/docker-compose-expressjs]
    ee7764a6d718: Pushed
    a3c14e55c76b: Pushed
    2236e41f6d5d: Pushed
    5cc1b11e8894: Mounted from library/node
    4379817b82b1: Mounted from library/node
    c7b80f4f0500: Mounted from library/node
    d7369641c400: Mounted from library/node
    b829f6ba4e1d: Mounted from library/node
    a9286fedbd63: Mounted from library/node
    d50e7be1e737: Mounted from library/node
    6b114a2dd6de: Mounted from library/node
    bb9315db9240: Mounted from library/node
    latest: digest: sha256:cd5ab6b56e384a40ff58705a183f33e86e1dfc4c6e52c10d7868bdc2f5603c82 size: 2841
    ```

## Docker compose

1. Create a file named `docker-compose.yaml` and copy in the following spec:

    ```YAML
    8. Create a file named `docker-compose.yml`, and copy in the following spec:

```YAML
  version: "3.7"
  services:
    app:
      container_name: docker-compose-expressjs
      image: andreipopescu12/docker-compose-expressjs
      restart: always
      ports:
        - "3000:3000"
      expose:
        - "3000"
      links:
        - mysql
      depends_on:
        - mysql
    mysql:
      container_name: docker-compose-mysql
      image: mysql/mysql-server:5.7
      ports:
        - "3306:3306"
      expose:
        - "3306"
      environment:
        MYSQL_USER: "yourusername"
        MYSQL_PASSWORD: "yourpassword"
    ```
---
1. Run MySQL

```
docker run --name=mysql-demo -p 3306:3306 \
-e MYSQL_USER=yourusername \
-e MYSQL_PASSWORD=yourpassword \
-e MYSQL_DATABASE=yourdatabase \
--mount type=bind,src=$(PWD)/mysql/my.cnf,dst=/etc/my.cnf \
--mount type=bind,src=$(PWD)/mysql/datadir,dst=/var/lib/mysql \
-d mysql/mysql-server:5.7
```

