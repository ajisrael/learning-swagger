# Getting Started

## Source Project

This project is sourced from [rsbh.dev](https://rsbh.dev/blogs/rest-api-with-express-typescript)

## Project setup

Create and initialize project:

```bash
mkdir express-typescript
cd express-typescript
npm init -y
```

Install TS as dev dependency

```bash
npm i -D typescript
```

Add `tsconfig.json`, details on configurations [here](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)

```json
{
  "compilerOptions": {
    "target": "es6",
    "module": "commonjs",
    "outDir": "./build",
    "strict": true,
    "esModuleInterop": true
  }
}
```

Install express and type definitions as regular and dev dependencies respectively

```bash
npm i -S express
npm i -D @types/express @types/node
```

## Create API

Add your code for the api in something like `src/index.ts`

```typescript
import express, { Application } from 'express';

const PORT = process.env.PORT || 8000;

const app: Application = express();

app.get('/ping', async (_req, res) => {
  res.send({
    message: 'pong',
  });
});

app.listen(PORT, () => {
  console.log('Server is running on port', PORT);
});
```

Add build script to `package.json`

```json
"scripts": {
  "build": "tsc",
}
```

Build JS code with following command:

```bash
npm run build

```

Run JS code:

```bash
node ./build/index.js
```

## Configure development environment

Install some more QoL dev dependencies:

```bash
npm i -D ts-node nodemon
```

Add dev script and configuration for nodemon to `package.json`:

```json
"scripts": {
    "build": "tsc",
    "dev": "nodemon",
  },

  "nodemonConfig": {
    "watch": [
      "src"
    ],
    "ext": "ts",
    "exec": "ts-node src/index.ts"
  }
```

Run dev script:

```bash
npm run dev
```

## Add middleware

Install `morgan` for logging and its types:

```bash
npm i -S morgan
npm i -D @types/morgan
```

## Swagger integration

Download dependencies:

```bash
npm i -S tsoa swagger-ui-express
npm i -D @types/swagger-ui-express concurrently
```

Add support for decorators in `tsconfig.json`

```json
{
  "compilerOptions": {
    ...
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

Create config file for tsoa: `tsoa.json`

```json
{
  "entryFile": "src/index.ts",
  "noImplicitAdditionalProperties": "throw-on-extras",
  "spec": {
    "outputDirectory": "public",
    "specVersion": 3
  }
}
```

Update scripts:

```json
"scripts": {
    "start": "node build/index.js",
    "predev": "npm run swagger",
    "prebuild": "npm run swagger",
    "build": "tsc",
    "dev": "concurrently \"nodemon\" \"nodemon -x tsoa spec\"",
    "swagger": "tsoa spec",
  },
```

Build and run server:

```bash
npm run build
npm run dev
```

# Adding Docker

Following tutorial from [rsbh](https://rsbh.dev/blogs/rest-api-express-typescript-docker)

## Create Docker File

To Dockerize the server, we need to create a Dockerfile. A Dockerfile is just a list of instructions to create a docker image. Read more about Dockerfile [here](https://docs.docker.com/engine/reference/builder/)

```dockerfile
FROM node:12

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 8000

CMD ["npm", "run", "dev"]
```

Add `.dockerignore`:

```
node_modules
npm-debug.log
```

## Create Docker Image

After creating the Dockerfile, we need to run the docker build to create a docker image from the Dockerfile. Here we are naming the docker image as express-ts.

```bash
docker build -t express-ts .
```

Verify image created

```bash
docker images
```

Run docker image

```bash
docker run -p 8000:8000 express-ts
```

## Add Docker Compose

We need to mount the local src folder to the docker container folder, so every time we make any change inside the src folder, nodemon restarts the development server inside the docker container.

We will add the docker-compose.yml file to the root of the project to mount the local src folder. Read more about docker-compose [here](https://docs.docker.com/compose/)

```yml
version: '3'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - ./src:/app/src
    ports:
      - '8000:8000'
```

Run docker compose

```bash
docker-compose up
```

## Create Docker Files for each ENV

Change the existing `dockerfile` to `dockerfile.dev` and update `docker-compose.yml`, then create a new `dockerfile` for production env.

Build new `dockerfile` with following command:

```bash
docker build -t express-ts/alpine .
```

Now we need to optimize our production image.

Here we create two stages, one for building the server and the other for running the server. In the builder stage, we generate Javascript code from the Typescript files. Then in the server stage, we copy the generated files from the builder stage to the server stage. In the Server stage, we need only production dependencies, that's why we will pass the --production flag to the npm install command.

```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM node:18-alpine AS server
WORKDIR /app
COPY package* ./
RUN npm install --production
COPY --from=builder ./app/public ./public
COPY --from=builder ./app/build ./build
EXPOSE 8000
CMD ["npm", "start"]
```

Build new multi-staged (ms) docker image

```bash
docker build -t express-ts/ms .
```

Compare image sizes:

```bash
docker images
```

# Add PostgreSQL to API

Following tutorial by [rsbh](https://rsbh.dev/blogs/rest-api-express-postgres-typeorm)

## Install PostgreSQL and TypeORM

ORM (Object Relational Mapping) will be used to map objects and their relationships for getting information from our relational database.

Install command for typeorm and postgresql driver:

```bash
npm install typeorm reflect-metadata --save
npm install pg --save
```

## Setup Postgres database

Update `docker-compose.yml` to have a database service and mark that as a dependency of the app.

```yml
version: '3'

services:
  db:
    image: postgres:12
    environment:
      - POSTGRES_DB=express-ts
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - ./src:/app/src
    ports:
      - '8000:8000'
    depends_on:
      - db
    environment:
      - POSTGRES_DB=express-ts
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_HOST=db
```

Build and run container:

```bash
docker-compose build
docker-compose up
```

## Verify Database schema

Once we add the models to the server and start it, the tables will be created in the database. We can verify it by executing the bash command to the running docker container. In the docker container, we have to run the psql cli.

```bash
docker exec -it express-typescript_db_1 bash
```

```bash
psql -U postgres
```
