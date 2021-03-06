# Hooked-Head

[![npm version](https://badgen.net/npm/v/hooked-head)](https://www.npmjs.com/package/hooked-head)
[![Bundle size](https://badgen.net/bundlephobia/minzip/hooked-head)](https://badgen.net/bundlephobia/minzip/hooked-head)
[![codecov](https://codecov.io/gh/JoviDeCroock/hooked-head/branch/master/graph/badge.svg)](https://codecov.io/gh/JoviDeCroock/hooked-head)

This project aims at providing a set of hooks to populate `<meta>`, ... for each page. With crawlers now supporting
client-side alterations it's important to support a fallback model for our `<head>` tags. The dispatcher located in this
library will always make a queue of how we should fallback, ... This way we'll always have some information to give to a
visiting crawler.

```sh
npm i --save hooked-head
## OR
yarn add hooked-head
```

```jsx
import {
  useMeta,
  useLink,
  useLang,
  useTitle,
  useTitleTemplate,
} from 'hooked-head';

const App = () => {
  // Will set <html lang="en">
  useLang('en');

  // Will set title to "Welcome to hooked-head | 💭"
  useTitleTemplate('%s | 💭');
  useTitle('Welcome to hooked-head');

  useMeta({ name: 'author', content: 'Jovi De Croock' });
  useLink({ rel: 'me', href: 'https://jovidecroock.com' });

  return <p>Hooked-Head</p>;
};
```

## Preact

If you need support for [Preact](https://preactjs.com/) you can import from `hooked-head/preact` instead.

## Hooks

This package exports `useTitle`, `useTitleTemplate`, `useMeta`, `useLink` and `useLang`. These hooks
are used to control information conveyed by the `<head>` in an html document.

### useTitle

This hook accepts a string that will be used to set the `document.title`, every time the
given string changes it will update the property.

### useTitleTemplate

This hook accepts a string, which will be used to format the result of `useTitle` whenever
it updates. Similar to react-helmet, the placeholder `%s` will be replaced with the `title`.

### useMeta

This hook accepts the regular `<meta>` properties, being `name`, `property`, `httpEquiv`,
`charset` and `content`.

These have to be passed as an object and will update when `content` changes.

### useLink

This hook accepts the regular `<link>` properties, being `rel`, `as`, `media`,
`href`, `sizes` and `crossorigin`.

This will update within the same `useLink` but will never go outside

### useLang

This hook accepts a string that will be used to set the `lang` property on the
base `<html>` tag. Every time this string gets updated this will be reflected in the dom.

## SSR

We expose a method called `toStatic` that will return the following properties:

- title, the current `title` dictated by the deepest `useTitleTemplate` and `useTitle` combination
- lang, the current `lang` dictated by the deepest `useLang`
- metas, an array of unique metas by `keyword` (property, ...)
- links, the links aggregated from the render pass.

The reason we pass these as properties is to better support `gatsby`, ...

If you need to stringify these you can use the following algo:

```js
const stringify = (title, metas, links) => {
  const visited = new Set();
  return `
    <title>${title}</title>

    ${metaQueue.reduce((acc, meta) => {
      if (!visited.has(meta.charset ? meta.keyword : meta[meta.keyword])) {
        visited.add(meta.charset ? meta.keyword : meta[meta.keyword]);
        return `${acc}<meta ${meta.keyword}="${meta[meta.keyword]}"${
          meta.charset ? '' : ` content="${meta.content}"`
        }>`;
      }
      return acc;
    }, '')}

    ${linkQueue.reduce((acc, link) => {
      return `${acc}<link${Object.keys(link).reduce(
        (properties, key) => `${properties} ${key}="${link[key]}"`,
        ''
      )}>`;
    }, '')}
  `;
};
```

```js
import { toStatic } from 'hooked-head';

const reactStuff = renderToString();
const { metas, links, title, lang } = toStatic();
const stringified = stringify(title, metas, links);

const html = `
  <!doctype html>
    <html lang="${lang}">
      <head>
        ${stringified}
      </head>
      <body>
        <div id="content">
          ${reactStuff}
        </div>
      </body>
  </html>
`;
```

## Goals

- [x] React support
- [x] Add majority of types for meta and link.
- [x] Concurrent friendly
- [x] Preact support
- [x] Support `<link>`
- [x] Stricter typings
- [x] Document the hooks
- [x] Document the dispatcher
- [x] SSR support
- [x] Consider moving from `doc.title = x` to inserting `<title>x</title>`
- [x] Golf bytes
- [ ] improve typings, there are probably missing possibilities in `types.ts`
