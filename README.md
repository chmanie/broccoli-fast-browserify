# broccoli-fast-browserify

The broccoli-fast-browserify-watchify plugin bundles your assets with
[browserify](https://github.com/substack/browserify).

This plugin differs from the other watchify/browserify plugins for broccoli in
that it uses caching to significantly speed up compilation. There is an existing
broccoli-watchify plugin that would seem to use this caching, however it does
not detect changes through symlinks, which many broccoli plugins use under the
hood.

`broccoli-fast-browserify` does not depend on the `watchify` package, which uses filesystem change detection to determine when to rebuild. We don't need this functionality because broccoli does that for us! Instead, this plugin borrows the caching logic from `watchify` and adapt it to be more suitable for broccoli.

## Installation

```bash
npm install --save-dev broccoli-fast-browserify
```

## Getting Started Quickly

By default, `broccoli-fast-browserify` turns all `*.browserify` files in the input tree into a bundle with the same name, but with a `.js` extension. Each `*.browserify` file is also the sole entry point for the bundle.

No other files are output from `broccoli-fast-browserify`. Here is how you would use `broccoli-fast-browserify` with the default configuration:

```js
var fastBrowserify = require('broccoli-fast-browserify');

var tree = fastBrowserify(tree);
```

## Customizing broccoli-fast-browserify

`broccoli-fast-browserify` provides some simple options that let you control how bundles are created. If you want to use a different extension for detecting bundles, or a different output extension, you can provide the `bundleExtension` or `outputExtension` options:

```js
var fastBrowserify = require('broccoli-fast-browserify');

var tree = fastBrowserify(tree, {
  bundleExtension: ".bundle",
  outputExtension: ".es5"
});
```

If you want to manually specify what bundle to build, with specific input files, you can do that too, but the syntax is a little different. The best way to show how to do this is with an example. If you wanted to create an output bundle called `lib/bundle.js`, where all `index.js` files from the input tree are entrypoints, this is what you would do:

```js
var fastBrowserify = require('broccoli-fast-browserify');

var tree = fastBrowserify(tree, {
  bundles: {
    'lib/bundle.js': {
      entryPoints: ['**/index.js']
    }
  }
});
```

Entrypoints is an array of globs, so you have a lot of power over what gets built.

## More Customizability, Generating Multiple Bundles with One Configuration Entry

If you would like to generate multiple bundles based on some criteria, like the default behavior (i.e. generating one bundle per `*.browserify` input file), then you can set the `glob` option to `true` in your bundle specification.

For example, to achieve `broccoli-fast-browserify`'s default behavior of generating one bundle per `*.browserify` file, here is what you would use (this *is* the implementation used by `broccoli-fast-browserify`, adapted for this example):

```javascript
var fastBrowserify = require('broccoli-fast-browserify');

var tree = fastBrowserify(tree, {
  bundles: {
    "**/*.browserify": {
      glob: true,

      entryPoints: function(relativePath) {
        return [relativePath];
      },

      outputPath: function(relativePath) {
        // remove the bundle extension from the filename
        var out = relativePath.replace(/\.browserify$/, '');

        // If the file already has the output file extension (e.g. bundle.js.browserify)
        // Then don't do anything, else add the output extension to the filename
        if (! out.match(/\.js$/)) {
          out = out + '.js';
        }

        return out;
      }
    }
  }
});
```

With the `glob` option set to true, the plugin will generate a bundle for each file or directory matching the key of the bundle specification (in this case, `**/*.browserify`).

You must provide functions for `entryPoints` and `outputPath`, which are passed each file/directory matched by the bundle glob. In this example, `entryPoints` and `outputPath` are both passed the relative paths to each `*.browserify` file in the input tree.

Let's look at the implementation of these methods:

The `entryPoints` function simply returns the `*.browserify` file wrapped in an array, since we only want one entryPoint per bundle for this example.

The `outputPath` function first strips off the ".browserify" extension, and if the file doesn't already end with '.js', it adds the '.js' extension (this handles cases where the input file is `.js.browserify`).

### Another Example

Suppose you have the following directory structure:

```
ROOT/
  lib/
    packages/
      utilities/
        index.js
        ... more files
      ui/
        index.js
        ... more files
      server/
        index.js
        ... more files
```

Let's say you want to generate a bundle for each of the packages, where the entry point is that package's `index.js` file. And then you wanted to generate a package which contains all three sub packages. We will put these packages in the root of the output tree, with a name which is their package name plus '.js'.

This is how you might accomplish that:

```javascript
var fastBrowserify = require('broccoli-fast-browserify');

var tree = fastBrowserify(tree, {
  bundles: {
    "lib/packages/*": {
      glob: true,

      entryPoints: function(relativePath) {
        // in this case, relativePath is the specific packages directory,
        // since the glob above matches our directories, rather than files
        return [path.join(relativePath, 'index.js')];
      },

      outputPath: function(relativePath) {
        // Again, relativePath is a directory here (e.g. lib/packages/utilities),
        // and we want to return a javascript file, so we take the basename (utilities)
        // and tack '.js' onto the end, so the output bundle file will be
        // '/utilities.js', '/ui.js' and '/server.js'
        return path.basename(relativePath) + '.js'
      }
    }
  }
});
```

## License

The 3 Clause BSD License (3-Clause-BSD). See [LICENSE](LICENSE) for details.

Copyright © 2014 Caleb Land.
