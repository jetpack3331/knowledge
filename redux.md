# Redux

## RxJS epic middleware example

```javascript
// Action creator
const fetchTodos = () => {
  return {
    type: AJAX_REQUEST,
    loading: () => {type: FETCH_TODOS},
    done: (data, error) => {type: FETCH_TODOS_DONE, data, error}
    request: {
      url: 'http://example.com/api/todos',
      method: 'POST',
      body: {
        ...
      }
    }
  }
}

// Epic middleware
const ajaxEpicMiddleware = (action$, store) => {
  return action$.ofType(AJAX_REQUEST)
    .do(action => store.dispatch(action.loading ? action.loading() : {type: AJAX_REQUEST_START})) // Will dispatch loading
    .switchMap(action => {
      const {url, method, body, headers} = action.request;
      return ajax({
        url,
        method: method ? method.toLowerCase() : 'get',
        body,
        headers: headers ? headers : {},
        responseType: 'json'
      })
      .map(request => action.done(request.response, null))
      .catch(error => Observable.of(action.done(null, error)))
      .takeUntil(action$.ofType(AJAX_REQUEST_CANCEL))
    })

};
```
