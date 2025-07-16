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



<!-- STEP 2 -->
<!-- 
# Now the next thing is setting up CI pipeline 
# first we have to create this inside .github/workflows/ as we are going to use github actions.
# After that let's come to the syntax why are we writing what we are writing.
name: CI Pipeline(name of our workflow it appears in the github actions workflow too.)

on:(The name is itself valid it means that execute this when we are pushing to main branch or pulling from there.)
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:(jobs are something like set of steps in a workflow that is executed on a same runner. Jobs run in parallel by default.)
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4(we can find these from github marketplace. Checkout is used to extract code from repo and pasting it on runner.)

      - name: Set up Node.js
        uses: actions/setup-node@v4(once the code is extracted using checkout then setup node js)
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Run basic lint or test (optional)
        run: echo "No tests configured, skipping for now"

      - name: Build Docker image
        run: docker build -t nodejs-sample-app .

 -->