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
