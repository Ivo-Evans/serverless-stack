---
id: installation
title: Installation
description: "Creating a new Serverless Stack Toolkit (SST) app"
---

import config from "../config";

SST is a collection of <a href={ `${ config.github }/tree/master/packages` }>npm packages</a>.

## Requirements

- [Node.js](https://nodejs.org/en/download/) >= 10.15.1
- An [AWS account](https://serverless-stack.com/chapters/create-an-aws-account.html) with the [AWS CLI configured locally](https://serverless-stack.com/chapters/configure-the-aws-cli.html)

## Getting started

Create a new project using.

```bash
npx create-serverless-stack@latest my-sst-app
```

Or alternatively, with a newer version of npm or Yarn.

```bash
# With npm 6+
npm init serverless-stack@latest my-sst-app
# Or with Yarn 0.25+
yarn create serverless-stack my-sst-app
```

This by default creates a JavaScript/ES project. If you instead want to use **TypeScript**.

```bash
npm init serverless-stack@latest my-sst-app --language typescript
```

By default your project is using npm as the package manager, if you'd like to use **Yarn**.

```bash
npm init serverless-stack@latest my-sst-app --use-yarn
```

You can read more about the [**create-serverless-stack** CLI here](packages/create-serverless-stack.md).

## Project layout

Your app starts out with the following structure.

```
my-sst-app
├── README.md
├── node_modules
├── .gitignore
├── package.json
├── sst.json
├── test
│   └── MyStack.test.js
├── lib
|   ├── MyStack.js
|   └── index.js
└── src
    └── lambda.js
```

An SST app is made up of a couple of parts.

- `lib/` — App Infrastructure

  The code that describes the infrastructure of your serverless app is placed in the `lib/` directory of your project. SST uses [AWS CDK](https://aws.amazon.com/cdk/), to create the infrastructure.

- `src/` — App Code

  The code that’s run when your app is invoked is placed in the `src/` directory of your project. These are your Lambda functions.

- `test/` — Unit tests

  There's also a `test/` directory where you can add your tests. SST uses [Jest](https://jestjs.io/) internally to run your tests.

You can change this structure around to fit your workflow. This is just a good way to get started.

### Infrastructure

The `lib/index.js` file is the entry point for defining the infrastructure of your app. It has a default export function to add your stacks.

```jsx title="lib/index.js"
import MyStack from "./MyStack";

export default function main(app) {
  new MyStack(app, "my-stack");

  // Add more stacks
}
```

You'll notice that we are using `import` and `export`. This is because SST automatically transpiles your ES (and TypeScript) code using [esbuild](https://esbuild.github.io/).

In the sample `lib/MyStack.js` you can add the resources to your stack.

```jsx title="lib/MyStack.js"
import * as sst from "@serverless-stack/resources";

export default class MyStack extends sst.Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // Define your stack
  }
}
```

Note that the stacks in SST use [`sst.Stack`](constructs/stack.md) as opposed to `cdk.Stack`. This allows us to deploy the same stack to multiple environments.

In the sample app we are using [a higher-level API construct](constructs/api.md) to define a simple API endpoint.

```js
const api = new sst.Api(this, "Api", {
  routes: {
    "GET /": "src/lambda.handler",
  },
});
```

### Functions

The above API endpoint invokes the `handler` function in `src/lambda.js`.

```js title="src/lambda.js"
export async function handler() {
  return {
    statusCode: 200,
    body: "Hello World!",
    headers: { "Content-Type": "text/plain" },
  };
}
```

Notice that we are using `export` here as well. SST also transpiles your function code.

## Project config

Your SST app also includes a config file in `sst.json`.

```json title="sst.json"
{
  "name": "my-sst-app",
  "stage": "dev",
  "region": "us-east-1"
}
```

The **stage** and the **region** are defaults for your app and can be overridden using the `--stage` and `--region` options. The **name** is used while prefixing your stack and resource names.

You'll be able to access the stage, region, and name of your app in `lib/index.js`.

```js
app.stage; // "dev"
app.region; // "us-east-1"
app.name; // "my-sst-app"
```

You can also access them in your stacks, `lib/MyStack.js`.

```js
this.node.root.stage; // "dev"
this.node.root.region; // "us-east-1"
this.node.root.name; // "my-sst-app"
```

You can read more about [the additional set of constructs that SST provides here](packages/resources.md).