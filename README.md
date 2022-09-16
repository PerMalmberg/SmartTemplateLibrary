# Smart Template Library (STL)

A library that provides a simple smart template library, written in Lua. It uses a syntax similar to Twig:

`{{ variable }}` will print any variable existing on the current environment

`{% some_lua %}` will execute the Lua expression in place of `some_lua`

You can use normal Lua inside `{% ... %}` blocks, so for example, this is a simple conditional check:

```
Greeting:
{% if name == 'Wolfe' then %}
  Hola, {{ name }}!
{% else %}
  Hello, {{ name }}!
{% end %}
```

When provided with the "Wolfe" name, the output will be "Hola, Wolfe!" while in any other case the output will be "Hello, Your Name Here!"

## Installation

The library can be used both in vanilla Lua or any game supporting it, as far it supports `require`, or with the [Lua CLI for Dual Universe](https://github.com/wolfe-labs/DU-LuaC).

For those using the Lua CLI for DU, install the library using `du-lua import https://github.com/wolfe-labs/SmartTemplateLibrary` and include it in your code with `STL = require('@wolfe-labs/STL:Template')`

For vanilla Lua or those not using the CLI, just copy the `Template.lua` file into your project's source location and include it with `STL = require('Template')`

## Usage

The library is very simple and doesn't provide many functions, it's focused for usage in games (specially Dual Universe), where script space is often limited.

After using `require` to bring the library to your code, you can build a template by invoking it straight away. It will return an instance of that template, which can be executed by invoking that instance:

```lua
-- Imports our template library
STL = require('Template')

-- Compiles our template
hello = STL('Hello, {{name}}!')

-- Executes our template and prints its result for a few different names:
print(hello{ name = 'Matt' })
print(hello{ name = 'Wolfe' })
print(hello{ name = 'Example' })
```

Since the templates use basically Lua inside the `{% ... %}` tags, you are free to add as much complexity you feel like you need, just keep in mind that every time your template is rendered all code inside it will be executed, so things like function declarations should be avoided. You can, though, pass them as a template global or as an argument for only one execution:

```lua
-- Imports our template library
STL = require('Template')

-- Does addition
function add(a, b)
  return a + b
end

-- Does subtraction
function sub(a, b)
  return a - b
end

-- Compiles our template, passing "add" as a global
hello = STL('{{ A }} + {{ B }} = {{ add(A, B) }}, while {{ A }} - {{ B }} = {{ sub(A, B) }}', {
  add = add,
})

-- This will work, since "sub" is present on our scope
print(hello{ sub = sub, A = 1, B = 2 } })

-- This will not work, since "sub" is missing on our scope
print(hello{ A = 1, B = 2 } })
```

## Performance and Internals

This library tries to keep things as simple as possible regarding its inner workings, while keeping as performant as possible. The most complex part of it happens when you first "compiles" your template.

At that point all `{{ ... }}` and `{% ... %}` tags will be processed in advance and replaced with Lua. Also, everything else that is not part of those tags will also be converted to Lua. You can see the generated code by accessing the `code` attribute of the returned template instance.

That code is then inserted into a small snippet of code that returns the proper rendering function, that snippet of code takes care of initializing all globals on the environment which your template code will be running with the variables passed to it.

The snippet is finally passed into `load` which will do the actual Lua compilation and we're left with a callable function that is executed every time a render happens.panel

Since we're using that cached function instead of using regular expressions or calling `load` every time its rendered, things usually happen very fast and work well even for things such as Dual Universe HUDs updated in realtime.

Another thing that is taken into consideration is coroutines, while they weren't really an option for render due to our approach, they are taken into consideration during "compilation", so have in mind that yield will be called quite often if the compilation happens inside a coroutine.