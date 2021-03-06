# Apiato REST Data Provider For React-Admin

Apiato REST Data Provider for [react-admin](https://github.com/marmelab/react-admin).

## Installation

```sh
npm install --save ra-data-apiato-rest
```

## REST Dialect

This Data Provider fits REST APIs using simple GET parameters for filters and sorting. This is the dialect used for instance in [FakeRest](https://github.com/marmelab/FakeRest).

| Method             | API calls
|--------------------|----------------------------------------------------------------
| `getList`          | `GET http://my.api.url/posts?limit=10&page=1`
| `getOne`           | `GET http://my.api.url/posts/123`
| `getMany`          | `GET http://my.api.url/posts`
| `getManyReference` | `GET http://my.api.url/posts`
| `create`           | `POST http://my.api.url/posts/123`
| `update`           | `PUT http://my.api.url/posts/123`
| `updateMany`       | Multiple calls to `PUT http://my.api.url/posts/123`
| `delete`           | `DELETE http://my.api.url/posts/123`
| `deteleMany`       | Multiple calls to `DELETE http://my.api.url/posts/123`

**Note**: The Apiato REST data provider expects the API to include a `Content-Range` header in the response to `getList` calls. The value must be the total number of resources in the collection. This allows react-admin to know how many pages of resources there are in total, and build the pagination controls.

```
Content-Range: posts 0-24/319
```

If your API is on another domain as the JS code, you'll need to whitelist this header with an `Access-Control-Expose-Headers` [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS) header.

```
Access-Control-Expose-Headers: Content-Range
```

## Usage

```jsx
// in src/App.js
import React from 'react';
import { Admin, Resource } from 'react-admin';
import apiatoRestProvider from 'ra-data-apiato-rest';

import { PostList } from './posts';

const App = () => (
    <Admin dataProvider={apiatoRestProvider('http://path.to.my.api/')}>
        <Resource name="posts" list={PostList} />
    </Admin>
);

export default App;
```

### Adding Custom Headers

The provider function accepts an HTTP client function as second argument. By default, they use react-admin's `fetchUtils.fetchJson()` as HTTP client. It's similar to HTML5 `fetch()`, except it handles JSON decoding and HTTP error codes automatically.

That means that if you need to add custom headers to your requests, you just need to *wrap* the `fetchJson()` call inside your own function:

```jsx
import { fetchUtils, Admin, Resource } from 'react-admin';
import apiatoRestProvider from 'ra-data-apiato-rest';

const httpClient = (url, options = {}) => {
    if (!options.headers) {
        options.headers = new Headers({ Accept: 'application/json' });
    }
    // add your own headers here
    options.headers.set('X-Custom-Header', 'foobar');
    return fetchUtils.fetchJson(url, options);
};
const dataProvider = apiatoRestProvider('http://localhost:3000', httpClient);

render(
    <Admin dataProvider={dataProvider} title="Example Admin">
       ...
    </Admin>,
    document.getElementById('root')
);
```

Now all the requests to the REST API will contain the `X-Custom-Header: foobar` header.

**Tip**: The most common usage of custom headers is for authentication. `fetchJson` has built-on support for the `Authorization` token header:

```js
const httpClient = (url, options = {}) => {
    options.user = {
        authenticated: true,
        token: 'SRTRDFVESGNJYTUKTYTHRG'
    };
    return fetchUtils.fetchJson(url, options);
};
```

Now all the requests to the REST API will contain the `Authorization: SRTRDFVESGNJYTUKTYTHRG` header.

## License

This data provider is licensed under the MIT License, and sponsored by [marmelab](http://marmelab.com).
