<img width="945" alt="2017-07-26 9 27 05" src="https://user-images.githubusercontent.com/8784712/28623641-373450f4-7249-11e7-854d-1b076dab274d.png">

[![NPM version](https://img.shields.io/npm/v/cac.svg?style=flat)](https://npmjs.com/package/cac) [![NPM downloads](https://img.shields.io/npm/dm/cac.svg?style=flat)](https://npmjs.com/package/cac) [![CircleCI](https://circleci.com/gh/cacjs/cac/tree/master.svg?style=shield)](https://circleci.com/gh/cacjs/cac/tree/master) [![donate](https://img.shields.io/badge/$-donate-ff69b4.svg?maxAge=2592000&style=flat)](https://github.com/egoist/donate) [![chat](https://img.shields.io/badge/chat-on%20discord-7289DA.svg?style=flat)](https://chat.egoist.moe) [![install size](https://badgen.net/packagephobia/install/cac)](https://packagephobia.now.sh/result?p=cac)

## Introduction

**C**ommand **A**nd **C**onquer is a JavaScript library for building CLI apps.

## Table of Contents

<!-- toc -->

- [Install](#install)
- [Usage](#usage)
  - [Simple Parsing](#simple-parsing)
  - [Display Help Message and Version](#display-help-message-and-version)
  - [Command-specific Options](#command-specific-options)
  - [Brackets](#brackets)
  - [Variadic Arguments](#variadic-arguments)
  - [Dot-nested Options](#dot-nested-options)
  - [Default Command](#default-command)
  - [With TypeScript](#with-typescript)
- [Projects Using CAC](#projects-using-cac)
- [References](#references)
  - [CLI Instance](#cli-instance)
    - [cli.command(name, description)](#clicommandname-description)
    - [cli.option(name, description, config?)](#clioptionname-description-config)
    - [cli.parse(argv?)](#cliparseargv)
    - [cli.version(version)](#cliversionversion)
    - [cli.help(callback?)](#clihelpcallback)
    - [cli.outputHelp(subCommand?)](#clioutputhelpsubcommand)
  - [Command Instance](#command-instance)
    - [command.option()](#commandoption)
    - [command.action(callback)](#commandactioncallback)
    - [command.alias(name)](#commandaliasname)
    - [command.allowUnknownOptions()](#commandallowunknownoptions)
    - [command.example(example)](#commandexampleexample)
  - [Events](#events)
- [FAQ](#faq)
  - [How is the name written and pronounced?](#how-is-the-name-written-and-pronounced)
- [Contributing](#contributing)
- [Author](#author)

<!-- tocstop -->

## Install

```bash
yarn add cac
```

## Usage

### Simple Parsing

Use CAC as simple argument parser:

```js
// examples/basic-usage.js
const cli = require('cac')()

cli.option('--type [type]', 'Choose a project type', {
  default: 'node'
})

const parsed = cli.parse()

console.log(JSON.stringify(parsed, null, 2))
```

<img width="500" alt="2018-11-26 12 28 03" src="https://user-images.githubusercontent.com/8784712/48981576-2a871000-f112-11e8-8151-80f61e9b9908.png">

### Display Help Message and Version

```js
// examples/help.js
const cli = require('cac')()

cli.option('--type [type]', 'Choose a project type', {
  default: 'node'
})
cli.option('--name <name>', 'Provide your name')

cli.command('lint [...files]', 'Lint files').action((files, options) => {
  console.log(files, options)
})

// Display help message when `-h` or `--help` appears
cli.help()
// Display version number when `-h` or `--help` appears
cli.version('0.0.0')

cli.parse()
```

<img width="500" alt="2018-11-25 8 21 14" src="https://user-images.githubusercontent.com/8784712/48979012-acb20d00-f0ef-11e8-9cc6-8ffca00ab78a.png">

### Command-specific Options

You can attach options to a command.

```js
const cli = require('cac')()

cli
  .command('rm <dir>')
  .option('-r, --recursive', 'Remove recursively')
  .action((dir, options) => {
    console.log('remove ' + dir + (options.recursive ? ' recursively' : ''))
  })

cli.parse()
```

A command's options are validated when the command is used. Any unknown options will be reported as an error. However, if an action-based command does not define an action, then the options are not validated. If you really want to use unknown options, use [`command.allowUnknownOptions`](#commandallowunknownoptions).

### Brackets

When using brackets in command name, angled brackets indicate required command arguments, while sqaure bracket indicate optional arguments.

When using brackets in option name, angled brackets indicate that the option value is required, while sqaure bracket indicate that the value is optional.

```js
const cli = require('cac')()

cli
  .command('deploy <folder>', 'Deploy a folder to AWS')
  .option('--scale [level]', 'Scaling level')
  .action((folder, options) => {
    console.log(folder)
    console.log(options)
  })

cli.parse()
```

### Variadic Arguments

The last argument of a command can be variadic, and only the last argument. To make an argument variadic you have to add `...` to the start of argument name, just like the rest operator in JavaScript. Here is an example:

```js
const cli = require('cac')()

cli
  .command('build <entry> [...otherFiles]', 'Build your app')
  .option('--foo', 'Foo option')
  .action((entry, otherFiles, options) => {
    console.log(entry)
    console.log(otherFiles)
    console.log(options)
  })

cli.help()

cli.parse()
```

<img width="500" alt="2018-11-25 8 25 30" src="https://user-images.githubusercontent.com/8784712/48979056-47125080-f0f0-11e8-9d8f-3219e0beb0ed.png">

### Dot-nested Options

Dot-nested options will be merged into a single option.

```js
const cli = require('cac')()

cli
  .command('build', 'desc')
  .option('--env <env>', 'Set envs')
  .example('--env.API_SECRET xxx')
  .action(options => {
    console.log(options)
  })

cli.help()

cli.parse()
```

<img width="500" alt="2018-11-25 9 37 53" src="https://user-images.githubusercontent.com/8784712/48979771-6ada9400-f0fa-11e8-8192-e541b2cfd9da.png">

### Default Command

Register a command that will be used when no other command is matched.

```js
const cli = require('cac')()

cli
  // Simply omit the command name, just brackets
  .command('[...files]', 'Build files')
  .option('--minimize', 'Minimize output')
  .action((files, options) => {
    console.log(files)
    console.log(options.minimize)
  })

cli.parse()
```

### With TypeScript

First you need `@types/node` to be installed as a dev dependency in your project:

```bash
yarn add @types/node --dev
```

Then everything just works out of the box:

```js
const cac = require('cac')
// OR ES modules
import cac from 'cac'
```

## Projects Using CAC

Projects that use **CAC**:

- [SAO](https://github.com/egoist/sao): ⚔️ Futuristic scaffolding tool.
- [DocPad](https://github.com/docpad/docpad): 🏹 Powerful Static Site Generator.
- [Poi](https://github.com/egoist/poi): ⚡️ Delightful web development.
- [bili](https://github.com/egoist/bili): 🥂 Schweizer Armeemesser for bundling JavaScript libraries.
- [lass](https://github.com/lassjs/lass): 💁🏻 Scaffold a modern package boilerplate for Node.js.
- Feel free to add yours here...

## References

### CLI Instance

CLI instance is created by invoking the `cac` function:

```js
const cac = require('cac')
const cli = cac()
```

#### cli.command(name, description)

- Type: `(name: string, description: string) => Command`

Create a command instance.

#### cli.option(name, description, config?)

- Type: `(name: string, description: string, config?: OptionConfig) => CLI`

Add a global option.

The option also accepts a third argument `config` for addtional config:

- `config.default`: Default value for the option.
- `config.coerce`: `(value: any) => newValue` A function to process the option value.

#### cli.parse(argv?)

- Type: `(argv = process.argv) => ParsedArgv`

```ts
interface ParsedArgv {
  args: string[]
  options: {
    [k: string]: any
  }
}
```

When this method is called, `cli.rawArgs` `cli.args` `cli.options` `cli.matchedCommand` will also be available.

#### cli.version(version)

- Type: `(version: string) => CLI`

Output version number when `-v, --version` flag appears.

#### cli.help(callback?)

- Type: `(callback?: HelpCallback) => CLI`

Output help message when `-h, --help` flag appears.

Optional `callback` allows post-processing of help text before it is displayed:

```ts
type HelpCallback = (sections: HelpSection[]) => void

interface HelpSection {
  title?: string
  body: string
}
```

#### cli.outputHelp(subCommand?)

- Type: `(subCommand?: boolean) => CLI`

Output help message. Optional `subCommand` argument if you want to output the help message for the matched sub-command instead of the global help message.

### Command Instance

Command instance is created by invoking the `cli.command` method:

```js
const command = cli.command('build [...files]', 'Build given files')
```

#### command.option()

Basically the same as `cli.option` but this adds the option to specific command.

#### command.action(callback)

- Type: `(callback: ActionCallback) => Command`

Use a callback function as the command action when the command matches user inputs.

```ts
type ActionCallback = (
  // Parsed CLI args
  // The last arg will be an array if it's an varadic argument
  ...args: string | string[] | number | number[]
  // Parsed CLI options
  options: Options
) => any

interface Options {
  [k: string]: any
}
```

#### command.alias(name)

- Type: `(name: string) => Command`

Add an alias name to this command, the `name` here can't contain brackets.

#### command.allowUnknownOptions()

- Type: `() => Command`

Allow unknown options in this command, by default CAC will log an error when unknown options are used.

#### command.example(example)

- Type: `(example: CommandExample) => Command`

Add an example which will be displayed at the end of help message.

```ts
type CommandExample = ((bin: string) => string) | string
```

### Events

Listen to commands:

```js
// Listen to the `foo` command
cli.on('command:foo', () => {
  // Do something
})

// Listen to the default command
cli.on('command:!', () => {
  // Do something
})

// Listen to unknown commands
cli.on('command:*', () => {
  console.error('Invalid command: %', cli.args.join(' '))
  process.exit(1)
})
```

## FAQ

### How is the name written and pronounced?

CAC, or cac, pronounced `C-A-C`.

This project is dedicated to our lovely C.C. sama. Maybe CAC stands for C&C as well :P

<img src="http://i.giphy.com/v3FeH4swox9mg.gif" width="400"/>

## Contributing

1. Fork it!
2. Create your feature branch: `git checkout -b my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin my-new-feature`
5. Submit a pull request :D

## Author

**CAC** © [EGOIST](https://github.com/egoist), Released under the [MIT](./LICENSE) License.<br>
Authored and maintained by egoist with help from contributors ([list](https://github.com/cacjs/cac/contributors)).

> [Website](https://egoist.sh) · GitHub [@egoist](https://github.com/egoist) · Twitter [@\_egoistlily](https://twitter.com/_egoistlily)
