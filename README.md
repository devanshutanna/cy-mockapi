# cypress-mockapi

Easily mock your REST API in [Cypress](https://www.cypress.io/) by putting responses in the fixtures directory tree.

## Installation and Setup

```sh
npm install --save-dev cypress-mockapi
```

Import and setup the plugin in `cypress/plugins/index.js`:

```js
import { installPlugin } from 'cypress-mockapi/plugin'

module.exports = (on, config) => {
  installPlugin(on, config);
}
```

Import the Cypress commands in `cypress/support/index.js`

```js
import 'cypress-mockapi/commands';
```

## Usage

Add one or more folders in `cypress/fixtures` to act as the mock API.  Within this folder add files with the naming scheme:

```
{API PATH}/{METHOD}.{EXT}
```

For example:

```
cypress/fixtures/mocks
                  └── user
                      ├── get.json
                      ├── post.json
                      └── messages
                          ├── get.json
                          ├── delete.json
                          └── put.json
```

In your test files:

```js
describe('Some Feature', () => {
  beforeEach(() => {
    cy.server();

    cy.mockApi({
      apiPath: '/api/',
      mocksFolder: './mocks/',
      cache: true
    });
  });

  it('should do the thing', () => {
    // Your test code
  });
})
```

In this example, `cypress-mockapi` will read the `cypress/fixtures/mocks` folder and stub responses to the API requests; functionally equivalent to:

```js
cy.route('GET', '/api/user', 'fixture:mocks/user/get.json').as('GET:user');
cy.route('POST', '/api/user', 'fixture:mocks/user/post.json').as('POST:user');
cy.route('GET', '/api/user/messages', 'fixture:mocks/user/messages/get.json').as('GET:user/messages');
cy.route('DELETE', '/api/user/messages', 'fixture:mocks/user/messages/delete.json').as('DELETE:user/messages');
cy.route('PUT', '/api/user/messages', 'fixture:mocks/user/messages/put.json').as('PUT:user/messages');
```

## Commands

### `cy.mockApi`

Reads and (optionally) caches the `mocksFolder` tree and sets up stubs.  Note that this does not read the fixtures themselves; instead it sets up the stubbing (internally using `cy.route`).  `cy.mockApi` may be called multiple times to mock different sets of fixtures or combine sets of fixtures.

#### Options

- `apiPath` - The API root path to mock.
- `mocksFolder` - The fixtures folder to use.
- `cache` - Boolean to use caching

### `cy.logExtraApiCalls`

Call `cy.logExtraApiCalls(apiPath)` after `cy.mockApi` to log requests and responses that are not stubbed by `cy.mockApi`.  This is useful for capturing responses while building out your mock files.

### `cy.failExtraApiCalls`

Call `cy.failExtraApiCalls(apiPath)` after `cy.mockApi` to log and fail on requests that are not stubbed by `cy.mockApi`.

## Fixture files

Fixture files (located within `mocksFolder`) in general should be named `{method}.{ext}` (for now only `json` and `txt` are supported).  The API path will be generated from the path as described in the example above.

### Wildcard slugs

To match against a route with a wildcard create a directory named or containing __ in place of the wildcard.  For example, to mock the response of a GET request to `/user/*/profile`, create fixtures `user/__/profile/get.json`.

### `option` files

For additional flexibility create `options.json` files within the `apiPath`.  These files are read by `cy.mockApi` and passed to `cy.route`.  For example, the following file:

```json
[
  {
    "url": "/api/user?userid=*",
    "response": "fx:mocks/a/query/delete-user.json",
    "method": "DELETE",
    "alias": "deleteUser",
  }
]
```

will setup file following route and stubbing:

```js
cy.route({
  url: '/api/user?userid=*',
  response: 'fx:mocks/a/query/delete-user.json',
  method: 'DELETE',
}).as('deleteUser');
```

See the (Cypress Documentation)[https://docs.cypress.io/api/commands/route.html#Arguments] for more details on the options available.

## Credits
