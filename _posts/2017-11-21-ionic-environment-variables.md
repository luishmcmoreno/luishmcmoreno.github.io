---
title: "Ionic 2+ Environment Variables"
categories:
  - Continuous Integration
tags:
  - continuous-integration
---

Recently, I and my team were facing a problem here at Pluritech with Ionic 2+ projects. We wanted to implement environment variables like the @angular/cli projects do. Ionic projects doesn't handle it.

By searching the web, we founded some issues at Github that the answers made it possible to do and I'll share our solution.

First, we need to create a folder ``src/environments`` to handle the different environments there, like ``src/environments/environment.ts`` (prod environment) and ``src/environments/environment.dev.ts``. We can create as much as possible environments we want, like ``src/environments/environment.staging.ts``.

Now, we need to handle this file with node environments. To do it, we need a trick. We'll use a typescript alias to handle which file will be used.

To create this alias we need to include, at ``tsconfig.json``, inside ``compilerOptions`` the following:

```json
    "baseUrl": "./src",
    "paths": {
      "@app/env": [
        "environments/environment"
      ]
```

This way, when typescript compiler found ``@app/env``, it will be swapped by ``environments/environment``, then when we need to include the environment in a file, we need to import this way:

```typescript
  import { env } from '@app/env';
```

But now, we need to change this value accordingly with node envs. To do it we'll use webpack. First, create a file ``config/webpack.config.js`` and adds to our ``package.json`` the following instruction:

```json
    "config": {
        "ionic_webpack": "./config/webpack.config.js"
    }
```

Now the Ionic will know to use this new webpack configuration.

To finish it, we need to type the following:

```js
var chalk = require('chalk');
var fs = require('fs');
var path = require('path');
var useDefaultConfig = require('@ionic/app-scripts/config/webpack.config.js');

var env = process.env.PLURI_ENV;

useDefaultConfig.prod.resolve.alias = {
  '@app/env': path.resolve(environmentPath())
};

useDefaultConfig.dev.resolve.alias = {
  '@app/env': path.resolve(environmentPath())
};

function environmentPath() {
  var filePath;
  if (!env) {
    filePath = './src/environments/environment.dev.ts';
  } else {
    filePath = './src/environments/environment' + (env === 'prod' ? '' : '.' + env) + '.ts';
  }
  if (!fs.existsSync(filePath)) {
    console.log(chalk.red('\n' + filePath + ' does not exist!'));
  } else {
    return filePath;
  }
}

module.exports = function () {
  return useDefaultConfig;
};

```

Now, this webpack configuration will change the ``@app/env`` value to the corresponding file. To do it, we just need to include a node env to the CLI command.

```bash

  PLURI_ENV=prod ionic serve # to serve our project with prod env
  PLURI_ENV=prod ionic cordova build android --prod # to build our android project with prod env
  ionic serve # to serve our project with dev env

```

<strong>Note:</strong> When running with a custom variable, production builds still need --prod flag.


If you have any questions, feel free to comment here.