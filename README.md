e# Svelte Preprocess

> A [Svelte](https://svelte.technology) preprocessor wrapper with support for: PostCSS, SCSS, Less, Stylus, Coffeescript, TypeScript and Pug.

## Installation

`npm install --save-dev svelte-preprocess`

The preprocessor module installation is up to the developer.

- `postcss`: `npm install --save-dev postcss`
- `coffeescript`: `npm install --save-dev coffeescript`
- `typescript`: `npm install --save-dev typescript`
- `less`: `npm install --save-dev less`
- `sass`: `npm install --save-dev node-sass`
- `pug`: `npm install --save-dev pug`
- `stylus`: `npm install --save-dev stylus`

## Features

### Template tag support

_Note: only for auto preprocessing_

```html
<template>
  <div>Hey</div>
</template>

<style></style>

<script></script>
```

### External files support

_Note: only for auto preprocessing_

```html
<template src="template.html"></template>
<script src="./script.js"></script>
<style src="./style.css"></style>
```

### Preprocessors support

Current supported out-of-the-box preprocessors are `SCSS`, `Stylus`, `Less`, `Coffeescript`, `TypeScript`, `Pug` and `PostCSS`.

```html
<template lang="pug">
  div Hey
</template>

<script type="text/coffeescript">
  export default
    methods:
      foo: () ->
        console.log('Hey')
</script>

<script lang="typescript">
  export const hello: string = 'world';
</script>

<style src="./style.scss"></style>

<!-- Or -->

<style src="./style.styl"></style>

<!-- Or -->

<style lang="scss">
  $color: red;
  div {
    color: $color;
  }
</style>

<!-- Or -->

<style type="text/stylus">
  $color = red

  div
    color: $color;
</style>
```

## Usage

### Auto Preprocessing

In auto preprocessing mode, `svelte-preprocess` automatically use the respective preprocessor for your code based on your `type="..."` or `lang="..."` attributes.

#### Basic

```js
const svelte = require('svelte')
const preprocess = require('svelte-preprocess')

svelte.preprocess(input, preprocess({ /* options */ })).then(...)
```

#### Advanced

```js
const svelte = require('svelte')
const preprocess = require('svelte-preprocess')
const options = {
  /**
   * Extend the default language alias dictionary.
   * Each entry must follow: ['alias', 'languageName']
   */
  aliases: [
    /**
     * Means
     * <... src="file.cst"> or
     * <... lang="cst"> or
     * <... type="text/customLanguage">
     * <... type="application/customLanguage">
     * will be treated as the language 'customLanguage'
    */
    ['cst', 'customLanguage']
  ],

  preserve: [
    /**
     * Using the same matching algorithm as above, don't parse,
     * modify, or remove from the markup, tags which match the
     * language / types listed below.
     * **/
    'ld+json'
  ],

  /** Disable a language by setting it to 'false' */
  scss: false,

  /**  Pass options to the default preprocessor method */
  stylus: {
    paths: ['node_modules']
  },

  /**
   * Post process css with PostCSS by defining 'transformers.postcss' property,
   * either pass 'true' to activate PostCSS transforms and use the `postcss.config.js`
   */
  postcss: true,

  /** or pass an object with postcss plugins and their options directly. */
  postcss: {
    plugins: [
      require('autoprefixer')({ browsers: 'last 2 versions' })
    ]
  },

  typescript: {
    /**
     * Optionally specify the directory to load the tsconfig from
     */
    tsconfigDirectory: './configs',

    /**
     * Optionally specify the full path to the tsconfig
     */
    tsconfigFile: './tsconfig.app.json',

    /**
     * Optionally specify compiler options.
     * These will be merged with options from the tsconfig if found.
     */
    compilerOptions: {
      module: 'es2015'
    }
  },

  /** Use a custom preprocess method by passing a function. */
  pug({ content, filename }) {
      const code = pug.render(content)
      return { code, map: null }
  },

  /** Add a custom language preprocessor */
  customLanguage({ content, filename }) {
    const { code, map } = require('custom-language-compiler')(content)
    return { code, map }
  },
}

svelte.preprocess(input, preprocess(options)).then(...)
```

### Standalone processors

Instead of a single processor, [Svelte v3 has added support for multiple processors](https://v3.svelte.technology/docs#svelte-preprocess). In case you want to manually configure your preprocessing step, `svelte-preprocess` exports these named processors: `pug`, `coffeescript` or `coffee`, `less`, `scss` or `sass`, `stylus`, `postcss`.

```js
import { scss, coffeescript, pug } from 'svelte-preprocess'

svelte.preprocess(input, [
  pug(),
  coffeescript(),
  scss(),
]).then(...)
```

Every processor accepts an option object which is passed to its respective underlying tool.

```js
import { scss, postcss } from 'svelte-preprocess'

svelte.preprocess(input, [
  scss(),
  postcss({
    plugins: [require('autoprefixer')({ browsers: 'last 2 versions' })],
  }),
])
```

**Note**: there's no built-in support for \<template\> tag or external files when using standalone processors.

### With `rollup-plugin-svelte`

```js
// rollup.config.js
import svelte from 'rollup-plugin-svelte';
import autoPreprocess from 'svelte-preprocess'
import { scss, coffeescript, pug } from 'svelte-preprocess'

export default {
  ...,
  plugins: [
    svelte({
      /**
       * Auto preprocess supported languages with
       * '<template>'/'external src files' support
       **/
      preprocess: autoPreprocess({ /* options */ })
      /**
       * It is also possible to manually enqueue
       * stand-alone processors
       * */
      preprocess: [
        pug({ /* pug options */ }),
        scss({ /* scss options */ }),
        coffeescript({ /* coffeescript options */ })
      ]
    })
  ]
}
```

### With `svelte-loader`

```js
  ...
  module: {
    rules: [
      ...
      {
        test: /\.(html|svelte)$/,
        exclude: /node_modules/,
        use: {
          loader: 'svelte-loader',
          options: {
            preprocess: require('svelte-preprocess')({ /* options */ })
          },
        },
      },
      ...
    ]
  }
  ...
```

### With [Sapper](https://sapper.svelte.dev/)

Sapper has two build configurations, one for the client bundle and one for the server. To use `svelte-preprocess` with Sapper, you need to define it on both configurations.

```js
// ...
import sveltePreprocess from 'svelte-preprocess';

const preprocess = sveltePreprocess({
  postcss: true,
  // ...
};

export default {
  client: {
    plugins: [
      svelte({
        preprocess,
        // ...
      }),
  },
  server: {
    plugins: [
      svelte({
        preprocess,
        // ...
      }),
    ],
  },
};
```

### Limitations

#### `typescript`

Since `typescript` is not officially supported by `svelte` for its template language, `svelte-preprocess` only type checks code between the `<script></script>` tag.

#### `pug`

Some of Svelte's template syntax is invalid in `pug`. `svelte-preprocess` provides some pug mixins to represent svelte's `{#...}{/...}` blocks: `+if()`, `+else()`, `+elseif()`, `+each()`, `+await()`, `+then()`, `+catch()`, `+debug()`.

```pug
ul
  +if('posts && posts.length > 1')
    +each('posts as post')
      li
        a(rel="prefetch" href="blog/{post.slug}") {post.title}
    +else()
      span No posts :c
```
