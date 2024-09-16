# Docker Node Typescript Starter Project
<p align="center" valign="middle">
<img src="https://www.vectorlogo.zone/logos/docker/docker-official.svg" alt="Docker logo" width="200"/>
<img src="http://www.techjini.com/wp-content/uploads/2017/01/nodejs-logo.png" alt="Nodejs logo" width="200"/>
<img src="http://mherman.org/assets/img/blog/typescript-logo.png" alt="Typescript logo" width="240"/>
</p>



## Overview
Want to build a service (like an API) or web app fast using [Node](https://nodejs.org) and [Typescript](https://typescriptlang.org).  This repo sets up your filestructure and project using [containers](https://www.cio.com/article/2924995/software/what-are-containers-and-why-do-you-need-them.html), or, more specifically [Docker](https://www.docker.com/) containers.  This setup, and your app, is then easily transferable to any other computer and you can easily replicate an exact build environment.  Sound cool?  It really is.



## The result
This app builds a simple web app chat client using `socket.io`.  This is just an example to show you how easy it is to get started.  Everything is built from that single `docker-compose up` command.  You're intended to change the app to your own liking and build literally anything you'd like.  `index.ts` is assumed to be your app's starting point.


#### File Structure
```
docker-node-typescript
|   docker-compose.yml
|   Dockerfile
|   gulpfile.js
|   package.json
|   README.md
|   tsconfig.json
│
└───app // This is where you put your app files.  These will be automatically compiled
│   │   index.ts
│   |   index.html
│
└───dist // This is where your compiled files gets copied to at build-time
```

## Features
#### No Dependencies Necessary
This requires no dependencies on the host system except Docker and [Docker Compose](https://docs.docker.com/compose/).
When developing, if you want to avoid Typescript yelling at you, it will (as always) be necessary to download the required type definition files.  This is easy.  Using `npm i @types/node @types/express --save-dev` you can download the types you need for Node and Express.  You may do the same for any other type definitions.  You should always use `--save-dev` when downloading types.

#### Not just Typescript
Some other Docker builds are only capable of compiling Typescript because they just use `tsc` without `gulp`.  This is great, in most cases.  But this seed goes one step further by using `gulp` to move your `html` files into the `dist` folder as well.  I know gulpfiles used to have a tendency to scare the crap out of me, so I kept it simple. You can open the **tiny, simple** gulpfile and change the arguments to copy any files you'd like, you don't have to limit yourself to just `html` and `ts`.

#### Quicker builds
This project uses [`tsc`](https://www.npmjs.com/package/tsc) to build the Typescript.  `tsc` offers incremental builds and is faster than `gulp-typescript` at compiling your code.  As a side-note, we're also using [`gulp.watch`](https://github.com/gulpjs/gulp/blob/master/docs/API.md#gulpwatchglobs-opts-fn) for non-ts files and [`tsc-watch`](https://www.npmjs.com/package/tsc-watch) for `ts` files.  `tsc-watch` allows us to re-load the app (like nodemon would) once compilation is completed.  Since `nodemon` requires a `legacy-watch` variable (`-L`) on many windows machines, I opted to go an arguably better route by using `gulp.watch` and `tsc-watch`.

#### Easily deployable
If you're familiar with [Heroku](https://heroku.com), and you have the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) installed, you can run two commands to get this app deployed.  `heroku create` followed by `heroku container:push web`.  And... Wa-la! `heroku open` will open your browser to your brand new app.

## The `Dockerfile` Breakdown
```
FROM node:8

ENV USER=app

ENV SUBDIR=appDir

RUN useradd --user-group --create-home --shell /bin/false $USER &&\
	npm install --global tsc-watch npm ntypescript typescript gulp-cli

ENV HOME=/home/$USER

COPY package.json gulpfile.js $HOME/$SUBDIR/

RUN chown -R $USER:$USER $HOME/*

USER $USER

WORKDIR $HOME/$SUBDIR

RUN npm install

CMD ["node", "dist/index.js"]
```

---

#### Command:
```
FROM node:8
```
##### What:
Build this image from the [`node:8`](https://hub.docker.com/_/node/) image that is maintained by docker on the [Docker Hub](https://hub.docker.com).
    Docker is auto-magically creating an instance of Linux with Node installed (that's what the `node:8` means), and this whole instance will still be on your machine.  If you're familiar with Virtual Machines, this might sound familiar to you.  What is actually happening is a little more complex, but that's a topic for another time.  As discussed, this platform will be based in Linux.  Every command that follows is essentially a command that will be run from the terminal of this newly created Linux instance.  So every command will either be a Linux command or a special Docker command.
##### Why:
This gives Docker the context it needs to get us up and running.  A simple, stable version of Linux with Node installed is a great place to start building your Node and Typescript app!

---

#### Command:
```
ENV USER=app
```
##### What:
Set the environment variable of USER to the string "app".  We can later reference this by using `$USER` in our Dockerfile.
##### Why:
This will become clearer later on. This is just syntactic sugar.  It prevents us from needing to write "app" multiple times in our Dockerfile.  "app" will be the name of our non-root user in Linux that ends up running future commands from this `Dockerfile`.  We could theoretically name this user anything.

---

#### Command:
```
ENV SUBDIR=appDir
```
##### What:
Set the environment variable of SUBDIR to the string "appDir".  We can later reference this by using `$SUBDIR` in our Dockerfile.
##### Why:
This will become clearer later on. This is just syntactic sugar.  It prevents us from needing to write "appDir" multiple times in our Dockerfile.  SUBDIR could theoretically be named anything.  It's just the name of a subdirectory in which we'll put our files.

---

##### Command:
```
RUN useradd --user-group --create-home --shell /bin/false app &&\
   ```
##### What:
This command is part of a two-line command. Let's explain this first line first.  This command runs `useradd`, which is a linux command to create a user.  What follows are a few options, which I'll explain.
- The `--user-group` command assigns this user it's own user group.  A user group is just a way of grouping multiple users together.  User groups are often used to perform various actions -- like permissions changes -- for multiple users at the same time.  In this case, we're just grouping one user.  Even though we're not grouping multiple users together, it's good to assign a user to a user-group instead of it automatically being assigned the default user-group.  Calling `--user-group` will assign this user a usergroup which is identical to its name.  That's fine.

- The `--create-home` command dedicates a home directory by default for this soon-to-be-created user.  In node, by default, that home directory lives at the path `/home/user_name_here`.  This means that `home/$USER` will be a real directory created after this command finishes.

- The `--shell /bin/false` command sets the `shell` of this user to an existing command defined by `/bin/false`.  That's a real command which actually does nothing but exit with an error code.  The `shell` is the terminal you see after you SSH or remote login to an FTP server.  We set the user to have essentially an error as their shell to prevent anyone from logging in as this user.  We do this because no one will ever need to log-in as this user, so this is a security precaution, and is perfectly fine.

- The final argument: `$USER` is simply finishing the command and telling the system to name this new user whatever `$USER` equates to.  Remember, `$USER` is our environment variable we specified up top.  It can be anything we want.

 - Lastly, the `&&\` command allows us to chain two commands together.  The `&&` means "do the command before me, and then do the command after me".  The `\` is just an escape character, letting us write a command that spans two lines.  We need it because we have that line-break in there. This line break is just to keep things easily readable.  If we don't use this escape character, we'll need to remove that line-break, or we'll get an error.

##### Why:
Creating a new user is a recommended practice when creating production-ready Docker apps in Node.  Running Node as the root user poses a security concern, and may pose other issues as well.  Therefore this line alleviates that concern.

---

#### Command:
```
    npm install --global npm ntypescript typescript gulp-cli
   ```
##### What:
We run `npm install --global` on a number of global packages.  This `npm install` is running as the default `root` user because we haven't yet switched users using the `USER` command (which we will do shortly).  We are using the `--global` tag to identify that the following packages will be installed to the root user's global dependencies. If we tried to run `npm install -g some_package_name` as a non-root user we would run into issues.  The non-root user does not have permissions to install global packages.

##### Why:
We need to install these global dependencies here because they are crucial for building or running our app inside the container.  For example, in order to build our app, the container needs to be able to run `gulp` commands natively, which requires that `gulp-cli` be a global package on the container itself.  To help you better understand this, you can try to run something like `gulp` from the command line of your machine.  If you don't have the `gulp-cli` installed globally, you'll get an error. Once `gulp-cli` is installed globally, the `gulp` command should be available inside any terminal window. Therefore, since this Docker container is essentially an entire new machine, we need to install gulp on it, along with some other packages.  This line accomplishes that task.

---

#### Command:
```
ENV HOME=/home/$USER
```
##### What:
Set the environment variable of HOME to the string "/home/$USER" where `$USER` is substituted by Docker for the `USER` environment variable above.  We can later reference this by using `$HOME` in our Dockerfile.  Remember that the home directory is created by default from Node as `/home/some_user_name` when we created the previous user.
##### Why:
This is syntactic sugar.  It prevents us from needing to write "/home/$USER" multiple times in our Dockerfile.  We eventually switch into this home directory because that's where all our files will be stored on the Container.  Using this home directory structure is a best-practice.

---

#### Command:
```
COPY package.json tsconfig.json gulpfile.js $HOME/$SUBDIR/
```
##### What:
Copy the `package.json` file, `tsconfig.json` file, and the `gulpfile.js` file on our host machine into the Docker container at `$HOME/$SUBDIR/`
##### Why:
Our Docker container needs to have access to these files, and, lo and behold, they come from our machine.  The `package.json` is a critically important file because the subsequent `npm install` command (which we'll get to) will read this `package.json` file in order to know what to install on the Docker container.  The `gulpfile.js` is also critically important because we will be running `gulp` and utilizing the presets we've defined in that file.

Note - Because the Container itself will run `npm install` against this `package.json` file, you don't actually need to keep dependencies in your project on your host machine, you only need to keep the `package.json` file up to date.  Dependencies will be downloaded **inside the container**.  You will, however, need to install type declarations like `@types/node` on your host machine if you're writing in Typescript.  This is because Typescript will still throw warnings/errors locally if it doesn't know where to look for type definitions.

As many other blogs note, this `COPY` command comes before the `npm install` so that we can cache the results of `npm install` and only re-run that command when any of the above files has changed.

---

#### Command:
```
RUN chown -R $USER:$USER $HOME/*
```
##### What:
The user/usergroup defined by the statement `$USER(user name):$USER(user group name)` will be given ownership permissions of the directory at `$HOME/` and everything else beneath that directory(`*`).
##### Why:
When we `COPY`'d files from our previous command, we were doing so under the `root` user permissions.  We needed to be the `root` user to do that without permissions errors.  However, in doing do, those files we copied are owned by the `root` user.  We want our **non-root user** to own these files, so that when we eventually switch over to using that non-root user, we have full permissions of these files.

---

#### Command:
```
USER $USER
```
##### What:
Change our working user within the Docker container to be the username defined by `$USER`.  Every subsequent command (unless we change the user again) will be run as this user.

##### Why:
We want to perform the following actions as our non-root user.  This will allow us to do so.

---

#### Command:
```
WORKDIR $HOME/$SUBDIR
```
##### What:
Change directory in the docker container to the directory defined by `$HOME/$SUBDIR`

##### Why:
We're about to perform actions that need us to be in this directory.

---

#### Command:
```
RUN npm install
```
##### What:
Install the packages defined by our `package.json` file.  This is being run inside the Docker container by our `$USER` user.
##### Why:
We need to build our NPM dependencies. We run this command **after** we copy in our `package.json` because of how caching works in Docker.  Every line in a Dockerfile is cached until Docker recognizes it has been changed.  Meaning that when you re-build a Dockerfile, if some line has not changed, then that line will run from the cache, and take very little time to complete. By putting the `package.json` line above this `npm install` line, we are caching the `npm install` command until the `package.json` line (or file) gets changed.  If the `package.json` file is changed, Docker will recognize this on it's next build and it will re-run this `npm install` command anew.

---

#### Command:
```
CMD ["node", "dist/index.js"]
```
##### What:
Run `node dist/index.js` from the command-line programmatically.  This runs our `dist/index.js` file. `dist/index.js` is the compiled version of the Typescript `/index.ts` file, and this file is the entry point of our app.
##### Why:
As discussed, this kicks off our app.  You might be inclined to use something like `RUN npm run some_package-json_script_here` but doing so has some quirks and it's best to use a `CMD` command.

---


## The `docker-compose.yml` Breakdown
```
version: '3.1'

services:
  app:
    build: .
    command: npm run build
    environment:
      NODE_ENV: development
    ports:
      - '3000:3000'
    volumes:
      - .:/home/app/appDir
      - /home/app/appDir/node_modules
```

#### Command:
```
version: '3.1'
```
##### What:
Use `docker-compose` format version `3.1` for the duration of this file.
##### Why:
Why not?  As of writing this, version `3.1` is the latest version.  I don't see any reason why you shouldn't use the latest version.

---

#### Command:
```
services
```
##### What:
Tell docker that everything within this block will be `services` of Docker.  Services are documented [here](https://docs.docker.com/get-started/part3/).  They are separate pieces of your presumably distributed application.  They are individual "components" of your app.  They are "containers in production".  A service block describes the way in which that piece of your app should be built and run.  Sound confusing?  It's not.  Check the [documentation](https://docs.docker.com/get-started/part3/).
##### Why:
Every Docker Compose file needs a service.  We will later find that services are created by the Dockerfile and that the `build` property of each `service` block tells `docker-compose` from where to load that `Dockerfile`

---

#### Command:
```
app:
```
##### What:
Name this service `app`.  Why "app"?  Well, We can name this service anything we want.  Feel free to change it.
##### Why:
Every service needs a name.  Sometimes in a service we can use this name to reference the service elsewhere in our `docker-compose.yml` file.

---

#### Command
```
build: .
```
##### What:
Build this app from the current directory. The `.` means "this directory".  The `docker-compose.yml` file will then look here for a `Dockerfile` when building the `app` service.  It finds that Dockerfile, and runs it.
##### Why:
We need to build these services somehow.  This is one way to do so.  Not every service needs a `build` command, but we do here because we are running this from a specific Dockerfile.  Since `build` looks in the subsequent directory for a `Dockerfile`, and since we already have a great Dockerfile, we're good to go.  In a different use-case we could run this service from a pre-built image that we have hosted on a repository.  Doing that would look something like this: `build: image: username/repo:tag`.

---

#### Command:
```
command: npm run build
```
##### What:
This line says that once the `Dockerfile` is complete, we will run the command `npm run build` from the Docker container's command line.
##### Why:
The `command` command is often used for kicking things off once a Dockerfile is complete.  In this case we're running a predefined command called `build` that is defined in our `package.json` file.  This `build` command does a couple necessary steps to build the Typescript into JavaScript.  It uses `gulp` and `tsc-watch`, among a couple other things.  Nothing crazy.

---

#### Command:
```
environment:
```
##### What:
Set some environment variables on the Docker container based on the commands that follow
##### Why:
We often want environment-specific variables set which help us build our app.  Things like API keys that we want to keep out of the source-code of our app.  Or things that might change when we are running in development vs production.  All of these are great candidates for environment variables.

---

#### Command:
```
NODE_ENV: production
```
##### What:
Set the `NODE_ENV` environment variable to equal the string "production".  This means that within our Docker container, `process.env.NODE_ENV` will equal `production`.
##### Why:
Based on several **best-practices** guidelines, we want our standard `docker-compose.yml` file to be a production-based build.  Development environments will be set up through the merging of separate compose files.

---

#### Command:
```
ports:
```
##### What:
Define what ports will be exposed from this Docker container.  The ports will follow on the next line(s).
##### Why:
In order to view our web-based app, we will want to see it on our local machine's through a browser directed to `localhost`.  You may not know it, but when you view a website, you're operating by default through port `80`.  If you run a host on your own machine and go to `localhost`, that "url" is equivalent to `localhost:80`, meaning that we're connecting to the localhost server through port 80.  If we want to expose any ports on our Docker container, we need to expose those ports, too.  We can expose, for example, the port `3000` on the host machine and we can later map this to port `3000` on the host machine.  We cover this next.

---

#### Command:
```
3000:3000
```
##### What:
Link the **host** machine's port `3000` to hook into the **container's** port `3000`.  The number before the semicolon `:` is the port on the host, and the number after the semicolon is the port on the container.
##### Why:
The container is running its webserver on port `3000` for development. By exposing this port `3000` to our host port `3000`, we're able to point our machine to the address `localhost:3000` (or `192.168.99.100:3000` if you're on windows with docker toolbox) and we can view the development version of our site.

---

#### Command:
```
volumes:
```
##### What:
Bind the following volumes to the Docker container.  A volume is documented [here](https://docs.docker.com/engine/admin/volumes/).  Normally, when you build a container, the contents within that container are not persistent.  This means that if you stop that container and then re-start it, the data within it would cease to exist.  Volumes allow data to persist between sessions so that this data is not lost.
##### Why:
We want parts of our container to persist, such as the `node_modules` folder.  We also want to have a directory on our host machine persist inside the Docker containe. We cover this next.

---

#### Command:
```
- .:/home/app/appDir
```
##### What:
Mount the directory that this `docker-compose.yml` file is located (`.` means "this directory") to the `/home/app/appDir` directory of the Docker container.  The string before the period (`.`) refers to the directory on the host machine and the string after the period references the directory on the Docker container.
##### Why:
We want everything in this directory to persist on to the Docker container.  This line accomplishes that.

---

#### Command:
```
- /home/app/appDir/node_modules
```
##### What:
Persist the `/home/app/appDir/node_modules` directory across container instances within this container.  If the container gets rebuilt, still save this folder.
##### Why:
This saves time when caching and rebuilding the container.  `node_modules` will be persisted.

## Deploying to Heroku
`heroku create`

`heroku container:push web`

profit


## FAQ
- Q: What about COPYing the files after `npm install`? Most tutorials do that.

  A: You're correct, most tutorials **do** need this.  We don't need to do this because our `docker-compose.yml` creates a volume on this working directory.
    If you really still wanted to COPY files like that, you can use the following code in-between `RUN npm install` and `CMD`.
    ```
    USER root
    COPY . $HOME/$SUBDIR
    RUN chown -R $USER:$USER $HOME/**/*
    USER $USER
    ```

- Q: I can't see my local app at `localhost:3000`

  A: If your container is running and `localhost` doesn't load, you may be on Windows using Docker Toolbox.  If that's the case you should navigate to `192.168.99.100:3000` to see your app.

- Q: There's HTML in here.  We're serving the front-end HTML files from this container, too?

  A: For this *example*, yes.  A true microservices buff would know that we probably want to keep these features into separate services.  A typescript service would most likely build something like an API or background processing service, but yet, in this example I provided HTML just to get you started.

---

## Any Questions?
Let me know, I'll be happy to help you understand.

## Want to contribute?  Could something be improved?
Let me know, I'm happy to learn and make this better.

## Inpired by
 - [Lessons Learned While Building a Node App](http://jdlm.info/articles/2016/03/06/lessons-building-node-app-docker.html)

 - [Node Docker Good Defaults](https://github.com/BretFisher/node-docker-good-defaults)

 - [Docker and Node.js Best Practices](https://github.com/nodejs/docker-node/blob/master/docs/BestPractices.md)

 - The other countless tutorials I wrapped my brain around while tackling this whole Docker beast.
