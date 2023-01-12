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
