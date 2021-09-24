# jit-env-webpack-plugin

[![MIT](https://img.shields.io/github/license/weavedev/jit-env-webpack-plugin.svg)](https://github.com/weavedev/jit-env-webpack-plugin/blob/master/LICENSE)
[![NPM](https://img.shields.io/npm/v/@weavedev/jit-env-webpack-plugin.svg)](https://www.npmjs.com/package/@weavedev/jit-env-webpack-plugin)

Injects .env.json files into the web application for development and adds an injection point for just-in-time injection when used in production.

## Install

```
npm i @weavedev/jit-env-webpack-plugin
```

## Why‽

The reason we created this plugin is that we want our staging containers to be used in production without having to rebuild them. This plugin adds a bit of code that allows you to inject a JSON env into your project with minimal effort when used in production. It also allows you to inject your local env files while developing locally.

## Usage

### `webpack.config.js`

```js
const HtmlWebpackPlugin = require('html-webpack-plugin');
const JitEnvWebpackPlugin = require('@weavedev/jit-env-webpack-plugin');

module.exports = {
  plugins: [
    new HtmlWebpackPlugin(),
    new JitEnvWebpackPlugin(),
  ],
};
```

### Options

JitEnvWebpackPlugin provides the following options.

```js
new JitEnvWebpackPlugin({
    /**
     * The default env file to use. This is usefull if you want to provide an
     * example structure or provide a default config to a testing environment.
     * This config is also used when generating type definitions with the
     * `emitTypes` option.
     */
    defaultEnv: "./default.env.json",

    /**
     * The path to a local env file to use. If the file can not be found the
     * `defaultEnv` file is used and a warning is shown in the browser's
     * console.
     *
     * Add this path to your .gitignore to prevent developers from adding their
     * local env file to the repository.
     */
    userEnv: "./user.env.json",

    /**
     * Generate a simple TypeScript types file from the `defaultEnv` file. You
     * may need to import this file somewhere (depending on your TypeScript
     * configuration) for TypeScript to find the types.
     *
     * This file will also export an env object to use if you don't want to use
     * `window.env` in your project.
     */
    emitTypes: "./src/myEnv.ts",

    /**
     * If the emitted TypeScript types file causes linting issues you can
     * provide a prefix string that will be added to the `emitTypes` file
     * before it is emitted.
     *
     * You probably don't need this because most linters allow you to exclude
     * files in the linter's configuration, but it is here if you need it.
     */
    emitTypesPrefix: "/* tslint:disable */",
});
```

###### NOTE

These options should really only be used while developing your application. See the full config example below for more details on using JitEnvWebpackPlugin in production.

### Usage in code

If we have an env file...

```json
{
    "baseUrl": "http://localhost:8001",
    "devMode": true
}
```

...we can use the variables in our code like this:

```js
if (window.env.devMode) {
    console.log(`Using dev mode with API: ${window.env.baseUrl}`);
}
```

...and in TypeScript like this:

```ts
// Configure this path in the `emitTypes` option
import { env } from './myEnv.ts';

if (env.devMode) {
    console.log(`Using dev mode with API: ${env.baseUrl}`);
}
```

### Emit TypeScript types

You can configure a target path to emit a TypeScript types file that will be generated from the default env file.

By configuring JitEnvWebpackPlugin like this:

```js
new JitEnvWebpackPlugin({
    defaultEnv: "./default.env.json",
    emitTypes: "./src/myEnv.ts",
});
```

If your default env file looks like this...

```json
{
    "baseUrl": "http://localhost:8001",
    "devMode": true,
    "retry": 3,
    "servers": {
        "cdn": "http://localhost:8002",
        "s3": "http://localhost:9000"
    },
    "users": [
        {
            "name": "Will",
            "id": 1
        },
        {
            "name": "Matt",
            "id": 2
        }
    ]
}
```

...JitEnvWebpackPlugin will generate a TypeScript types file that looks like this:

```ts
// This file was generated by JitEnvWebpackPlugin.
//
// If this file causes linting issues, you can pass a linting disable string
// with the emitTypesPrefix option.

if (window.env === undefined) {
  throw new Error("[JIT-ENV] Missing env");
}

export const env: Window['env'] = window.env;

declare global {
  interface Window {
    env: {
      "baseUrl"?: string;
      "devMode"?: boolean;
      "retry"?: number;
      "servers"?: {
        "cdn"?: string;
        "s3"?: string;
      };
      "users"?: {
        "name"?: string;
        "id"?: number;
      }[];
    };
  }
}

```

### Full `webpack.config.js` example

```js
const HtmlWebpackPlugin = require('html-webpack-plugin');
const JitEnvWebpackPlugin = require('@weavedev/jit-env-webpack-plugin');

/**
 * When building for production you use:
 * 
 * webpack --env.production
 */
const PROD = env.production;

const jitEnvWebpackPlugin = PROD ? new JitEnvWebpackPlugin() : new JitEnvWebpackPlugin({
    defaultEnv: "./default.env.json",
    userEnv: "./user.env.json",
    emitTypes: "./src/myEnv.ts",
});

module.exports = {
  plugins: [
    new HtmlWebpackPlugin(),
    jitEnvWebpackPlugin,
  ],
};
```

### Replacing in CI/CD or in containers

```sh
# Load path to your env file
export REPLACE_CONFIG=/path/to/config.env.json

# Load env file contents
CONFIG=$(cat $REPLACE_CONFIG | tr -d '\n' | tr -d '\r' | sed 's/"/\\"/g' | sed "s/\//\\\\\//g");

# Inject env file contents into your index.html
sed -i "s/\"___INJECT_ENV___\"/$CONFIG/g" /path/to/index.html;
```

We use a Dockerfile that mounts a JSON file and uses a script with the above as entrypoint to handle this step.

## License

[MIT](https://github.com/weavedev/store/blob/master/LICENSE)

Made by [Paul Gerarts](https://github.com/gerarts) and [Weave](https://weave.nl)
