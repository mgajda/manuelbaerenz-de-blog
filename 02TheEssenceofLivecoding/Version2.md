<!---
```haskell
{-# LANGUAGE NamedFieldPuns #-}
{-# LANGUAGE OverloadedStrings #-}

module Version2 where

-- wai
import Network.Wai

-- lucid
import Lucid

-- essence-of-live-coding
import LiveCoding

-- essence-of-live-coding-warp
import LiveCoding.Warp

-- manuelbaerenz-de-blog
import Version0 (LiveWebApp)
import Version1 hiding (mutate)
```
-->

#### ...keep the state?!

It's getting really interesting as soon as we change the _internal state type_ of our web application.
How about we don't only display the counter,
but also the time when someone last visited the website?

Let's do everything nice and tidy,
and wrap the counter state into a newtype
before extending it:

```haskell
data State = State
  { counter :: Integer
  }
  deriving Data

initialState :: State
initialState = State 0

counterApp :: LiveWebApp
counterApp = Cell { cellState = initialState, cellStep }
  where
    cellStep counter request
      = let counter' = mutate request counter
        in return (response counter', counter')

mutate :: Request -> State -> State
mutate request State { counter } = State $ case pathInfo request of
  ["inc"] -> counter + 1
  ["dec"] -> counter - 1
  _       -> counter

content :: State -> Html ()
content State { counter } = do
  h3_ $ "Counter: " <> toHtml (show counter)
  anchors
```

So far, nothing has changed semantically.
But we've refactored in such a way that we can now add a second field to our state type.
