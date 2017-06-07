# Webpack

## Webpack doesn't reload correctly after code changes
This happend more often on Windows OS

Add into webpack
```javascript
watchOptions: {
  aggregateTimeout: 300,
  poll: 1000,
},
  ```
[Watch and watchOptions](https://webpack.js.org/configuration/watch/)
