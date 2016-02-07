# Signals

In purely functional languages variables are immutable. This is really nice when we want to reason about our program, because values do not change. So we don't have to figure out what the value of a particular variable was at a given point in time. But what if we want a dynamic app, something where we can click on buttons to make stuff change on the screen. It sounds like this is something people would like to do. It seems imperative we need the concept of things that can change value over time to achieve this. How do we make this work in a purely functional language where values are immutable?

This is why there is such a thing as "signals" in Elm. Signals are basically values that can change. The most basic signal is just a signal that will give the current time:

```elm
import Signal exposing (Signal)
import Time exposing (every, second, Time)


time : Signal Time
time =
  every second
```

We see here that the module `Time` exposes a datatype called `Time` which is just a representation of a given time. The expression `every second` will give us a very simple signal that will change every second to the current time.

Awesome, we now have variables that can change over time, they're just the same variables we had before but now wrapped in a `Signal` constructor. So how do we create new ones? Creating a signal from a basic variable is simple, we just make it a constant signal:

```
import Signal exposing (Signal, constant)

universe : Signal Int
universe =
  constant 42
```


## Mapping

But we will also need a way to change these values, we will need some kind of tool to map one signal to another.

```
import Signal exposing (Signal, constant, map)

universe : Signal Int
universe =
  constant 42

alternateUniverse : Signal Int
  map ((+) 1) universe
```

The function `Signal.map` takes a normal function that works on non-signal variables and lifts it to the signal level. The type is `(a -> result) -> Signal a -> Signal result`.

Using this new found knowledge, we can build a simple clock app that will show the current Unix time in seconds and will automatically update every second:

```elm
module Main (..) where

import Html exposing (Html, div, text)
import Signal exposing (Signal, map)
import Time exposing (every, second, inSeconds)


time : Signal Float
time =
  map inSeconds (every second)


representTime : Float -> Html
representTime time =
  div [] [ text <| toString time ]


main : Signal Html.Html
main =
  map representTime time
```


## Merging

In the previous section we built a simple clock that continuously updates the time. But the universe has more dimensions than just time, it also has special dimensions. We also want to know where we are in the vast universe! So we need another signal to represent our position.

Elm has a built in module `Mouse` that can give us the current position of the mouse. Let's define our place in the universe as

```elm
place : Signal (Int, Int)
place =
  Mouse.position
```

Now we would like to have a data type that defines our universe like:

```elm
type alias Universe =
  { time : Time
  , place : (Int, Int)
  }
```

and we would like to make a signal of the known universe. In principle we already have all the components we need, we have a time signal and a place signal. But how do we combine those two into one signal?

To do this, we can define a `UniverseEvent`. Whenever the time changes, or our position changes, this will be a `UniverseEvent`.

```elm
type UniverseEvent
  = ClockTick Time
  | Movement (Int, Int)
```

We can redefine our `time` and `place` in terms of `UniverseEvent`s:

```elm
time : Signal UniverseEvent
time = 
  map (ClockTick << inSeconds) (every second)

place : Signal UniverseEvent
place =
  map Movement Mouse.position
```

Because our two signals now have the same type, we can easily merge them together into one signal containing all the changes in the universe!

```elm
events : Signal UniverseEvent
events =
  Signal.merge time place
```

## Folding

Now that we have the signal that contains all the changes happening in the universe, how do we make a signal out of the current state of the universe? Since we know all the changes, the only information we're missing is what was there at the beginning, the big bang. So let's define that:

```elm
initialUniverse : Universe
initialUniverse =
  { time = 0
  , place = (0, 0)
  }
```

To get to the state of the current universe, we just need to continuously update the `initialUniverse` with all the changes contained in the `events` signal:

```elm
updateUniverse : UniverseEvent -> Universe -> Universe
updateUniverse event universe =
  case event of
    ClockTick time ->
      { universe | time = time }

    Movement movement ->
      { universe | place = movement }
```

This update function will just say how to update a given universe based on a given event. So if an event tells us that the time has changed to a new value, we will take the information contained in the universe from before the event, and update the time with the new time. The same thing happens when there's movement except then we will update our place in the universe.

The final piece in the puzzle to actually create a signal of the current universe is a function that wil continuously apply the update function on an initial value. This function is called `foldp`. Its type is `(a -> state -> state) -> state -> Signal a -> Signal state)`, which is exactly what we needed. Now we can define the current universe:

```elm
currentUniverse : Signal Universe
currentUniverse =
  Signal.foldp updateUniverse initialUniverse events
```

Throwing all this together we get an Elm app which will show us the current time and the current position of the mouse.

```elm
module Main (main) where

import Html exposing (div, Html, text)
import Mouse
import Signal exposing (map, merge, Signal)
import Time exposing (every, inSeconds, second, Time)


type alias Universe =
  { time : Time
  , place : ( Int, Int )
  }


initialUniverse : Universe
initialUniverse =
  { time = 0
  , place = ( 0, 0 )
  }


type UniverseEvent
  = ClockTick Time
  | Movement ( Int, Int )


updateUniverse : UniverseEvent -> Universe -> Universe
updateUniverse event universe =
  case event of
    ClockTick time ->
      { universe | time = time }

    Movement movement ->
      { universe | place = movement }


time : Signal UniverseEvent
time =
  map (ClockTick << inSeconds) (every second)


place : Signal UniverseEvent
place =
  map Movement Mouse.position


events : Signal UniverseEvent
events =
  Signal.merge time place


viewUniverse : Universe -> Html
viewUniverse universe =
  div
    []
    [ div [] [ text <| "Time: " ++ toString (inSeconds universe.time) ]
    , div [] [ text <| "x: " ++ toString (fst universe.place) ]
    , div [] [ text <| "y: " ++ toString (snd universe.place) ]
    ]


currentUniverse : Signal Universe
currentUniverse =
  Signal.foldp updateUniverse initialUniverse events


main : Signal Html.Html
main =
  Signal.map viewUniverse currentUniverse
```
