# Dynamic Routes for Next.js

[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg)](https://conventionalcommits.org)
[![Coverage Status](https://coveralls.io/repos/github/trovit/next-routes/badge.svg)](https://coveralls.io/github/trovit/next-routes)
[![Build Status](https://travis-ci.org/trovit/next-routes.svg?branch=master)](https://travis-ci.org/trovit/next-routes)

Easy to use universal dynamic routes for [Next.js](https://github.com/zeit/next.js)

- Express-style route and parameters matching
- Request handler middleware for express & co
- `Link` and `Router` that generate URLs by route definition

----
> NOTE: This project was forked from [next-routes](https://github.com/fridays/next-routes) on version 1.0.40.
----

## How to use

Install:

```bash
npm install next-routes --save
```

Create `routes.js` inside your project:

```javascript
const routes = module.exports = require('next-routes')()

routes
  .add('about')
  .add('blog', '/blog/:slug')
  .add('user', '/user/:id', 'profile')
  .add('/:noname/:lang(en|es)/:wow+', 'complex')
  .add({name: 'beta', pattern: '/v3', page: 'v3'})
  .add({name: 'beta', pattern: '/:noname/:lang', page: 'v3', params: {type: 2}})
```

This file is used both on the server and the client.

API:

- `routes.add(name, pattern = /name, page = name)`
- `routes.add(pattern, page)`
- `routes.add(object)`

Arguments:

- `name` - Route name
- `pattern` - Route pattern (like express, see [path-to-regexp](https://github.com/pillarjs/path-to-regexp))
- `page` - Page inside `./pages` to be rendered
- `object` - Contains properties `name`, `pattern` and `page` in addition to `params` that is an object with extra properties that will be added to the `query` object. It can be used to set default value or to attach new properties.

The page component receives the matched URL parameters merged into `query`. The `query` will receive:

- Matched properties on the pattern
- Extra properties specified in the `params` object
- A `routeName` property with the value of the `name` param. 

```javascript
export default class Blog extends React.Component {
  static async getInitialProps ({query}) {
    // query.slug
  }
  render () {
    // this.props.url.query.slug
  }
}
```

## On the server

```javascript
// server.js
const next = require('next')
const routes = require('./routes')
const app = next({dev: process.env.NODE_ENV !== 'production'})
const handler = routes.getRequestHandler(app)

// With express
const express = require('express')
app.prepare().then(() => {
  express().use(handler).listen(3000)
})

// Without express
const {createServer} = require('http')
app.prepare().then(() => {
  createServer(handler).listen(3000)
})
```

Optionally you can pass a custom handler, for example:

```javascript
const handler = routes.getRequestHandler(app, ({req, res, route, query}) => {
  app.render(req, res, route.page, query)
})
```

Make sure to use `server.js` in your `package.json` scripts:

```json
"scripts": {
  "dev": "node server.js",
  "build": "next build",
  "start": "NODE_ENV=production node server.js"
}
```

## On the client

Import `Link` and `Router` from your `routes.js` file to generate URLs based on route definition:

### `Link` example

```jsx
// pages/index.js
import {Link} from '../routes'

export default () => (
  <div>
    <div>Welcome to Next.js!</div>
    <Link route='blog' params={{slug: 'hello-world'}}>
      <a>Hello world</a>
    </Link>
    or
    <Link route='/blog/hello-world'>
      <a>Hello world</a>
    </Link>
  </div>
)
```

API:

- `<Link route='name'>...</Link>`
- `<Link route='name' params={params}> ... </Link>`
- `<Link route='/path/to/match'> ... </Link>`

Props:

- `route` - Route name or URL to match (alias: `to`)
- `params` - Optional parameters for named routes

It generates the URLs for `href` and `as` and renders `next/link`. Other props like `prefetch` will work as well.

### `Router` example

```jsx
// pages/blog.js
import React from 'react'
import {Router} from '../routes'

export default class Blog extends React.Component {
  handleClick () {
    // With route name and params
    Router.pushRoute('blog', {slug: 'hello-world'})
    // With route URL
    Router.pushRoute('/blog/hello-world')
  }
  render () {
    return (
      <div>
        <div>{this.props.url.query.slug}</div>
        <button onClick={this.handleClick}>Home</button>
      </div>
    )
  }
}
```

API:

- `Router.pushRoute(route)`
- `Router.pushRoute(route, params)`
- `Router.pushRoute(route, params, options)`

Arguments:

- `route` - Route name or URL to match
- `params` - Optional parameters for named routes
- `options` - Passed to Next.js

The same works with `.replaceRoute()` and `.prefetchRoute()`

It generates the URLs and calls `next/router`

---

Optionally you can provide custom `Link` and `Router` objects, for example:

```javascript
const routes = module.exports = require('next-routes')({
  Link: require('./my/link')
  Router: require('./my/router')
})
```

---

##### Related links

- [zeit/next.js](https://github.com/zeit/next.js) - Framework for server-rendered React applications
- [path-to-regexp](https://github.com/pillarjs/path-to-regexp) - Express-style path to regexp
