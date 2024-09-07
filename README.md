# vue3-in-vue2-mf
micro frontend,3 apps, app1-vue2-host,app2-vue2-remote.but app2-vue2-host too, app3-vue3-remote

error: Error loading remote component  app3 not found while loading

app2 webpack config
```
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin')
new ModuleFederationPlugin({
        name: `config.name`,
        filename: `remoteEntry.js`,
        library: {
          type: 'window',
          name: `config.name`
        },
        exposes: {
          './main': './src/main/index.js',
        },
        remotes: {
          app3: 'app3@http://localhost:3000/assets/remoteEntry-app3.js?timestamp='+new Date().getTime(),
        },
        shared: {
          vue: {
            singleton: true,
            requiredVersion: '^3.2.0',
          },
        }
      })
```

app2 public/index.html
```
<!DOCTYPE html>
<html lang="">

<head>
  <meta charset="utf-8">
  <meta name="referrer" content="no-referrer">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width,initial-scale=1.0">
  <link rel="icon" id="faviconLink" href="static/img/favicon.ico">
    <script src="http://localhost:3000/assets/remoteEntry-app3.js"></script>
</head>

  <body>
    <div id="app"></div>
  </body>
</html>

```
app2 test bootstrap app3 component

bootstrap.js
```
const loadRemote = async () => {
  try {
    const remote = await import('app3/Component')
    console.log('Successfully loaded remote component:', remote)
  } catch (error) {
    console.error('Error loading remote component:', error)
  }
}

loadRemote()
```
app2 main.js

```
import('./bootstrap').catch((err) => console.error('Bootstrap error:', err))
```



app3 webpack config
```
import { defineConfig } from 'vite'
import federation from "@originjs/vite-plugin-federation"
export default defineConfig({
  plugins:[
    federation({
      name: 'app3',
      filename: 'remoteEntry-app3.js',
      exposes: {
        './App': './src/App.vue',
        './Component': './src/views/index.vue'
      },
      shared: {
        vue: {
          singleton: true, 
          requiredVersion: '^3.2.0',
          eager: true
        },  
      },
    })
  ],
  server: {
    port: 3000,
    open: true,
    proxy: {
    },
    cors: true,
    headers: {
      "Access-Control-Allow-Origin": "*",
      "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, PATCH, OPTIONS",
      "Access-Control-Allow-Headers": "X-Requested-With, content-type, Authorization"
    },
  }
})
```
app1 bootstrap app2 totally correctly,so pay attention to the problem,which is app2 bootstrap app3

error detail
```
Error loading remote component: Error: app3 not found
while loading "./Component" from webpack/container/reference/app3
```



expecting:app2 bootstrap app3 totally correctly too
