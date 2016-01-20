# Hello World

We will start with "Hello world" as programming tradition requires:

```elm
module Main (main) where

import Html exposing (Html)


main : Html
main =
  Html.div [] [ Html.text "Hello world" ]
```

An Elm program consists of one or more modules, where the `main` function acts
as the entry point. The current example exists of a single module called `Main`.[^1]

[^1]: Calling this module `Main` is not necessary, but just a convention we like to follow.
