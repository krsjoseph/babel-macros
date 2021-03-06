# `babel-macros` Usage for macros authors

> See also: [the `user` docs](https://github.com/kentcdodds/babel-macros/blob/master/other/docs/user.md).

## Writing a macro

A macro is a JavaScript module that exports a function. It can be published to
the npm registry (for generic macros, like a css-in-js library) or used locally
(for domain-specific macros, like handling some special case for your company's
localization efforts).

> Before you write a custom macro, you might consider whether
> [`babel-plugin-preval`][preval] help you do what you want as it's pretty
> powerful.

There are two parts to the `babel-macros` API:

1. The filename convention
2. The function you export

**Filename**:

The way that `babel-macros` determines whether to run a macro is based on the
source string of the `import` or `require` statement. It must match this regex:
`/[./]macro(\.js)?$/` for example:

_matches_:

```
'my.macro'
'my.macro.js'
'my/macro'
'my/macro.js'
```

_does not match_:

```
'my-macro'
'my.macro.is-sweet'
'my/macro/rocks'
```

> So long as your file can be required at a matching path, you're good. So you
> could put it in: `my/macro/index.js` and people would: `require('my/macro')`
> which would work fine.

**If you're going to publish this to npm,** the most ergonomic thing would be to
name it something that ends in `.macro`. If it's part of a larger package,
then calling the file `macro.js` or placing it in `macro/index.js` is a great
way to go as well. Then people could do:

```js
import Nice from 'nice.macro'
// or
import Sweet from 'sweet/macro'
```

**Function API**:

The macro you create should export a function. That function accepts a single
parameter which is an object with the following properties:

**state**: The state of the file being traversed. It's the second argument
you receive in a visitor function in a normal babel plugin.

**references**: This is an object that contains arrays of all the references to
things imported from macro keyed based on the name of the import. The items
in each array are the paths to the references.

<details>

<summary>Some examples:</summary>

```javascript
import MyMacro from './my.macro'

MyMacro({someOption: true}, `
  some stuff
`)

// references: { default: [BabelPath] }
```

```javascript
import {foo as FooMacro} from './my.macro'

FooMacro({someOption: true}, `
  some stuff
`)

// references: { foo: [BabelPath] }
```

```javascript
import {foo as FooMacro} from './my.macro'

// no usage...

// references: {}
```

</details>

From here, it's just a matter of doing doing stuff with the `BabelPath`s that
you're given. For that check out [the babel handbook][babel-handbook].

> One other thing to note is that after your macro has run, babel-macros will
> remove the import/require statement for you.


## Testing your macro

The best way to test your macro is using [`babel-plugin-tester`][tester]:

```javascript
import path from 'path'
import pluginTester from 'babel-plugin-tester'
import plugin from 'babel-macros'

pluginTester({
  plugin,
  snapshot: true,
  tests: withFilename([
    `
      import MyMacro from '../my.macro'

      MyMacro({someOption: true}, \`
        some stuff
      \`)
    `
  ]),
})

/*
 * This adds the filename to each test so you can do require/import relative
 * to this test file.
 */
function withFilename(tests) {
  return tests.map(t => {
    const test = {babelOptions: {filename: __filename}}
    if (typeof t === 'string') {
      test.code = t
    } else {
      Object.assign(test, t)
    }
    return test
  })
}
```

There is currently no way to get code coverage for your macro this way however.
If you want code coverage, you'll have to call your macro yourself.
Contributions to improve this experience are definitely welcome!

[preval]: https://github.com/kentcdodds/babel-plugin-preval
[babel-handbook]: https://github.com/thejameskyle/babel-handbook/blob/master/translations/en/plugin-handbook.md
[tester]: https://github.com/babel-utils/babel-plugin-tester
