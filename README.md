# article-parser

Extract main article, main image and meta data from URL.

[![NPM](https://badge.fury.io/js/article-parser.svg)](https://badge.fury.io/js/article-parser)
![CI test](https://github.com/ndaidong/article-parser/workflows/ci-test/badge.svg)
[![Coverage Status](https://coveralls.io/repos/github/ndaidong/article-parser/badge.svg)](https://coveralls.io/github/ndaidong/article-parser)
![CodeQL](https://github.com/ndaidong/article-parser/workflows/CodeQL/badge.svg)
[![JavaScript Style Guide](https://img.shields.io/badge/code_style-standard-brightgreen.svg)](https://standardjs.com)


## Intro

*article-parser* is a part of tool sets for content builder:

- [feed-reader](https://github.com/ndaidong/feed-reader): extract & normalize RSS/ATOM/JSON feed
- [article-parser](https://github.com/ndaidong/article-parser): extract main article from given URL
- [oembed-parser](https://github.com/ndaidong/oembed-parser): extract oEmbed data from supported providers

You can use one or combination of these tools to build news sites, create automated content systems for marketing campaign or gather dataset for NLP projects...

```
                                    ┌────────────────┐
                            ┌───────► article-parser ├──────────┐
                            │       └────────────────┘          │
┌─────────────┐   ┌─────────┴────┐                     ┌────────▼─────────┐   ┌─────────────┐
│ feed-reader ├───► feed entries │                     │ content database ├───► public APIs │
└─────────────┘   └─────────┬────┘                     └────────▲─────────┘   └─────────────┘
                            │       ┌────────────────┐          │
                            └───────► oembed-parser  ├──────────┘
                                    └────────────────┘
```

## Demo

- [Give it a try!](https://demos.pwshub.com/article-parser)
- [Example FaaS](https://extract-article.deta.dev/?url=https://www.freethink.com/technology/virtual-world)


## Install & Usage

### Node.js

```bash
npm i article-parser

# pnpm
pnpm i article-parser

# yarn
yarn add article-parser
```

```ts
// es6 module
import { extract } from 'article-parser'

// CommonJS
const { extract } = require('article-parser')

// or specify exactly path to CommonJS variant
const { extract } = require('article-parser/dist/cjs/article-parser.js')
```

### Deno

```ts
import { extract } from 'https://esm.sh/article-parser'
```

### Browser

```ts
import { extract } from 'https://unpkg.com/article-parser@latest/dist/article-parser.esm.js'
```

Please check [the examples](https://github.com/ndaidong/article-parser/tree/main/examples) for reference.


### Deta cloud

For [Deta](https://www.deta.sh/) devs please refer [the source code and guideline here](https://github.com/ndaidong/article-parser-deta) or simply click the button below.

[![Deploy](https://button.deta.dev/1/svg)](https://go.deta.dev/deploy?repo=https://github.com/ndaidong/article-parser-deta)


## APIs

- [extract()](#extract)
- [Transformations](#transformations)
  - [`transformation` object](#transformation-object)
  - [.addTransformations](#addtransformationsobject-transformation--array-transformations)
  - [.removeTransformations](#removetransformationsarray-patterns)
  - [Priority order](#priority-order)
- [`sanitize-html`'s options](#sanitize-htmls-options)

---

### `extract()`

Load and extract article data. Return a Promise object.

#### Syntax

```ts
extract(String input)
extract(String input, Object parserOptions)
extract(String input, Object parserOptions, Object fetchOptions)
```

#### Parameters

##### `input` *required*

URL string links to the article or HTML content of that web page.

For example:

```js
import { extract } from 'article-parser'

const input = 'https://www.cnbc.com/2022/09/21/what-another-major-rate-hike-by-the-federal-reserve-means-to-you.html'
extract(input)
  .then(article => console.log(article))
  .catch(err => console.error(err))
```

The result - `article` - can be `null` or an object with the following structure:

```ts
{
  url: String,
  title: String,
  description: String,
  image: String,
  author: String,
  content: String,
  published: Date String,
  source: String, // original publisher
  links: Array, // list of alternative links
  ttr: Number, // time to read in second, 0 = unknown
}
```

[Click here](https://extract-article.deta.dev/?url=https://www.freethink.com/technology/virtual-world) for seeing an actual result.


##### `parserOptions` *optional*

Object with all or several of the following properties:

  - `wordsPerMinute`: Number, to estimate time to read. Default `300`.
  - `descriptionTruncateLen`: Number, max num of chars generated for description. Default `210`.
  - `descriptionLengthThreshold`: Number, min num of chars required for description. Default `180`.
  - `contentLengthThreshold`: Number, min num of chars required for content. Default `200`.

For example:

```js
import { extract } from 'article-parser'

extract('https://www.cnbc.com/2022/09/21/what-another-major-rate-hike-by-the-federal-reserve-means-to-you.html', {
  descriptionLengthThreshold: 120,
  contentLengthThreshold: 500
})
```

##### `fetchOptions` *optional*

You can use this param to set request headers to [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch).

For example:

```js
import { extract } from 'article-parser'

const url = 'https://www.cnbc.com/2022/09/21/what-another-major-rate-hike-by-the-federal-reserve-means-to-you.html'
extract(url, null, {
  headers: {
    'user-agent': 'Opera/9.60 (Windows NT 6.0; U; en) Presto/2.1.1'
  }
})
```

You can also specify a proxy endpoint to load remote content, instead of fetching directly.

For example:

```js
import { extract } from 'article-parser'

const url = 'https://www.cnbc.com/2022/09/21/what-another-major-rate-hike-by-the-federal-reserve-means-to-you.html'

extract(url, null, {
  headers: {
    'user-agent': 'Opera/9.60 (Windows NT 6.0; U; en) Presto/2.1.1'
  },
  proxy: {
    target: 'https://your-secret-proxy.io/loadXml?url=',
    headers: {
      'Proxy-Authorization': 'Bearer YWxhZGRpbjpvcGVuc2VzYW1l...'
    }
  }
})
```

Passing requests to proxy is useful while running `article-parser` on browser. View [examples/browser-article-parser](https://github.com/ndaidong/article-parser/tree/main/examples/browser-article-parser) as reference example.

For more info about proxy authentication, please refer [HTTP authentication](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication)

For a deeper customization, you can consider using [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) to replace `fetch` behaviors with your own handlers.

---

### Transformations

Sometimes the default extraction algorithm may not work well. That is the time when we need transformations.

By adding some functions before and after the main extraction step, we aim to come up with a better result as much as possible.

There are 2 methods to play with transformations:

- `addTransformations(Object transformation | Array transformations)`
- `removeTransformations(Array patterns)`

At first, let's talk about `transformation` object.

#### `transformation` object

In `article-parser`, `transformation` is an object with the following properties:

- `patterns`: required, a list of regexps to match the URLs
- `pre`: optional, a function to process raw HTML
- `post`: optional, a function to process extracted article

Basically, the meaning of `transformation` can be interpreted like this:

> with the urls which match these `patterns` <br>
> let's run `pre` function to normalize HTML content <br>
> then extract main article content with normalized HTML, and if success <br>
> let's run `post` function to normalize extracted article content

![article-parser extraction process](https://res.cloudinary.com/pwshub/image/upload/v1657336822/documentation/article-parser_extraction_process.png)

Here is an example transformation:

```js
{
  patterns: [
    /([\w]+.)?domain.tld\/*/,
    /domain.tld\/articles\/*/
  ],
  pre: (document) => {
    // remove all .advertise-area and its siblings from raw HTML content
    document.querySelectorAll('.advertise-area').forEach((element) => {
      if (element.nodeName === 'DIV') {
        while (element.nextSibling) {
          element.parentNode.removeChild(element.nextSibling)
        }
        element.parentNode.removeChild(element)
      }
    })
    return document
  },
  post: (document) => {
    // with extracted article, replace all h4 tags with h2
    document.querySelectorAll('h4').forEach((element) => {
      const h2Element = document.createElement('h2')
      h2Element.innerHTML = element.innerHTML
      element.parentNode.replaceChild(h2Element, element)
    })
    // change small sized images to original version
    document.querySelectorAll('img').forEach((element) => {
      const src = element.getAttribute('src')
      if (src.includes('domain.tld/pics/150x120/')) {
        const fullSrc = src.replace('/pics/150x120/', '/pics/original/')
        element.setAttribute('src', fullSrc)
      }
    })
    return document
  }
}
```

- To write better transformation logic, please refer [linkedom](https://github.com/WebReflection/linkedom) and [Document Object](https://developer.mozilla.org/en-US/docs/Web/API/Document).

#### `addTransformations(Object transformation | Array transformations)`

Add a single transformation or a list of transformations. For example:

```js
import { addTransformations } from 'article-parser'

addTransformations({
  patterns: [
    /([\w]+.)?abc.tld\/*/
  ],
  pre: (document) => {
    // do something with document
    return document
  },
  post: (document) => {
    // do something with document
    return document
  }
})

addTransformations([
  {
    patterns: [
      /([\w]+.)?def.tld\/*/
    ],
    pre: (document) => {
      // do something with document
      return document
    },
    post: (document) => {
      // do something with document
      return document
    }
  },
  {
    patterns: [
      /([\w]+.)?xyz.tld\/*/
    ],
    pre: (document) => {
      // do something with document
      return document
    },
    post: (document) => {
      // do something with document
      return document
    }
  }
])
````

The transformations without `patterns` will be ignored.

#### `removeTransformations(Array patterns)`

To remove transformations that match the specific patterns.

For example, we can remove all added transformations above:

```js
import { removeTransformations } from 'article-parser'

removeTransformations([
  /([\w]+.)?abc.tld\/*/,
  /([\w]+.)?def.tld\/*/,
  /([\w]+.)?xyz.tld\/*/
])
```

Calling `removeTransformations()` without parameter will remove all current transformations.

#### Priority order

While processing an article, more than one transformation can be applied.

Suppose that we have the following transformations:

```ts
[
  {
    patterns: [
      /http(s?):\/\/google.com\/*/,
      /http(s?):\/\/goo.gl\/*/
    ],
    pre: function_one,
    post: function_two
  },
  {
    patterns: [
      /http(s?):\/\/goo.gl\/*/,
      /http(s?):\/\/google.inc\/*/
    ],
    pre: function_three,
    post: function_four
  }
]
```

As you can see, an article from `goo.gl` certainly matches both them.

In this scenario, `article-parser` will execute both transformations, one by one:

`function_one` -> `function_three` -> extraction -> `function_two` -> `function_four`

---

### `sanitize-html`'s options

`article-parser` uses [sanitize-html](https://www.npmjs.com/package/sanitize-html) to make a clean sweep of HTML content.

Here is the [default options](https://github.com/ndaidong/article-parser/blob/main/src/config.js#L5)

Depending on the needs of your content system, you might want to gather some HTML tags/attributes, while ignoring others.

There are 2 methods to access and modify these options in `article-parser`.

- `getSanitizeHtmlOptions()`
- `setSanitizeHtmlOptions(Object sanitizeHtmlOptions)`

Read [sanitize-html](https://www.npmjs.com/package/sanitize-html#what-are-the-default-options) docs for more info.

---

## Quick evaluation

```bash
git clone https://github.com/ndaidong/article-parser.git
cd article-parser
pnpm i

npm run eval {URL_TO_PARSE_ARTICLE}
```

## License
The MIT License (MIT)

---
