# prisma-mongodb-cloudflare

# Deploy a TypeScript application to Cloudflare Workers that connects to MongoDB Atlas using Prisma

# 🚀 Javascript full-stack 🚀

https://github.com/coding-to-music/prisma-mongodb-cloudflare

https://prisma-mongodb-cloudflare.coding-to-music.workers.dev

https://github.com/prisma/prisma/issues/13434

By Prisma Documentation - Guides

https://www.prisma.io/docs/guides/deployment/deployment-guides/deploying-to-cloudflare-workers

## Environment Values

```java
# Environment variables declared in this file are automatically made available to Prisma.
# See the documentation for more detail: https://pris.ly/d/prisma-schema#using-environment-variables

# Prisma supports the native connection string format for PostgreSQL, MySQL, SQLite, SQL Server and MongoDB (Preview).
# See the documentation for all the connection string options: https://pris.ly/d/connection-strings

DATABASE_URL="prisma://aws-us-east-1.prisma-data.com/?api_key=get-from-prisma-dashboard"

MONGODB_URI="mongodb+srv://userid:password@cluster0.zadqe.mongodb.net/prisma-mongodb-cloudflare?retryWrites=true&w=majority"

# DATABASE_URL="postgresql://johndoe:randompassword@localhost:5432/mydb?schema=public"
```

## GitHub

```java
git init
git add .
git remote remove origin
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:coding-to-music/prisma-mongodb-cloudflare.git
git push -u origin main
```

https://vercel.com/guides/nextjs-prisma-postgres

# To actually create the tables in your database, you now can use the following command of the Prisma CLI:

- be sure to set DATABASE_URL to the Atlas database not the AWS proxy

```java
npx prisma db push
```

## Step 3. Install and generate Prisma Client

Before you can access your database from Next.js using Prisma, you first need to install Prisma Client in your app. You can install it via npm as follows:

```java
npm install @prisma/client
```

Install the Prisma Client package.

Because Prisma Client is tailored to your own schema, you need to update it every time your Prisma schema file is changing by running the following command:

```java
npx prisma generate
```

Regenerate your Prisma Schema.

Output

```java
Environment variables loaded from .env
Prisma schema loaded from prisma/schema.prisma

✔ Generated Prisma Client (3.14.0 | library) to ./node_modules/@prisma/client in 147ms
You can now start using Prisma Client in your code. Reference: https://pris.ly/d/client


import { PrismaClient } from '@prisma/client'
const prisma = new PrismaClient()

```

# Deploy a TypeScript application to Cloudflare Workers that connects to MongoDB Atlas using Prisma

- title: 'Deploying to Cloudflare Workers'
- metaTitle: 'Deploying to Cloudflare Workers'
- metaDescription: 'Learn how to deploy a TypeScript application to Cloudflare Workers that connects to MongoDB Atlas.'

https://www.prisma.io/docs/guides/deployment/deployment-guides/deploying-to-cloudflare-workers

https://github.com/prisma/docs/blob/771136a7af748ec1c7efd345d03d35d75f985e8e/content/300-guides/200-deployment/110-deployment-guides/450-deploying-to-cloudflare-workers.mdx

<TopBlock>

Today you'll be building and deploying a Cloudflare Worker that uses Prisma to save every request to a MongoDB database for inspection later.

This guide will cover Prisma, TypeScript, MongoDB, Prisma Data Proxy and Cloudflare Workers from the ground up. Let's get started!

</TopBlock>

<Admonition type="warning">

**Early Access Feature** <br />
Please note that the Prisma Data Proxy is an early-stage product and should not be used in production environments.
</Admonition>

## Prerequisites

- Free [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) database with an accessible URL
  - e.g. `mongodb+srv://<user>:<password>@cluster.mongodb.net`.
- Free [Cloudflare Worker](https://workers.cloudflare.com/) account.
- Free [GitHub](https://github.com/) account.
- Node.js & NPM installed.
- Git installed.

## 1. Set up your Application

Open your terminal and navigate to a location of your choice. Run the following commands to set up your application.

```terminal
mkdir prisma-mongodb-cloudflare
cd prisma-mongodb-cloudflare
npm init -y
npm install -D prisma typescript @cloudflare/wrangler webpack
```

## 2. Set up Wrangler

Wrangler is the official Cloudflare Worker CLI. You'll use it to develop and deploy Cloudflare Workers.

```terminal
npx wrangler init
```

This will create a `wrangler.toml` file with some initial configuration.

Next, you'll want to authenticate the Wrangler CLI with your Cloudflare Worker account. To do this, run the following command in your terminal:

```terminal
npx wrangler login
Allow Wrangler to open a page in your browser? [y/n] y
```

You can verify that you're logged in by running `npx wrangler whoami`.

## 3. Set up TypeScript

The Cloudflare Worker environment natively supports Javascript, Rust, C, and C++. To get TypeScript working on Cloudflare Workers, you need to compile TypeScript to JavaScript before deploying to a Worker.

To setup this up, create a `tsconfig.json` in your project root with the following content:

```json
{
  "compilerOptions": {
    "outDir": "./dist",
    "module": "commonjs",
    "target": "esnext",
    "lib": ["esnext"],
    "alwaysStrict": true,
    "strict": true,
    "preserveConstEnums": true,
    "moduleResolution": "node",
    "sourceMap": true,
    "esModuleInterop": true,
    "types": ["@cloudflare/workers-types"]
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

And then install `@cloudflare/workers-types` with npm:

```terminal
npm install -D @cloudflare/workers-types ts-loader@8.3.0
```

<Admonition type="info">

You need to pin `ts-loader` for the Wrangler CLI to work properly. See [this issue](https://github.com/cloudflare/wrangler/issues/1927) for more details.

</Admonition>

## 4. Set up Webpack

Wrangler has built-in webpack support that can be used to compile your code in development and before publishing on Cloudflare.

To configure webpack support, first create a `webpack.config.js` in the project root with the following code:

```js
const path = require("path");

module.exports = {
  entry: "./src/index.ts",
  output: {
    filename: "worker.js",
    path: path.join(__dirname, "dist"),
  },
  // Cloudflare Worker environment is similar to a webworker
  target: "webworker",
  resolve: {
    extensions: [".ts", ".tsx", ".js"],
    // Alias for resolving the Prisma Client properly
    alias: {
      "@prisma/client$": require.resolve("@prisma/client"),
    },
  },
  mode: "development",
  // Wrangler doesn't like eval which devtools use in development.
  devtool: "none",
  module: {
    rules: [
      {
        // Compile Typescript code
        test: /\.tsx?$/,
        loader: "ts-loader",
        options: {
          transpileOnly: true,
        },
      },
    ],
  },
};
```

Next, add the following line to your `wrangler.toml` file to point Wrangler to use this webpack configuration:

```diff
  name = "prisma-mongodb-cloudflare"
  type = "webpack"
+ webpack_config = "webpack.config.js"
```

## 5. Set up Prisma

Now you're ready to add Prisma to the project.

```terminal
npx prisma init
```

```java

✔ Your Prisma schema was created at prisma/schema.prisma
  You can now open it in your favorite editor.

warn Prisma would have added DATABASE_URL but it already exists in .env
warn You already have a .gitignore. Don't forget to exclude .env to not commit any secret.

Next steps:
1. Set the DATABASE_URL in the .env file to point to your existing database. If your database has no tables yet, read https://pris.ly/d/getting-started
2. Set the provider of the datasource block in schema.prisma to match your database: postgresql, mysql, sqlite, sqlserver, mongodb or cockroachdb (Preview).
3. Run prisma db pull to turn your database schema into a Prisma schema.
4. Run prisma generate to generate the Prisma Client. You can then start querying your database.

More information in our documentation:
https://pris.ly/d/getting-started
```

This will create a Prisma Schema in `prisma/schema.prisma` and an `.env` file.

Inside `prisma/schema.prisma`, add the following schema:

```prisma
generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["mongoDb", "dataProxy"]
}

datasource db {
  provider = "mongodb"
  url      = env("DATABASE_URL")
}

model Log {
  id      String @id @default(dbgenerated()) @map("_id") @db.ObjectId
  level   Level
  message String
  meta    Json
}

enum Level {
  Info
  Warn
  Error
}
```

This data model will be used to store incoming requests from your Worker.

# To actually create the tables in your database, you now can use the following command of the Prisma CLI:

```java
npx prisma db push
```

Output

```java
Environment variables loaded from .env
Prisma schema loaded from prisma/schema.prisma
Error: Get config: Schema Parsing P1012

error: Error validating datasource `db`: the URL must start with the protocol `mongo`.
  -->  schema.prisma:11
   |
10 |   provider = "mongodb"
11 |   url      = env("DATABASE_URL")
   |

Validation Error Count: 1
```

So I changed the Database URL to now point to the Atlas MongoDB rather than the AWS proxy, worked, created the schema on Atlas and thus cloud.Prisma will by synched correctly

```java
Environment variables loaded from .env
Prisma schema loaded from prisma/schema.prisma
Datasource "db"
Applying the following changes:

[+] Collection `Log`


🚀  Your database is now in sync with your schema. Done in 635ms

✔ Generated Prisma Client (3.14.0 | library) to ./node_modules/@prisma/client in 116ms
```

## 6. Create Repository and Push to GitHub

To prepare for the steps ahead, let's [create a private repository](https://github.com/new) on GitHub.

![Create a private repository](https://github.com/coding-to-music/prisma-mongodb-cloudflare/blob/main/images/450-01-create-repo.png?raw=true)

Next, initialize your repository, then push your changes up to GitHub.

```terminal
git init -b main
git remote add origin https://github.com/<username>/prisma-mongodb-cloudflare
git add .
git commit -m "initial commit"
git push -u origin main
```

You're ready to import your project into the Prisma Data Platform.

## 7. Importing your Project into the Prisma Data Platform

With Cloudflare Workers, you can't directly access your database because there is no TCP support. Fortunately, Prisma has your back with the Prisma Data Proxy.

To get started, sign up for a free [Prisma Data Platform account](https://cloud.prisma.io/).

![Signup for Prisma Data Platform](https://github.com/coding-to-music/prisma-mongodb-cloudflare/blob/main/images/450-02-signup-pdp.png?raw=true)

Once you're in, click **New Project**, then **Import a Project**. Fill in the repository and project details, and then click **Create Project**.

![Import a Project](https://github.com/coding-to-music/prisma-mongodb-cloudflare/blob/main/images/450-03-import-project.png?raw=true)

Next, you'll connect the Prisma Data Platform to MongoDB Atlas database and set up the Prisma Data Proxy:

![Connect your Database](https://github.com/coding-to-music/prisma-mongodb-cloudflare/blob/main/images/450-04-connect-db.png?raw=true)

Click **Create Project** to test the connection and set up the Data Proxy.

<Admonition type="warning">

If you see, "The database needs to be empty to proceed", you can simply use a
different database name. Using the screenshot above, that would be renaming
`logs` to something else.

</Admonition>

If all goes well, you'll be greeted with a new connection string that starts with `prisma://`. Copy this connection string to your clipboard.

![Data Proxy Page](https://github.com/coding-to-music/prisma-mongodb-cloudflare/blob/main/images/450-05-data-proxy.png?raw=true)

Then you can hop back into your code editor and paste the connection string into your `.env` file:

```env
DATABASE_URL="prisma://aws-us-east-1.prisma-data.com/?api_key=•••••••••••••••••"
```

You're all setup and ready to generate a Prisma Client!

<Admonition type="warning">
Please be aware that the Data Proxy is in Early Access and should not be used in production.
</Admonition>

## 9. Generate a Prisma Client

Next you'll generate a Prisma Client that connects through the Data Proxy over HTTP.

```terminal
PRISMA_CLIENT_ENGINE_TYPE=dataproxy npx prisma generate
```

This client is optimized for edge environments like Cloudflare Workers.

<Admonition type="warning">

Please be aware that the proxy-enabled Prisma Client is in Early Access and subject to change.

</Admonition>

## 10. Develop the Cloudflare Worker function

You're now ready to create a Cloudflare Worker. Create a `src/index.ts` file with the following code:

```ts
import { PrismaClient } from "@prisma/client";
const prisma = new PrismaClient();

addEventListener("fetch", (event) => {
  event.respondWith(handleEvent(event));
});

async function handleEvent(event: FetchEvent): Promise<Response> {
  const { request } = event;

  // waitUntil method is used for sending logs, after response is sent
  event.waitUntil(
    prisma.log.create({
      data: {
        level: "Info",
        message: `${request.method} ${request.url}`,
        meta: {
          headers: JSON.stringify(request.headers),
        },
      },
    })
  );
  return new Response(`request method: ${request.method}!`);
}
```

Run `npx wrangler dev` to see your worker in development:

```java
npx wrangler dev
```

Output

```
👂  Listening on http://127.0.0.1:8787
```

Go ahead and open `http://127.0.0.1:8787`. If all goes well, you should see:

```
request method: GET!
```

Refresh the page a couple times to verify that it's working. Now if you click on the **Data Browser** tab on your Prisma Cloud project, you should see Info logs written to your database.

![Prisma Data Browser](https://github.com/coding-to-music/prisma-mongodb-cloudflare/blob/main/images/450-06-data-browser.png?raw=true)

It's working locally!

## 11. Publishing to Cloudflare Workers

You're now ready to deploy to Cloudflare Workers. Run the following command:

```terminal
npx wrangler publish
```

This will pack your application with webpack and upload to Cloudflare. With a bit of luck, you'll see the following:

```
✨  Built successfully, built project size is 94 KiB.
✨  Successfully published your script to
https://prisma-mongodb-cloudflare.mattm.workers.dev
```

Visit your deployment URL and you'll again see:

```
request method: GET!
```

You're all set! You've successfully deployed a Cloudflare Worker written in TypeScript that uses Prisma to talk to your MongoDB database.

Give yourself a pat on the back, you deserve it!

# Errors

```java
npx wrangler dev
```

Output

```java
👀  ./node_modules/@prisma/client/runtime/index.js
Module not found: Error: Can't resolve 'async_hooks' in '/mnt/volume_nyc1_01/prisma-mongodb-cloudflare/node_modules/@prisma/client/runtime'
resolve 'async_hooks' in '/mnt/volume_nyc1_01/prisma-mongodb-cloudflare/node_modules/@prisma/client/runtime'
  Parsed request is a module
  using description file: /mnt/volume_nyc1_01/prisma-mongodb-cloudflare/node_modules/@prisma/client/package.json (relative path: ./runtime)
    Field 'browser' doesn't contain a valid alias configuration
    resolve as module
```

snip, it's very long

```java
[/mnt/volume_nyc1_01/prisma-mongodb-cloudflare/node_modules/util/types]
[/mnt/volume_nyc1_01/prisma-mongodb-cloudflare/node_modules/util/types.ts]
[/mnt/volume_nyc1_01/prisma-mongodb-cloudflare/node_modules/util/types.tsx]
[/mnt/volume_nyc1_01/prisma-mongodb-cloudflare/node_modules/util/types.js]
 @ ./node_modules/@prisma/client/runtime/index.js 33546:27-48
 @ ./node_modules/.prisma/client/index.js
 @ ./node_modules/@prisma/client/index.js
 @ ./src/index.ts
Error: webpack returned an error. Try configuring `entry` in your webpack config relative to the current working directory, or setting `context = __dirname` in your webpack config.
```
