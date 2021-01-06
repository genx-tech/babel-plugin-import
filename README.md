# babel-plugin-import

Modular import plugin for babel, compatible with [antd](https://github.com/ant-design/ant-design), [antd-mobile](https://github.com/ant-design/ant-design-mobile), lodash, [material-ui](http://material-ui.com/), and so on.


**PRIVATE BUILD** (forked from [ant-design repo](https://github.com/ant-design/babel-plugin-import))


## Installation
```
npm i -D https://github.com/genx-tech/babel-plugin-import.git
```

## New features added

- [Library prefix match](https://github.com/genx-tech/babel-plugin-import#prefixmatch)

----

## Example

#### `{ "libraryName": "antd" }`

```javascript
import { Button } from 'antd';
ReactDOM.render(<Button>xxxx</Button>);

      ↓ ↓ ↓ ↓ ↓ ↓

var _button = require('antd/lib/button');
ReactDOM.render(<_button>xxxx</_button>);
```

#### `{ "libraryName": "antd", style: "css" }`

```javascript
import { Button } from 'antd';
ReactDOM.render(<Button>xxxx</Button>);

      ↓ ↓ ↓ ↓ ↓ ↓

var _button = require('antd/lib/button');
require('antd/lib/button/style/css');
ReactDOM.render(<_button>xxxx</_button>);
```

#### `{ "libraryName": "antd", style: true }`

```javascript
import { Button } from 'antd';
ReactDOM.render(<Button>xxxx</Button>);

      ↓ ↓ ↓ ↓ ↓ ↓

var _button = require('antd/lib/button');
require('antd/lib/button/style');
ReactDOM.render(<_button>xxxx</_button>);
```

Note : with `style: true` css source files are imported and optimizations can be done during compilation time. With `style: "css"`, pre bundled css files are imported as they are.

`style: true` can reduce the bundle size significantly, depending on your usage of the library.

## Usage

```bash
npm install babel-plugin-import --save-dev
```

Via `.babelrc` or babel-loader.

```js
{
  "plugins": [["import", options]]
}
```

### options

`options` can be object.

```javascript
{
  "libraryName": "antd",
  "style": true,   // or 'css'
}
```

```javascript
{
  "libraryName": "lodash",
  "libraryDirectory": "",
  "camel2DashComponentName": false,  // default: true
}
```

```javascript
{
  "libraryName": "@material-ui/core",
  "libraryDirectory": "components",  // default: lib
  "camel2DashComponentName": false,  // default: true
}
```

~`options` can be an array.~ It's not available in babel@7+

For Example:

```javascript
[
  {
    "libraryName": "antd",
    "libraryDirectory": "lib",   // default: lib
    "style": true
  },
  {
    "libraryName": "antd-mobile"
  },
]
```
`Options` can't be an array in babel@7+, but you can add plugins with name to support multiple dependencies.

For Example:

```javascrit
// .babelrc
"plugins": [
  ["import", { "libraryName": "antd", "libraryDirectory": "lib"}, "antd"],
  ["import", { "libraryName": "antd-mobile", "libraryDirectory": "lib"}, "antd-mobile"]
]
```

#### prefixMatch

When `prefixMatch` is enabled (`"prefixMatch": true`), this plugin will also transpile libraries starting with with the `libraryName` as prefix.

For Example:
```javascrit
{
    libraryName: '@genx/react',
    prefixMatch: true, // will transpile '@genx/react/i18n' to '@genx/react/lib/commonjs/i18n' as well
    libraryDirectory: 'lib/commonjs',
    camel2DashComponentName: false
}

// The below import will be transpiled into import I18nProvider from '@genx/react/lib/commonjs/i18n/I18nProvider';
import { I18nProvider } from '@genx/react/i18n';
```

#### style

- `["import", { "libraryName": "antd" }]`: import js modularly
- `["import", { "libraryName": "antd", "style": true }]`: import js and css modularly (LESS/Sass source files)
- `["import", { "libraryName": "antd", "style": "css" }]`: import js and css modularly (css built files)

If option style is a `Function`, `babel-plugin-import` will auto import the file which filepath equal to the function return value. This is useful for the components library developers.

e.g.
- ``["import", { "libraryName": "antd", "style": (name) => `${name}/style/2x` }]``: import js and css modularly & css file path is `ComponentName/style/2x`

If a component has no style, you can use the `style` function to return a `false` and the style will be ignored.

e.g.
```js
[
  "import",
    {
      "libraryName": "antd",
      "style": (name: string, file: Object) => {
        if(name === 'antd/lib/utils'){
          return false;
        }
        return `${name}/style/2x`;
      }
    }
]
```

#### styleLibraryDirectory

- `["import", { "libraryName": "element-ui", "styleLibraryDirectory": "lib/theme-chalk" }]`: import js and css modularly

If `styleLibraryDirectory` is provided (default `null`), it will be used to form style file path,
`style` will be ignored then. e.g.

```javascript
{
  "libraryName": "element-ui",
  "styleLibraryDirectory": "lib/theme-chalk",
}

import { Button } from 'element-ui';

      ↓ ↓ ↓ ↓ ↓ ↓

var _button = require('element-ui/lib/button');
require('element-ui/lib/theme-chalk/button');
```

#### customName

We can use `customName` to customize import file path.

For example, the default behavior:

```typescript
import { TimePicker } from "antd"
↓ ↓ ↓ ↓ ↓ ↓
var _button = require('antd/lib/time-picker');
```

You can set `camel2DashComponentName` to `false` to disable transfer from camel to dash:

```typescript
import { TimePicker } from "antd"
↓ ↓ ↓ ↓ ↓ ↓
var _button = require('antd/lib/TimePicker');
```

And finally, you can use `customName` to customize each name parsing:

```js
[
  "import",
    {
      "libraryName": "antd",
      "customName": (name: string, file: object) => {
        const filename = file.opts.filename;
        if (name === 'TimePicker'){
          return 'antd/lib/custom-time-picker';
        }
        if (filename.indexOf('/path/to/my/different.js') >= 0) {
          return 'antd/lib/custom-name';
        }
        return `antd/lib/${name}`;
      }
    }
]
```

So this result is:

```typescript
import { TimePicker } from "antd"
↓ ↓ ↓ ↓ ↓ ↓
var _button = require('antd/lib/custom-time-picker');
```

In some cases, the transformer may serialize the configuration object. If we set the `customName` to a function, it will lost after the serialization.

So we also support specifying the customName with a JavaScript source file path:

```js
[
  "import",
    {
      "libraryName": "antd",
      "customName": require('path').resolve(__dirname, './customName.js')
    }
]
```

The `customName.js` looks like this:

```js
module.exports = function customName(name) {
  return `antd/lib/${name}`;
};
```

#### customStyleName

`customStyleName` works exactly the same as customName, except that it deals with style file path.

#### transformToDefaultImport

Set this option to `false` if your module does not have a `default` export.

#### memberUseNamedImport

Set this option to `true` to make member reference use named import instead of default import.

When prefixMatch is enabled, memberUseNamedImport will use `true` as default.

As a CommonJS module, objects with named export are not reside in the default exported object. 

Named export cannot be retrieved using destructure assignment or member access. See MDN export for details.  

```javascrit

/* with the config below
{
    libraryName: '@genx/react',
    prefixMatch: true, // will transpile '@genx/react/i18n' to '@genx/react/lib/commonjs/i18n' as well
    libraryDirectory: 'lib/commonjs',
    camel2DashComponentName: false,
    memberUseNamedImport: true
}
*/

import RuntimeLib from '@genx/react/Runtime'; // transpile to '@genx/react/lib/commonjs/Runtime'
import { Runtime } from '@genx/react'; // transpile to '@genx/react/lib/commonjs/Runtime'

RuntimeLib.updateRuntime(...); // will import { updateRuntime } from '@genx/react/lib/commonjs/Runtime'

// however,
Runtime.updateRuntime(...); // will not import { updateRuntime } from '@genx/react/lib/commonjs/Runtime'
// will directly call member method updateRuntime

```

### Note

babel-plugin-import will not work properly if you add the library to the webpack config [vendor](https://webpack.js.org/concepts/entry-points/#separate-app-and-vendor-entries).

## License

MIT