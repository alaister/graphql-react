![Logo](https://cdn.rawgit.com/jaydenseric/graphql-react/b2e60e80/graphql-react-logo.svg)

# graphql-react

[![npm version](https://img.shields.io/npm/v/graphql-react.svg)](https://npm.im/graphql-react) ![Licence](https://img.shields.io/npm/l/graphql-react.svg) [![Github issues](https://img.shields.io/github/issues/jaydenseric/graphql-react.svg)](https://github.com/jaydenseric/graphql-react/issues) [![Github stars](https://img.shields.io/github/stars/jaydenseric/graphql-react.svg)](https://github.com/jaydenseric/graphql-react/stargazers) [![Travis status](https://img.shields.io/travis/jaydenseric/graphql-react.svg)](https://travis-ci.org/jaydenseric/graphql-react)

A lightweight GraphQL client for React.

> ⚠️ SSR API coming soon.

#### Easy 🤹

* Simple components, no decorators.
* Query components fetch on mount and when props change. While loading, cache from the last identical request is available to display.
* Automatically fresh cache, even after mutations.
* Use file input values as mutation arguments to upload files; compatible with [a variety of servers](https://github.com/jaydenseric/graphql-multipart-request-spec#server).

#### Smart 👨‍🔬

* Adds ~ 4 KB to a typical min+gzip bundle.
* [Native ESM in Node.js](https://nodejs.org/api/esm.html) via `.mjs`.
* [Package `module` entry](https://github.com/rollup/rollup/wiki/pkg.module) for [tree shaking](https://developer.mozilla.org/docs/Glossary/Tree_shaking) bundlers.
* Components use the [React v16.3 context API](https://github.com/facebook/react/pull/11818).
* All fetch options overridable per request.
* Request hash based cache:
  * No data denormalization and no need to query `id` fields.
  * No tampering with queries or `__typename` insertion.
  * Query multiple GraphQL services without stitching data.
  * Errors also cache, enabling error SSR.

## Setup

Install with [npm](https://npmjs.com):

```shell
npm install graphql-react
```

Create and provide a GraphQL client:

```jsx
import { GraphQL, GraphQLProvider } from 'graphql-react'

const graphql = new GraphQL()

const Page = () => (
  <GraphQLProvider value={graphql}>
    <!-- Now use GraphQLQuery or GraphQLMutation components -->
  </GraphQLProvider>
)
```

## Example

See the [example Next.js app and GraphQL API](example/readme.md).

## API

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

#### Table of Contents

* [GraphQLQuery](#graphqlquery)
* [GraphQLMutation](#graphqlmutation)
* [RenderQuery](#renderquery)
* [GraphQL](#graphql)
  * [reset](#reset)
  * [query](#query)
* [RequestCache](#requestcache)
* [CacheUpdateCallback](#cacheupdatecallback)
* [RequestCachePromise](#requestcachepromise)
* [ActiveQuery](#activequery)
* [HTTPError](#httperror)
* [RequestOptionsOverride](#requestoptionsoverride)
* [RequestOptions](#requestoptions)
* [Operation](#operation)

### GraphQLQuery

A React component to manage a GraphQL query.

**Parameters**

* `props` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** Component props.
  * `props.variables` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)?** GraphQL query variables.
  * `props.query` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** GraphQL query.
  * `props.loadOnMount` **[Boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** Should the query load when the component mounts. (optional, default `true`)
  * `props.loadOnReset` **[Boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** Should the query load when the GraphQL client cache is reset. (optional, default `true`)
  * `props.resetOnLoad` **[Boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** Should the GraphQL client cache reset when the query loads. (optional, default `false`)
* `children` **[RenderQuery](#renderquery)** Render function.

**Examples**

```javascript
import { GraphQLQuery } from 'graphql-react'

const Profile = ({ userId }) => (
  <GraphQLQuery
    variables={{ userId }}
    query={`
      query user($userId: ID!) {
        user(userId: $id) {
          name
        }
      }
    `}
  >
    {({ load, loading, httpError, parseError, graphQLErrors, data }) => (
      <article>
        <button onClick={load}>Reload</button>
        {loading && <span>Loading…</span>}
        {(httpError || parseError || graphQLErrors) && <strong>Error!</strong>}
        {data && <h1>{data.user.name}</h1>}
      </article>
    )}
  </GraphQLQuery>
)
```

Returns **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** HTML.

### GraphQLMutation

A React component to manage a GraphQL mutation. The same as [GraphQLQuery](#graphqlquery) but with different default props.

**Parameters**

* `props` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** Component props.
  * `props.variables` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)?** GraphQL query variables.
  * `props.query` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** GraphQL query.
  * `props.loadOnMount` **[Boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** Should the query load when the component mounts. (optional, default `false`)
  * `props.loadOnReset` **[Boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** Should the query load when the GraphQL client cache is reset. (optional, default `false`)
  * `props.resetOnLoad` **[Boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** Should the GraphQL client cache reset when the query loads. (optional, default `true`)
* `children` **[RenderQuery](#renderquery)** Render function.

**Examples**

```javascript
import { GraphQLMutation } from 'graphql-react'

const ClapArticleButton = ({ articleId }) => (
  <GraphQLMutation
    variables={{ articleId }}
    query={`
      mutation clapArticle($articleId: ID!) {
        clapArticle(articleId: $id) {
          clapCount
        }
      }
    `}
  >
    {({ load, loading, httpError, parseError, graphQLErrors, data }) => (
      <aside>
        <button onClick={load} disabled={loading}>
          Clap
        </button>
        {(httpError || parseError || graphQLErrors) && <strong>Error!</strong>}
        {data && <p>Clapped {data.clapArticle.clapCount} times.</p>}
      </aside>
    )}
  </GraphQLMutation>
)
```

Returns **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** HTML.

### RenderQuery

Renders the status of a query or mutation.

Type: [Function](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/function)

**Parameters**

* `load` **[Function](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/function)** Loads the query on demand, updating cache.
* `loading` **[Boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** Is the query loading.
* `httpError` **[HTTPError](#httperror)?** Fetch HTTP error.
* `parseError` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)?** Parse error message.
* `graphQLErrors` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)?** GraphQL response errors.
* `data` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)?** GraphQL response data.

**Examples**

```javascript
;({ load, loading, httpError, parseError, graphQLErrors, data }) => (
  <aside>
    <button onClick={load}>Reload</button>
    {loading && <span>Loading…</span>}
    {(httpError || parseError || graphQLErrors) && <strong>Error!</strong>}
    {data && <h1>{data.user.name}</h1>}
  </aside>
)
```

Returns **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** HTML.

### GraphQL

A lightweight GraphQL client with a request cache.

**Parameters**

* `options` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** Options. (optional, default `{}`)
  * `options.cache` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** Cache to import; useful once a SSR API is available. (optional, default `{}`)
  * `options.requestOptions` **[RequestOptionsOverride](#requestoptionsoverride)?** A function that accepts and modifies generated options for every request.

**Examples**

```javascript
const graphql = new GraphQL({
  requestOptions: options => {
    options.url = 'https://api.example.com/graphql'
    options.credentials = 'include'
  }
})
```

#### reset

Resets the cache. Useful when a user logs out.

**Examples**

```javascript
graphql.reset()
```

#### query

Queries a GraphQL server.

**Parameters**

* `operation` **[Operation](#operation)** GraphQL operation object.

Returns **[ActiveQuery](#activequery)** In-flight query details.

### RequestCache

JSON serializable result of a request (including all errors and data) for caching purposes.

Type: [Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)

**Properties**

* `httpError` **[HTTPError](#httperror)?** Fetch HTTP error.
* `parseError` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)?** Parse error message.
* `graphQLErrors` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)?** GraphQL response errors.
* `data` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)?** GraphQL response data.

### CacheUpdateCallback

A cache update listener callback.

Type: [Function](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/function)

**Parameters**

* `requestCache` **[RequestCache](#requestcache)** Request cache.

### RequestCachePromise

A promise for an in-flight query that resolves the request cache.

Type: [Promise](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise)&lt;[RequestCache](#requestcache)>

### ActiveQuery

Type: [Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)

**Properties**

* `pastRequestCache` **[RequestCache](#requestcache)?** Results from the last identical request.
* `requestHash` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** Request options hash.
* `request` **[RequestCachePromise](#requestcachepromise)** Promise that resolves fresh request cache.

### HTTPError

Fetch HTTP error.

Type: [Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)

**Properties**

* `status` **[Number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)** HTTP status code.
* `statusText` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** HTTP status text.

### RequestOptionsOverride

A way to override request options generated for a fetch. Modify the provided options object directly; no return.

Type: [Function](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/function)

**Parameters**

* `requestOptions` **[RequestOptions](#requestoptions)**

**Examples**

```javascript
options => {
  options.url = 'https://api.example.com/graphql'
  options.credentials = 'include'
}
```

### RequestOptions

Options for a GraphQL fetch request. See [polyfillable fetch options](https://github.github.io/fetch/#options).

Type: [Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)

**Properties**

* `url` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** A GraphQL API URL.
* `body` **([String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String) \| [FormData](https://developer.mozilla.org/docs/Web/API/FormData))** HTTP request body.
* `headers` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** HTTP request headers.
* `credentials` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)?** Authentication credentials mode.

### Operation

A GraphQL operation object. Additional properties may be used; all are sent to the GraphQL server.

Type: [Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)

**Properties**

* `query` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** GraphQL queries or mutations.
* `variables` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** Variables used by the query.

## Support

* Node.js v6.10+.
* Browsers [>1% usage](http://browserl.ist/?q=%3E1%25).
