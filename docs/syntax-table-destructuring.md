# Table Destructuring Syntax



## Summary
Introduce new syntax for unpacking values from tables.

```lua
local .{ .foo, one, ["@bar"] as bar } = thing
-- foo == thing.foo
-- one == thing[1]
-- bar == thing["@bar"]
```

This RFC addresses issues brought up from previous table destructuring RFC's.



## Motivation
Aliasing table items through variable assignments is a common pattern in luau, especially when working with large libraries. However doing this with a lot of items becomes repetitve:
```lua
local useEffect = React.useEffect
local useMemo = React.useMemo
local useState = React.useState
```

With destructuring syntax this could be shortened to the following:
```lua
local .{ .useEffect, .useMemo, .useState } = React
```

## Design

We propose that the table destructuring syntax is prefixed with a period (`.`) symbol. Without a prefix destructuring becomes ambiguous in certain scenarios, such as the one below.
```lua
baz = foo
{ bar } = test
```
*(This is ambiguous as it can be interpretted as `baz = foo { bar }; = test` or `baz = foo; { bar } = test`).*

### Key destructuring
We propose the following syntax for destructuring by keys:
```lua
.{ ["foo"] as foo } = thing
-- foo == thing["foo"]
```

If the key can be expressed as a valid variable name then a shorthand can be used instead:
```lua
.{ .foo } = thing
```

Shorthands can also be destructured with a different name:
```lua
.{ .foo as bar } = thing
-- bar == thing.foo
```

We propose nested key destructuring via the following syntax (This also works with shorthands):
```lua
.{ ["@foo"] as .{ .bar } } = thing
-- bar == thing["@foo"].bar
```

## Array destructuring
We propose the following syntax for array destructuring:
```lua
.{ one, two, three } = thing
-- one == thing[1]
-- two == thing[2]
-- three == thing[3]
```

We propose the following syntax for nested array destructuring:
```lua
.{ one, .{ apple }, three } = thing
-- one == thing[1]
-- apple == thing[2][1]
-- three == thing[3]
```

## Mixed Destructuring
Key and array destructuring can be used together:
```lua
.{ .foo, one, .bar as .{ .baz }, .{ apple } } = thing
-- foo == thing.foo
-- one == thing[1]
-- baz == thing.bar.baz
-- apple = thing[2][1]
```

## Rest Destructuring

We propose destructuring the rest of the properties via the following syntax (`rest` can be any identifier). Only one rest parameter can be defined.
```lua
.{ ["@foo"] as .{ .bar }, ...rest } = thing
```

### Local Destructuring
We propose that if a destructure assignment is preceeded with `local` then all of the destructured variables will be local.
```lua
local .{ hello, world } = thing
```

### Function Argument Destructuring
We propose that function arguments can be destructured:

```lua
type Props = {
    apple: any,
    pear: any
}

function test(.{ .apple, .pear }: Props)
    ...
end
```
*(The type annotation is not neccesary, its purely there for demonstration purposes).*


## Drawbacks
- Prefixing table destructuring with a symbol may not fit into the language as historically luau has generally not used symbol prefixes. However there are some examples of symbol prefixes (especially in recent years):
  - length operator (`#thing`).
  - function attributes (`@native`).
  - negation operator (`~SomeType`).

## Alternatives
- Alternative ideas for table destructuring have been proposed before, most of which have been rejected:
  - https://github.com/luau-lang/rfcs/pull/24 (`local {.a, .b} = t`)
  - https://github.com/luau-lang/luau/pull/629 (`local { a, b } = t`)
  - https://github.com/luau-lang/rfcs/pull/95 (`local { a, b } = t`)

  The main reasons for their rejection include ambiguity issues, not handling arrays, and not handling non-local assignments.

- For nested destructuring we could omit the period prefix however this may increase parsing complexity. For example the following:
  ```lua
  local .{ .foo as .{ .bar } }
  ```
  would become:
  ```lua
  local .{ .foo as { .bar } }
  ```

