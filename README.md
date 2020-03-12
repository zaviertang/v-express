# 模拟实现 [express](https://github.com/expressjs/express)

```javascript
const http = require('http')
const slice = Array.prototype.slice

class VExpress {
  constructor() {
    this.middlewares = []
  }

  _register(handle, path = '/', method = 'all') {
    const m = { path, method, handle }
    this.middlewares.push(m)
  }

  use() {
    typeof arguments[0] === 'string'
      ? this._register(slice.call(arguments, 1), arguments[0])
      : this._register(slice.call(arguments, 0))
  }

  get(path, ...handle) {
    this._register(handle, path, 'GET')
  }

  post(path, ...handle) {
    this._register(handle, path, 'POST')
  }

  _match(method, url) {
    let handlerList = []
    this.middlewares.forEach((m) => {
      if ([method, 'all'].indexOf(m.method) != -1 && pathMatch(m.path, url)
      ) {
        handlerList = handlerList.concat(m.handle)
      }
    })
    console.log('handleList:', handlerList.length)
    return handlerList
  }
  
  _run(req, res, middlewares) {
    const next = () => {
      const m = middlewares.shift()
      if (m) {
        m(req, res, next)
      } else {
        console.log('run end...')
      }
    }
    next()
  }

  _requestHandle(req, res) {
    res.json = (data) => {
      res.setHeader('Content-type', 'application/json')
      res.end(JSON.stringify(data))
    }

    const handlerList = this._match(
      req.method.toUpperCase(),
      req.url
    )
    this._run(req, res, handlerList)
  }

  listen() {
    const server = http.createServer(this._requestHandle.bind(this))
    server.listen(...arguments)
  }
}

module.exports = () => {
  return new VExpress()
}
```
