<!-- This is Nothing but my way to document whatever i am doing. -->
<!-- Step 1 -->
<!-- 
# Created a Dockerfile.
# Gonna use multi stage in this as building an app in one container and then run in another cleaner smaller container.
# Build Stage -> Compile, install dependencies, bundle code.
# Production Stage -> Running the app in clean env. 
# How to write a dockerFile 
FROM node:20-alpine AS builder (image that we have to use builder is the name of the stage.)
WORKDIR /app(it's like saying cd into this and from now on whatever is written in dockerfile happens inside this. This directory keeps things in one place inside a container.)

COPY package*.json ./(copying the dependency files to install it later.)
RUN npm ci(clean installation)

COPY . .(copy everything from your local working directory to the container working dire)

#Now the docker file is created , we are shifting our project to docker -> (docker build -t nodejs-sample-app .) -> (docker run -p 5000:5000 nodejs-sample-app)
-->