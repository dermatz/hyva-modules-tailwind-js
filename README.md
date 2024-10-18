# Hyvä-Themes Node Modules utilities

For Hyvä Themes with TailwindCSS we offer the following Node utility functions,
that make it easier to mange and customize your theme styles.

## Library module exports:

The follow functions are offered as an export:

| Function name              | Used in              | Description                                                                                                          |
| -------------------------- | -------------------- | -------------------------------------------------------------------------------------------------------------------- |
| [mergeTailwindConfig]      | `tailwind.config.js` | Function to merge tailwind config files from modules into the theme tailwind config                                  |
| [postcssImportHyvaModules] | `postcss.config.js`  | Postcss plugin to import `tailwind-source.css` files from modules into a theme                                       |
| [hyvaThemesConfig]         |                      | Parses the `app/etc/hyva-themes.json` file. This file is generated by the command `bin/magento hyva:config:generate` |
| [hyvaModuleDirs]           |                      | An array of absolute paths to Hyvä modules                                                                           |
| [twVar]                    | `tailwind.config.js` | Function to add CSS variable syntax to a specific Tailwind Token                                                     |
| [twProps]                  | `tailwind.config.js` | Function to add CSS variable syntax to a range of Tailwind Tokens                                                    |

Click the function name to read more about the function and how to use it in your project.

[mergeTailwindConfig]: #merging-module-tailwind-configurations
[postcssImportHyvaModules]: #merging-module-tailwind-css
[hyvaThemesConfig]: #the-hyvathemesconfig-and-hyvamoduledirs-exports
[hyvaModuleDirs]: #the-hyvathemesconfig-and-hyvamoduledirs-exports
[twVar]: #how-to-add-css-variables-to-tailwindcss-using-twvar-and-twprops
[twProps]: #how-to-add-css-variables-to-tailwindcss-using-twvar-and-twprops

## How to use

Before using the library, install it inside your Hyvä-Theme folder using

```sh
npm install @hyva-themes/hyva-modules
```

> [!NOTE]
> If you are using Hyvä 1.1.14 or newer it will already be included in the `package.json` file by default.

### Merging module tailwind configurations

To use the tailwind config merging, require this module in a themes' `tailwind.config.js` file and wrap the `module.exports` value in
a call to `mergeTailwindConfig()` like this:

```js
const { mergeTailwindConfig } = require('@hyva-themes/hyva-modules');

module.exports = mergeTailwindConfig({
  // theme tailwind config here ...
})
```

#### Specifying content (purge) paths

* **How do I specify relative purge paths content paths?**
  Specify the paths relative to the `view/frontend/tailwind/tailwind.config.js` file in the module, for example `../templates/**/*.phtml`
* **Do I use the TailwindCSS v2 or v3 purge content structure?**
  The script is smart enough to map a modules' purge config structure to the same one that is used in the theme.
  If you don't have any preference, then we suggest you use the v3 structure.

#### Using `require()`

When using `require` with a node library inside a modules' `view/frontend/tailwind/tailwind.config.js` file, prefix the
node module name the global variable `themeDirRequire`:
```js
const colors = require(`${themeDirRequire}/tailwindcss/colors`);
```

The global variable `themeDirRequire` maps to the `node_modules/` folder in the theme that is currently being built.

### Merging module tailwind CSS

To use the hyva-modules postcss plugin, require this library and add it as the first plugin in the themes'
`postcss.config.js` file. **Important:** The hyva-modules plugin has to go before the `require('postcss-import')` plugin!

```js
const { postcssImportHyvaModules } = require('@hyva-themes/hyva-modules');

module.exports = {
    plugins: [
        postcssImportHyvaModules,
        require('postcss-import'),
        ...other PostCSS plugins...
    ]
}
```

#### Creating a modules' `tailwind-source.css` file

To declare custom CSS, create a file `view/frontend/tailwind/tailwind-source.css` in a module.
This file will automatically be imported with `@import` at the end of a themes' `tailwind-source.css` file.

We recommend only using `@import` statements in a module `tailwind-source.css` file,
since other modules CSS files might be appended afterwards,
and `@import` statements are only allowed before regular CSS declarations.


### The `hyvaThemesConfig` and `hyvaModuleDirs` exports

These values are intended to be used in custom build tools and in future utilities.

### How to add CSS variables to TailwindCSS using `twVar` and `twProps`

You can opt into CSS variables in your tailwind configuration using the twVar and twProps function.

With `twVar` you can add a CSS variable to one or more TailwindCSS tokens, for example like this:

```js
const { twVar, mergeTailwindConfig } = require('@hyva-themes/hyva-modules');

module.exports = mergeTailwindConfig({
    theme: {
        extend: {
            colors: {
                primary: {
                    lighter: twVar('primary-lighter', colors.blue["600"]),
                    DEFAULT: twVar('primary', colors.blue["700"]),
                    darker: twVar('primary-darker', colors.blue["800"]),
                },
            }
        }
    }
    // The rest of your tailwind config...
})
```

this will render any Tailwind color utility class with the following CSS value:

```css
.bg-primary {
    --tw-bg-opacity: 1;
    background-color: color-mix(in srgb, var(--color-primary, #1d4ed8) calc(var(--tw-bg-opacity) * 100%), transparent);
}
```

> The method for handling opacity with all color syntaxes is based on the upcoming TailwindCSS v4,
> here they use the CSS function `color-mix()` to make this possible,
> we have reused this to make that transition easier in the future.

And with this you can change the value in your CSS or inline in the page with:

```css
:root {
    --color-primary: hsl(20 80% 50%);
}
```

Now if you don't want to set this for each TailwindCSS token we also offer the functions `twProps`.
This acts as a wrapper and will use the keys as the name for the CSS variable, so if the twProps wraps `primary > lighter` it will create the name `--primary-lighter`.
In this example we can create the same effect as `twVar` with less effort:

```js
const { twProps, mergeTailwindConfig } = require('@hyva-themes/hyva-modules');

module.exports = mergeTailwindConfig({
    theme: {
        extend: {
            colors: twProps({
                primary: {
                    lighter: colors.blue["600"],
                    DEFAULT: colors.blue["700"],
                    darker: colors.blue["800"],
                },
            }),
            textColors: {
                primary: twProps({
                    lighter: colors.blue["600"],
                    DEFAULT: colors.blue["700"],
                    darker: colors.blue["800"],
                }, 'text-primary'),
            }
        }
    }
    // The rest of your tailwind config...
})
```

**Important to note:**

- `twVar` only works with string values.
- `twProps` can be used on any level of your Tailwind config, but it only works with string values.
- `twProps` is best used inside the [reference function method](https://tailwindcss.com/docs/theme#referencing-other-values).

#### How to only apply `twProps` as wrapper but don't apply it to all TailwindCSS tokens

You can use `Object.assign()` to split 2 groups inside any Tailwind config group, e.g. `colors`,
so only one part gets the variables and the other part is left as is.

<details><summary>Code Sample</summary>

```js
const { twProps, mergeTailwindConfig } = require('@hyva-themes/hyva-modules');

module.exports = mergeTailwindConfig({
    theme: {
        extend: {
            colors: Object.assign(
                twProps({
                    primary: {
                        lighter: colors.blue["600"],
                        DEFAULT: colors.blue["700"],
                        darker: colors.blue["800"],
                    },
                    secondary: {
                        lighter: colors.blue["100"],
                        DEFAULT: colors.blue["200"],
                        darker: colors.blue["300"],
                    },
                }),
                {
                    background: {
                        lighter: colors.blue["100"],
                        DEFAULT: colors.blue["200"],
                        darker: colors.blue["300"],
                    },
                    green: colors.emerald,
                    yellow: colors.amber,
                    purple: colors.violet,
                }
            ),
        }
    }
    // The rest of your tailwind config...
})
```

</details>

#### `twVar` and `twProps` arguments

```js
twVar(
    name, // Name of CSS variable
    value, // (Optional) CSS variable fallback value
    prefix, // (Optional) prefix for the name value, allows you to override the automically added 'color' prefix for color values
);
```

```js
twProps(
    values, // The Object of TailwindCSS Tokens
    prefix, // (Optional) prefix for the name values
);
```

## The `hyva-themes.json` configuration

This library uses the file `app/etc/hyva-themes.json` to know which modules might contain tailwind configuration or CSS to merge.
This file is generate by the CLI command `bin/magento hyva:config:generate`.

To add a module to the list, the Magento event `hyva_config_generate_before` can be used.

This happens automatically for Hyvä compatibility modules that register themselves with the `Hyva\CompatModuleFallback\Model\CompatModuleRegistry`.

The `bin/magento hyva:config:generate` command should be run as part of the build, before `npm run build-prod`.

## License

Hyvä Themes - https://hyva.io  
Copyright © Hyvä Themes 2022-present. All rights reserved.  
This library is distributed under the BSD-3-Clause license.

See the [LICENSE.md](LICENSE.md) file for more information.
