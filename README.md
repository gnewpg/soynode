soynode
=======

Utility for working with [Closure Templates](https://developers.google.com/closure/templates/),
aka Soy, from within a node.js application.  Supports dynamic recompilation and loading for fast
iteration during development.

Installing
----------

Either:

1. Fork, clone or download the source from GitHub, or
2. Install via NPM using `npm install soynode`


Usage
-----

```js
var soynode = require('../lib/soynode')

soynode.setOptions({
    tmpDir: '/tmp/soynode-example'
  , allowDynamicRecompile: true
})

soynode.compileTemplates(__dirname, function (err) {
  if (err) throw err
  // Templates are now ready to use.
  console.log(soynode.render('example.message.hello', {
      name: process.env.USER
    , date: new Date().toLocaleTimeString()
  }))
})
```

Also, see `examples/example.js`.

`soynode.get(templatename)` - Returns a JS function corresponding to the template name.

`soynode.render(templatename, data)` - Returns a string that results from executing a template.

`soynode.setOptions(opts)` - Change the options, see section below.

`soynode.compileTemplates(dir, callback)` - Compiles and loads all `.soy` files in the directory.

`soynode.loadCompiledTemplates(dir, callback)` - Loads already compiled templates.

`soynode.loadCompiledTemplateFiles(files, callback)` - Loads already compiled templates.

Where "template name" is referred to, it means the namespace + template name as defined in the Soy
file, and the full JS name that the Soy Compiler generates, for example `project.section.screen`.
See the [Hello World JS](https://developers.google.com/closure/templates/docs/helloworld_js) doc on
the Closure site for more background.

Options
-------

Options can be set via `soynode.setOptions(options)`, the keys can contain the following:

- `tmpDir` {string} Path to a directory where temporary files will be written during compilation.
  [Default: /tmp/soynode]
- `allowDynamicRecompile` {boolean} Whether to watch for changes to the templates. [Default: false]
- `eraseTemporaryFiles` {boolean} Whether to erase temporary files after a compilation.
[Default: false]
- `classpath` {array} Additional classpath to pass to the soy template compiler. This makes the
  inclusion of plugins possible. [Default: empty]
- `pluginModules` {array} Java classnames of plugin modules to pass to the soy template compiler.
  [Default: empty]
- `additionalArguments` {array} Additional command-line arguments for the soy template compiler.
  [Default: empty]

**NOTE: Options should be set before templates are loaded or compiled.**

### Adding plugins

The soy template compiler can be added [Java plugins](https://developers.google.com/closure/templates/docs/plugins)
that provide additional functions. Consider you have a Java class `com.example.TestFunctions` in a file
`testFunctions.jar`, this would be the syntax to enable it:

```js
soynode.setOptions({
    classpath: [ './testFunctions.jar' ]
  , pluginModules: [ 'com.example.TestFunctions' ]
})
```

Implementation Notes
--------------------

The templates are loaded using Node's [VM Module](http://nodejs.org/api/vm.html).  This allows us to
execute the generated `.soy.js` files as is without a post processing step and without leaking the
template functions into the global scope.

Calling `soynode.get` executes code which returns a reference to the template function within the
VM Context.  The reference is cached, providing a 10x speed up over fetching the template function
each time, or evaluating it in place and returning the template output over the VM boundary.

Contributing
------------

Questions, comments, bug reports, and pull requests are all welcome. Submit them at
[the project on GitHub](https://github.com/Obvious/soynode/).

Bug reports that include steps-to-reproduce (including code) are the best. Even better, make them in
the form of pull requests.

Author
------

[Dan Pupius](https://github.com/dpup)
([personal website](http://pupius.co.uk/about/)), supported by
[The Obvious Corporation](http://obvious.com/).

License
-------

Copyright 2012 [The Obvious Corporation](http://obvious.com/).

Licensed under the Apache License, Version 2.0.
See the top-level file `LICENSE.txt` and
(http://www.apache.org/licenses/LICENSE-2.0).
