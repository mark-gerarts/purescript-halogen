# Module Documentation

## Module Halogen


The main module of the Halogen library. It defines functions for running applications
assembled from the parts defined in the various submodules:

- `Halogen.Signal` for responding to inputs and maintaining state
- `Halogen.HTML.*` for templating HTML documents
- `Halogen.Themes.*` for rendering using common front-end libraries
- `Halogen.Mixin.*` for common additional application features

The type signature and documentation for the [`runUI`](#runUI) function provides a good introduction 
to this library. For more advanced use-cases, you might like to look at the `runUIEff` function instead.


#### `HalogenEffects`

``` purescript
type HalogenEffects eff = (dom :: DOM, ref :: Ref | eff)
```

Wraps the effects required by the `runUI` and `runUIEff` functions.

#### `changes`

``` purescript
changes :: VTree -> SF VTree Patch
```

A signal which emits patches corresponding to successive `VTree`s.

This function can be used to create alternative top-level handlers which use `virtual-dom`.

#### `runUI`

``` purescript
runUI :: forall i eff. (forall a. SF1 i (HTML a i)) -> Eff (HalogenEffects eff) Node
```

`runUI` takes a UI represented as a signal function, and renders it to the DOM
using `virtual-dom`.

The signal function is responsible for rendering the HTML for the UI, and the 
HTML can generate inputs which will be fed back into the signal function,
resulting in DOM updates.

This function returns a `Node`, and the caller is responsible for adding the node
to the DOM.

As a simple example, we can create a signal which responds to button clicks:

```purescript
ui :: SF1 Unit (HTML Unit)
ui = view <$> stateful 0 (\n _ -> n + 1)
  where
  view :: Number -> HTML Unit
  view n = button [ onclick (const unit) ] [ text (show n) ]
```

#### `Handler`

``` purescript
type Handler r i eff = r -> Driver i eff -> Eff (HalogenEffects eff) Unit
```

This type synonym is provided to tidy up the type signature of `runUIEff`.

The _handler function_ is responsible for receiving requests from the UI, integrating with external
components, and providing inputs back to the system based on the results.

For example:

```purescript
data Input = SetDateAndTime DateAndTime | ...

data Request = GetDateAndTimeRequest | ...

appHandler :: forall eff. Handler Request Input eff 
appHandler GetDateAndTimeRequest k =
  get "/date" \response -> k (readDateAndTime response)
```

#### `Driver`

``` purescript
type Driver i eff = i -> Eff (HalogenEffects eff) Unit
```

This type synonym is provided to tidy up the type signature of `runUIEff`.

The _driver function_ can be used by the caller to inject additional inputs into the system at the top-level.

This is useful for supporting applications which respond to external events which originate
outside the UI, such as timers or hash-change events.

For example, to drive the UI with a `Tick` input every second, we might write something like the following:

```purescript
main = do
  Tuple node driver <- runUIEff ui absurd handler
  appendToBody node
  setInterval 1000 $ driver Tick
```

#### `runUIEff`

``` purescript
runUIEff :: forall i a r eff. SF1 i (HTML a (Either i r)) -> (a -> VTree) -> Handler r i eff -> Eff (HalogenEffects eff) (Tuple Node (Driver i eff))
```

`runUIEff` is a more general version of `runUI` which can be used to construct other
top-level handlers for applications.

`runUIEff` takes a signal function which creates HTML documents containing _requests_, 
and a handler function which accepts requests and provides new inputs to a continuation as they
become available.

For example, the handler function might be responsible for issuing AJAX requests on behalf of the
application.

In this way, all effects are pushed to the handler function at the boundary of the application.



