# Hello World

We will start with "Hello world" (as programming tradition requires):

```elm
module Main (main) where

import Html exposing (Html)


main : Html
main =
  Html.div [] [ Html.text "Hello world" ]
```

## Modules

An Elm program consists of one or more modules, where the `main` function acts
as the entry point. The current example exists of a single module called `Main`.[^1]

Modules are declared using the following syntax: <pre>module <i>Name</i> [<i>exports</i>] where</pre>
Where:

- _Name_ is the name of the module (obviously). This name has to match the file name, thus in this case the code has to be in a module called `Main.elm`.

[^1]: Calling this module `Main` is not necessary, but just a convention we like to follow to make it easier to find an application's entry point.
