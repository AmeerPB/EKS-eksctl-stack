

> [!NOTE]
> Creating an application


<br />
<br />


> [!TIP]
> 
> ## Reference
> https://www.youtube.com/watch?v=tYAenovWCRA

<br />
<br />

### From the AWS Cloudshell, init a NodeJS app


```bash

[cloudshell-user@ip-10-130-29-180 node-app]$ npm init -y
Wrote to /home/cloudshell-user/node-app/package.json:

{
  "name": "app",
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

<br />
<br />

### Save the application

``` bash
[cloudshell-user@ip-10-130-29-180 node-app]$ npm i express -S

added 64 packages, and audited 65 packages in 6s

12 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
npm notice 
npm notice New major version of npm available! 9.8.1 -> 10.8.1
npm notice Changelog: https://github.com/npm/cli/releases/tag/v10.8.1
npm notice Run npm install -g npm@10.8.1 to update!
npm notice 


```
<br />
<br />

### Create an index.js file

``` bash
const express = require("express");

const app = express();
app.get("/", (req,res) => {
  res.send("Hello world from Ameer !");
});

app.listen(30001, () => console.log("NodeJS app started on port 30001"));

```

<br />
<br />

### Create a Dockerfile

``` bash

FROM node:22-alpine3.19

WORKDIR /app

COPY package*.json ./

RUN npm install -g pm2 &&  npm install --production

COPY . .

EXPOSE 30001

#CMD node index.js
CMD ["pm2-runtime", "ecosystem.config.js"]

```

> [!TIP]
>
>  Global Installation of PM2: `npm install -g pm2`
> Local Installation of Project Dependencies: `npm install`
>
> npm: The Node.js package manager.
> `install -g pm2`  : This command installs PM2 globally on the Docker container.
> -g flag indicates a global installation, meaning PM2 will be available system-wide in the Docker container.
> pm2 is a process manager for Node.js applications. It allows you to keep your applications alive forever, reload them without downtime, and facilitate common system admin tasks.
>
> npm: The Node.js package manager.
> `install`: This command installs all the dependencies listed in your project's package.json file into the node_modules directory within the working directory (/usr/src/app).
>
>
> `npm install --production`
> `npm install`: This command installs all dependencies listed in the dependencies and >devDependencies sections of your package.json file by default.
> `--production`: This flag tells npm to install only the dependencies listed under dependencies in your package.json file and to ignore devDependencies.


<br />
<br />



### Additional Configuration: PM2 Ecosystem File
If your application needs an ecosystem file for PM2, create ecosystem.config.js in your application directory:

``` bash

module.exports = {
  apps: [{
    name: 'node-app',
    script: './app.js', // Your entry point file
    instances: 'max',
    exec_mode: 'cluster'
  }]
};


```


> [!TIP]
> Explanation
> module.exports:
> 
> This line exports an object so that PM2 can read and use it.
> apps:
> 
> This is an array of application configurations. You can define multiple applications in this array, but in this case, there's only one application configuration.
> name:
> 
> This sets the name of the application as 'node-app'. This name will be used by PM2 to manage the application.
> script:
> 
> This specifies the entry point file of your application, which in this case is ./app.js. PM2 will start this script to run your application.
> instances:
> 
> This property is set to 'max', which means PM2 will automatically scale the number of instances to the number of available CPU cores. This utilizes the full potential of the machine's CPU by running multiple instances of the application.
> exec_mode:
> 
> This property is set to 'cluster'. In cluster mode, PM2 will run multiple instances of the application, each on a different CPU core. This provides better performance and reliability as it allows the application to handle more requests and ensures that if one instance crashes, the others can continue running.



<br />
<br />


### K8s deployment and service file

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-hello-world-app
  namespace: testing
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nodejs-hello-world-app
  template:
    metadata:
      labels:
        app: nodejs-hello-world-app
    spec:
      containers:
      - name: nodejs-hello-world-app
        image: openrc/nodejs-hello-world:1.0
        ports:
        - containerPort: 3000


---

apiVersion: v1
kind: Service
metadata:
  name: nodejs-hello-world-service
  namespace: testing
spec:
  selector:
    app: nodejs-hello-world-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: LoadBalancer



```



















