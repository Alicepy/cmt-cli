#cmt-cli

A simple CLI for scaffolding Angular.js projects.

### Installation

Prerequisites: [Node.js](https://nodejs.org/en/) (>=6.x, 8.x preferred), npm version 3+ and [Git](https://git-scm.com/).

$ npm install -g cmt-cli
$ cmt init <template-name> <project-name>

cmt init username/repo my-project


### Local Templates

``` bash
vue init ~/fs/path/to-custom-template my-project
```

### Writing Custom Templates from Scratch

- A template repo **must** have a `template` directory that holds the template files.

- A template repo **may** have a metadata file for the template which can be either a `meta.js` or `meta.json` file. It can contain the following fields:

  - `prompts`: used to collect user options data;

  - `filters`: used to conditional filter files to render.
  
  - `metalsmith`: used to add custom metalsmith plugins in the chain.

  - `completeMessage`: the message to be displayed to the user when the template has been generated. You can include custom instruction here.

  - `complete`: Instead of using `completeMessage`, you can use a function to run stuffs when the template has been generated.

#### prompts

The `prompts` field in the metadata file should be an object hash containing prompts for the user. For each entry, the key is the variable name and the value is an [Inquirer.js question object](https://github.com/SBoudrias/Inquirer.js/#question). Example:

``` json
{
  "prompts": {
    "name": {
      "type": "string",
      "required": true,
      "message": "Project name"
    }
  }
}
```

After all prompts are finished, all files inside `template` will be rendered using [Handlebars](http://handlebarsjs.com/), with the prompt results as the data.

##### Conditional Prompts

A prompt can be made conditional by adding a `when` field, which should be a JavaScript expression evaluated with data collected from previous prompts. For example:

``` json
{
  "prompts": {
    "lint": {
      "type": "confirm",
      "message": "Use a linter?"
    },
    "lintConfig": {
      "when": "lint",
      "type": "list",
      "message": "Pick a lint config",
      "choices": [
        "standard",
        "airbnb",
        "none"
      ]
    }
  }
}
```

The prompt for `lintConfig` will only be triggered when the user answered yes to the `lint` prompt.

##### Pre-registered Handlebars Helpers

Two commonly used Handlebars helpers, `if_eq` and `unless_eq` are pre-registered:

``` handlebars
{{#if_eq lintConfig "airbnb"}};{{/if_eq}}
```

##### Custom Handlebars Helpers

You may want to register additional Handlebars helpers using the `helpers` property in the metadata file. The object key is the helper name:

``` js
module.exports = {
  helpers: {
    lowercase: str => str.toLowerCase()
  }
}
```

Upon registration, they can be used as follows:

``` handlebars
{{ lowercase name }}
```

#### File filters

The `filters` field in the metadata file should be an object hash containing file filtering rules. For each entry, the key is a [minimatch glob pattern](https://github.com/isaacs/minimatch) and the value is a JavaScript expression evaluated in the context of prompt answers data. Example:

``` json
{
  "filters": {
    "test/**/*": "needTests"
  }
}
```

Files under `test` will only be generated if the user answered yes to the prompt for `needTests`.

Note that the `dot` option for minimatch is set to `true` so glob patterns would also match dotfiles by default.

#### Skip rendering

The `skipInterpolation` field in the metadata file should be a [minimatch glob pattern](https://github.com/isaacs/minimatch). The files matched should skip rendering. Example:

``` json
{
  "skipInterpolation": "src/**/*.vue"
}
```

#### Metalsmith

`vue-cli` uses [metalsmith](https://github.com/segmentio/metalsmith) to generate the project.

You may customize the metalsmith builder created by vue-cli to register custom plugins.

```js
{
  "metalsmith": function (metalsmith, opts, helpers) {
    function customMetalsmithPlugin (files, metalsmith, done) {
      // Implement something really custom here.
      done(null, files)
    }
    
    metalsmith.use(customMetalsmithPlugin)
  }
}
```

If you need to hook metalsmith before questions are asked, you may use an object with `before` key.

```js
{
  "metalsmith": {
    before: function (metalsmith, opts, helpers) {},
    after: function (metalsmith, opts, helpers) {}
  }
}
```

#### Additional data available in meta.{js,json}

- `destDirName` - destination directory name

```json
{
  "completeMessage": "To get started:\n\n  cd {{destDirName}}\n  npm install\n  npm run dev"
}
```

- `inPlace` - generating template into current directory

```json
{
  "completeMessage": "{{#inPlace}}To get started:\n\n  npm install\n  npm run dev.{{else}}To get started:\n\n  cd {{destDirName}}\n  npm install\n  npm run dev.{{/inPlace}}"
}
```

### `complete` function

Arguments:

- `data`: the same data you can access in `completeMessage`:
  ```js
  {
    complete (data) {
      if (!data.inPlace) {
        console.log(`cd ${data.destDirName}`)
      }
    }
  }
  ```

- `helpers`: some helpers you can use to log results.
  - `chalk`: the `chalk` module
  - `logger`: [the built-in vue-cli logger](/lib/logger.js)
  - `files`: An array of generated files
  ```js
  {
    complete (data, {logger, chalk}) {
      if (!data.inPlace) {
        logger.log(`cd ${chalk.yellow(data.destDirName)}`)
      }
    }
  }
  ```

### Installing a specific template version
```
vue init '<template-name>#<branch-name>' <project-name>
```
## Thanks
To  [vue-cli](https://github.com/vuejs/vue-cli) for the head start.

