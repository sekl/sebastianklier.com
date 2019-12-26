---
title: "Create a Multiplayer Game with TypeScript, Node, Express, Socket.io and Phaser"
date: "2019-03-05T12:00:00.284Z"
layout: post
categories: [javascript, node, gamedev, phaser]
comments: true
---

This week I've started work on a new side-project, a multiplayer game based on Node, Websockets and Phaser. I'll go into more detail on each of my choices and also provide a starter pack that already has everything set up.

<!--more-->

As so often when starting a new project, I've spent a considerable amount of time going back and forth in my head trying to decide which stack to use. Every time I work on a game like this I'm tempted to just go with a Unity client - and I promise one day I will - and maybe Golang on the backend. But for now I chose JavaScript once again, for several reasons.

- I know JavaScript well and being able to use it on both backend and frontend *slightly* cuts down on cognitive load when switching back and forth. A minor advantage, but it also lets me re-use code, such as model definitions, more easily.
- I can use **TypeScript**. This brings one of my main reasons why I would normally choose Golang - type safety - to JavaScript, and I don't even have to give up any of JS flexibility.
- It lets me prototype much faster and try things out in the browser easily. I'm more familiar with web development than working with Unity, so I'll be more productive and spend less time learning the ins and outs of Unity.

Alright, after we've got that out of the way, let's set up a simple project. We'll add to it as we go along. Starting a new JS project can quickly escalate into fighting various transpilers, bundlers, task runners and whatnot, so I prefer to start simple, add one thing after another, keeping a working state as we go, and then adding more features to it later.

If you want to skip ahead and just get started with the code, you can find the repository on [GitHub](https://github.com/sekl/typescript-express-socketio-phaser-starter).

### Creating the project and server

Create a new directory for this project and `npm init`.

You can install TypeScript globally using `npm install -g typeScript`.

Then let's start with Express. As we will be using TypeScript, we also need to get the type definitions for Express:

`npm install express @types/express`

And create a `tsconfig.json` in the projects root folder with the following config:

``` json
{
  "compilerOptions": {
      "module": "commonjs",
      "esModuleInterop": false,
      "target": "es6",
      "noImplicitAny": true,
      "moduleResolution": "node",
      "sourceMap": true,
      "outDir": "dist",
      "baseUrl": ".",
      "paths": {
          "*": [
              "node_modules/",
              "src/types/*"
          ]
      }
  },
  "include": [
      "src/**/*"
  ]
}
```

This config will take our TS source files from the `src` directory and subfolders and transpile it to JS. We'll see how that works once we actually have some source code. So let's create the following folder structure:

```
- src
  - client
    - game.ts
    - index.html
  - server
    - server.ts
  - shared
    - model
      player.ts
```

We'll fill those with some basic logic in a minute. 

First let's add Socket.io for our Websocket connection: `npm install socket.io @types/socket.io`

Then let's create a simple server that can serve some HTML:

``` typescript
// src/server/server.ts
import * as express from "express"
import * as path from "path"

const app = express()
app.set("port", process.env.PORT || 9001)

let http = require("http").Server(app)
let io = require("socket.io")(http)

app.get("/", (req: any, res: any) => {
  res.sendFile(path.resolve("./src/client/index.html"))
})

io.on("connection", function(socket: any) {
  console.log("Client connected!")
  socket.on("msg", function(msg: any) {
    console.log(msg)
  })
})

http.listen(9001, function() {
  console.log("listening on *:9001")
})
```

Feel free to change the port. We'll make it configurable as an environment variable in another post.

For our client, we'll need a simple HTML file that we can use to establish a quick Websocket connection to see if it's working. The following should be enough for now, and already includes a `div` element that we can use later for Phaser to inject our game client, as well as the JS:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title></title>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
  </head>
  <body>
    <div id='game'></div>

    <script src="/socket.io/socket.io.js"></script>
    <script>
      const socket = io("http://localhost:9001")
      socket.emit("msg", "test")
    </script>
    <script src="game.js" type="text/javascript"></script>
  </body>
</html>
```

Note that we're serving the Socket.io client locally. This is a neat feature for development that Socket.io provides out of the box, but we'll have to change it in the future for production.

At this point we should already be able to test our server and websocket connection.

Run `tsc` in the project root to let the TypeScript compiler create our JS files. You should end up with a `dist` folder containing our server/client structure, some JavaScript files and some source maps (for TypeScript).

Let's see if this runs: `node dist/server/server.js`.

You should see `listening on *:9001`. Now visit `http://localhost:9001` in your browser. There is nothing to see on the page, but you should see the websocket connection in the dev tools, and you should see the following log messages in the server output:

```
Client connected!
test
```

### Adding the game client

So we've got our server running and a websocket connection established. Awesome! So now let's add Phaser. This is a great framework for HTML5 games using canvas or WebGL, and I'll go into much more detail about it in future posts. Unfortunately at this time, getting Phaser's TS typings is slightly more work than with other libraries, but it's not the end of the world. Let's install everything we need:

`npm install phaser@3.16.2`
and
`npm install -D github:photonstorm/phaser3-docs`

Then we have to add the following line to our `tsconfig.json`, at the top level, for example right after `include`:
``` json
"include": [
    "src/**/*"
],
"files": ["node_modules/phaser3-docs/typeScript/phaser.d.ts"]
```

At this point we can add some game logic. Let's initialize Phaser in `src/client/game.ts`:

``` typescript
import 'phaser'

const config: GameConfig = {
  type: Phaser.AUTO,
  width: 640,
  height: 480,
  parent: 'game',
  scene: {
    preload: {},
    create: {},
  }
}

export class Game extends Phaser.Game {
  constructor(config: GameConfig) {
    super(config)
  }
}

const game = new Game(config)
```

And add a simple player model in `src/shared/model/player.ts`:

``` typescript
export class Player {
  private name!: string

  constructor (name: string) {
    this.name = name
  }

  getName (): string {
    return this.name
  }
}
```

We are now already able to use this new structure in our server code by changing this function:

``` typescript
io.on("connection", function(socket: any) {
  console.log("Client connected!")
  socket.on("msg", function(msg: any) {
    let player = new Player(msg)
    console.log(player.getName())
  })
})
```

### All the other usual JavaScript stuff

The next step would be to get an asset pipeline going. Let's install Parcel, my new favorite JavaScript bundler. It comes with zero configuration required, and in many cases can save you hours of configuring Webpack. While we're at it, let's install a few more packages that we'll need to get our build scripts working:

`npm install -D parcel-bundler @types/parcel-bundler rimraf npm-run-all nodemon`

Armed with this list of tools, let's add some scripts to our `package.json`:

```
"scripts": {
  "clean": "rimraf dist/*",
  "tsc": "tsc",
  "assets": "cp -R src/client/index.html src/client/css src/client/img dist/client",
  "parcel": "parcel build src/client/game.ts -d dist/client",
  "build": "npm-run-all clean tsc assets parcel",
  "start": "node dist/server/server.js",
  "dev": "nodemon --watch src -e ts --exec npm-run-all build start"
},
```

There is a lot going on here, but most of this is common to many JavaScript projects.

- `clean` simply wipes out the `dist` folder to make sure we start with a clean state before building
- `tsc` runs the TypeScript compiler
- `assets` copies the `index.html` file as well as any css or images to the `dist` folder. When you add those later, make sure to mirror the folder structure here.
- `parcel` creates a minified bundle for the client, with the `game.ts` as an entry point. The result along with the source files then gets written to the `dist` folder as well. 
- `build` combines all of the above
- `start` runs the server
- `dev` builds and starts all our code, while at the same time watching for any changes and re-building if necessary

We also want to tell Express where to serve our static files from:

``` typescript
// src/server/server/ts
// ...
let http = require("http").Server(app)
let io = require("socket.io")(http)

app.use(express.static(path.join( __dirname, "../client")))
// ...
```

So from now on we can run `npm run dev` to start everything up. If you do that now, you should see a black canvas where Phaser injects the client - and if you look closer you will also see some errors in the JS console because we didn't give Phaser a valid scene to work with. But we can fix that later!

You can find the full source code on [GitHub](https://github.com/sekl/typescript-express-socketio-phaser-starter).

By the way, if you are interested in online games, you might want to check out my website for Amazon's [New World MMO](https://newworldfans.com). I also have a [New World Discord](https://newworldfans.com/discord) server running, and developed a bot that you can use on your server to get notified when there are new developer posts!
