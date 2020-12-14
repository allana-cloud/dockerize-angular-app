# dockerize-angular-app
Dockerize your angular-app without installing node and npm on your hosts
Steps on how to create your dockerize angular app without installing node or npm in your hosts


## Steps

1. Start by creating a new Dockerfile, and have have the following inside. We are going to use the latest node version
and install the angular/cli package    

```
FROM node:latest
RUN npm install -g @angular/cli
WORKDIR /app
```


2. Build the Dockerfile and name the image as angular-image, or you can choose to change the image name

```
docker build -t [container-image-name] .

docker build -t angular-image .
```


3. Create your angular container and map its volume to pwd(present working directory). Again you can also change the container name to your liking.

```
docker run -dit --name [container-name] -v ${PWD}:/app [container-image-name]

docker run -dit --name angular-container -v ${PWD}:/app angular-image
```    


4. Now that we have an angular container. Let us create your angular project by using docker exec command:

```
docker exec -it [container-name] ng new [projectName] --directory=.
    
docker exec -it angular-container ng new testApp --directory=.
```

You may encounter fatal error with Git identity configuration, this is okay you can ignore this for now.

```
** Please tell me who you are.

Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

to set your account's default identity.
Omit --global to set the identity only in this repository.

fatal: unable to auto-detect email address (got 'root@c4ee742f5149.(none)')

```


5. We now have a new Angular project created. And since the ng project was created via docker we need to change ownership to be able to edit 
the source files. Due to the fact that all containers default user is root.

```
    sudo chown -R $USER:$(id -gn $USER) ./*
    
    ## To include hidden files
    
    sudo chown -R $USER:$(id -gn $USER) .*
```


6. Let us now, remove the initial angular container and proceed to the final stretch of our configuration.

```
    docker rm -f angular-container
```


7. Update the Dockerfile with the following, now that we have the initial project files. We update the Dockerfile.


```
FROM node:latest

WORKDIR /app

COPY package.json ./

RUN npm install -g @angular/cli
RUN npm install

EXPOSE 4200

CMD ["ng", "serve", "--host", "0.0.0.0"]
```


8. Now let us create our docker-compose.yml file. And run it via docker-compose up

```
version: "3"
services:
    web:
        build: .
        ports:
            - "4200:4200"
        volumes:
            - "/app/node_modules"
            - ".:/app"
```

If you have multiple angular-app project, you must indicate other port aside from "4200". Example below shows that the hosts maps to 4201:4200 this will expose the this new
angular-app to this port instead of the default port it if's being used already.

```
version: "3"
services:
    web:
        build: .
        ports:
            - "4201:4200"
        volumes:
            - "/app/node_modules"
            - ".:/app"
```


Let us run this docker-compose via:

```
docker-compose up
```


9. Again change owenership, since the above update Dockerfile did an npm install, to ensure we have proper permissions to our source files

```
    sudo chown -R $USER:$(id -gn $USER) ./*
    
    ## To include hidden files
    
    sudo chown -R $USER:$(id -gn $USER) .*
```



## Adding new resource/components

1. To create additional resouce use "docker-compose exec web" and then the angular command to create/add your needs. Example to add a component xyz,
we do the following:

```
    docker-compose exec web ng generate component xyz
```
    
    The identifier "web" is from our docker-compose file, so if you have change to something else use that in reference for your doceker-compose exec.
Component xyz will be generate by angular, however you still need to change the owneship.

```    
    sudo chown -R $USER:$(id -gn $USER) ./*
```


