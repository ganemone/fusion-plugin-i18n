# fusion-plugin-i18n

[![Build status](https://badge.buildkite.com/3f2d84d5538d87a19677f5d79304ac46a8a67f970520d13884.svg?branch=master)](https://buildkite.com/uberopensource/fusion-plugin-i18n)

Adds I18n (Internationalization) support to a FusionJS app

For date i18n, consider using [date-fns](https://date-fns.org/)

If you're using React, you should use [`fusion-plugin-i18n-react`](https://github.com/fusionjs/fusion-plugin-i18n-react) instead of this package.

---

### Installation

```sh
yarn add fusion-plugin-i18n
```

---

### Example

```js
// src/main.js
import React from 'react';
import App from 'fusion-react';
import I18n, {I18nToken, I18nLoaderToken, createI18nLoader} from 'fusion-plugin-i18n';
import {FetchToken} from 'fusion-tokens';
import fetch from 'unfetch';
import Hello from './hello';

export default () => {
  const app = new App(<div></div>);

  app.register(I18nToken, I18n);
  __NODE__
    ? app.configure(I18nLoaderToken, createI18nLoader());
    : app.configure(FetchToken, fetch);

  app.register(Hello);

  return app;
}

// src/hello.js
import {I18nToken} from 'fusion-plugin-i18n';

export default withDependencies({I18n: I18nToken}, ({I18n}) => {
  return withMiddleware((ctx, next) => {
    if (__NODE__ && ctx.path === '/hello') {
      const i18n = I18n(ctx);
      ctx.body = {
        message: i18n.translate('test', {name: 'world'}), // hello world
      }
    }
    return next();
  });
});

// translations/en-US.json
{
  "test": "hello ${name}"
}
```

##### Custom translations loader example

```js
// src/main.js
import React from 'react';
import App from 'fusion-react';
import I18n, {I18nToken, I18nLoaderToken} from 'fusion-plugin-i18n';
import {FetchToken} from 'fusion-tokens';
import fetch from 'unfetch';
import Hello from './hello';
import I18nLoader from './translations';

export default () => {
  const app = new App(<div></div>);

  app.register(I18nToken, I18n);
  __NODE__
    ? app.configure(I18nLoaderToken, I18nLoader);
    : app.configure(FetchToken, fetch);

  app.register(Hello);

  return app;
}

// src/hello.js
import {I18nToken} from 'fusion-plugin-i18n';

export default withDependencies({I18n: I18nToken}, ({I18n}) => {
  return withMiddleware((ctx, next) => {
    if (__NODE__ && ctx.path === '/hello') {
      const i18n = I18n(ctx);
      ctx.body = {
        message: i18n.translate('test', {name: 'world'}), // hello world
      }
    }
    return next();
  });
});

// src/translation-loader.js
const translationData = {
  'en-US': {
    test: "hello ${name}"
  }
}

export default (ctx) => {
  // locale could be determined in different ways,
  // e.g. from ctx.headers['accept-language'] or from a /en-US/ URL
  const locale = 'en-US';
  const translations = translationData[locale];
  return {locale, translations};
}
```

---

### API

#### Dependency registration

```js
import I18n, {I18nToken, I18nLoaderToken} from 'fusion-plugin-i18n';
import {FetchToken} from 'fusion-tokens';

app.register(I18nToken, I18n);
__NODE__
  ? app.configure(I18nLoaderToken, I18nLoader);
  : app.configure(FetchToken, fetch);
```

- `I18n` - the core I18n library
- `I18nLoader: (ctx: Context) => ({locale: string, translations: Object<string, string>})` - A function that provides translations
  - `ctx: {headers: {'accept-language': string}}` - A Koa context object
- `fetch` - a [`fetch`](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) implementation

#### Factory

`I18n(ctx)`

- `ctx: {headers: {'accept-language': string}}` - a Koa context object

#### Instance methods

- `translate: (key: string, interpolations: Object) => string`

#### Server-side loader

This plugin has a simple loader implementation that looks for files in a `./translations` directory. Files should be named after their locales.

```js
import {I18nLoaderToken, createI18nLoader} from 'fusion-plugin-i18n';

app.configure(I18nLoaderToken, createI18nLoader());

// translations/en-US.json
{
  "some-translation-key": "hello",
}
```

