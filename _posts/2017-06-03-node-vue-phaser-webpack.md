---
layout: post
title:  "Getting Started with Node, Vue, Phaser and Webpack"
date:   2017-06-03 19:59:00
categories: [node, vue, phaser, game]
comments: true
---

Recently I've been working on a few, simple games based on [Phaser](http://phaser.io). I use Node to handle server-side logic, Vue for client-side routing and UI and Phaser to handle creating the actual game. Client and server communicate over websockets.

I usually base my Vue projects on the excellent [Vue Webpack template](https://github.com/vuejs-templates/webpack) and with this latest project I wanted to use this same template to get started, but adding a Node/Express server and Phaser into the mix and getting it all set up required a little bit more tinkering. As the JavaScript eco-system is so vast nowadays, finding an example for this exact setup was a bit difficult so I will try to provide some guidance in this post of what I ended up with.

##### Let's start with the Vue Webpack template!

As per their documentation:
```
npm install -g vue-cli
vue init webpack my-project
cd my-project
npm install
npm run dev
```

##### And then set up the Node/Express server!

In this case I'm mainly following the template structure and placing all my code in the `src` folder. The Node server will get its own directory inside `src`. Let's install express.

`npm install --save express`

Then let's create a simple script for the server in `src/server/server.js`:
``` javascript
var express = require('express')
var app = express()

app.get('/api', (req, res) => {
  res.json({message: 'Welcome to the Server'})
})

app.listen(8081, () => {
  console.log('API listening on port 8081')
})
```

Now to run the server automatically whenever we build the rest of the app with Webpack, let's install nodemon just for our dev environment and add it to our scripts in package.json.

`npm install --save-dev nodemon`

``` javascript
"scripts": {
  "dev": "npm run server | npm run start",
  "start": "node build/dev-server.js",
  "build": "node build/build.js",
  "lint": "eslint --ext .js,.vue src",
  "server": "nodemon ./src/server/server.js localhost 8081"
},
```

Now try to run `npm run dev` and see if the server starts running at localhost:8081 and the Vue client should run using webpack-dev-server on localhost:8080 with hot-reload.

##### Adding Phaser

The tricky part when adding Phaser is that it's not built in a modular way, so importing it requires a little more work. Let's install it first:

`npm install --save phaser-ce`

If you read the [documentation](https://github.com/photonstorm/phaser-ce) for Phaser, you will notice some comments about using it with Webpack, which will point you in the right direction.

Let's try to import Phaser and its requirements in our app. For me, this would be in a Vue component called `Game.vue`:

``` javascript
<template>
  <div id='gameScreen'></div>
</template>

<script>
  /* eslint-disable no-unused-vars */
  import 'pixi'
  import 'p2'
  import Phaser from 'phaser'
  /* eslint-enable no-unused-vars */

  export default{
    name: 'game',
    props: {
      width: Number,
      height: Number
    },
    mounted () {
      let self = this
      if (this.game == null) {
        this.game = new Phaser.Game(this.width, this.height, Phaser.AUTO, this.$el, {
          preload: function preload () {
            self.preload(this)
          },
          create: function create () {
            self.create(this)
          },
          update: function update () {
            self.update(this)
          }
        })
      }
    },
    methods: {
      preload () {
      },
      create (phaser) {
      },
      update (phaser) {
      }
    },
    data () {
      return {
        game: null
      }
    }
  }
</script>
```

However, to get it working with Webpack we need to modify the `webpack.base.conf.js` in our build folder so it uses the loaders for Phasers. In the example below I've omitted the rest of the config for brevity. There are no changes from the original config included in the Vue Webpack template, we just add a few bits and pieces to handle Phaser.

``` javascript
// some config omitted here
var phaserModule = path.join(__dirname, '../node_modules/phaser-ce/')
var phaser = path.join(phaserModule, 'build/custom/phaser-split.js')
var pixi = path.join(phaserModule, 'build/custom/pixi.js')
var p2 = path.join(phaserModule, 'build/custom/p2.js')

module.exports = {
  entry: {
    app: './src/main.js',
    vendor: ['pixi', 'p2', 'phaser']
  },
  output: {
    path: config.build.assetsRoot,
    filename: '[name].js',
    publicPath: process.env.NODE_ENV === 'production'
      ? config.build.assetsPublicPath
      : config.dev.assetsPublicPath
  },
  resolve: {
    extensions: ['.js', '.vue', '.json'],
    alias: {
      'vue$': 'vue/dist/vue.esm.js',
      '@': resolve('src'),
      'phaser': phaser,
      'pixi': pixi,
      'p2': p2
    }
  },
  module: {
    rules: [
      {
        test: /\.(js|vue)$/,
        loader: 'eslint-loader',
        enforce: 'pre',
        include: [resolve('src'), resolve('test')],
        options: {
          formatter: require('eslint-friendly-formatter')
        }
      },
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: vueLoaderConfig
      },
      { test: /pixi\.js/, use: ['expose-loader?PIXI'] },
      { test: /phaser-split\.js$/, use: ['expose-loader?Phaser'] },
      { test: /p2\.js/, use: ['expose-loader?p2'] }
      // Rest of the rules omitted
    ]
  }
}
```

If you try to run the app again now, Webpack should find Phaser and build it for us. We can now include the Phaser game object, implement some sort of Node API or integrate everything with WebSockets and get to work. ;)

I've made the result of this work available as a [boilerplate](https://github.com/sekl/node-vue-phaser-boilerplate) on GitHub. Please let me know if you run into any issues.
