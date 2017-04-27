# Elm Style Guide

Inspired by and based on NoRedInk's [Style Guide](https://github.com/NoRedInk/elm-style-guide/blob/master/README.md)

## Table of Contents

* [How to Namespace Modules](#how-to-namespace-modules)
* [How to Structure Modules for a Page](#how-to-structure-modules-for-a-page)
* [Ports](#ports)
* [Model](#model)
* [Naming](#naming)
* [Function Composition](#function-composition)
* [Syntax](#syntax)
* [Consider refactoring if...](#Consider-refacotring-if...)
* [Tooling](#tooling)


## How to Namespace Modules

### `Common.`
`Common.Form`, `Common.DailyCalendar`, `Common.ExplainerWell`

View functions which can be shared across pages. Modules here are specific to our product and our styles.

#### Examples
- The Explainer Well which is an abstraction of a common pattern found in multiple pages.

### `Util.`
`Util.Result`, `Util.List`, `Util.Form`

Utilities that help with using core or library data types in common ways. This can include functions which are already exist in large packages but that we aren't ready to import.
They often will be akin `.Extra` packages. Modules here differ from `Common.` by not being specific to SMRxT styles and differ from root-level modules by not completely encompassing a particular problem.

#### Examples
- Helpers for RemoteData
- Functions to generate dates

### `Types.`
`Types.Therapy`, `Types.Device`, `Types.AdherenceRecord`

Types (and functions closely related to that type) shared across multiple pages. Decoders and Encoders for the types are included alongside the type definition.

#### Examples
- Types that represent concepts used in multiple pages (`Types.MessageHistoryItem`)
- Helpers for those types

### `Pages.`
`Pages.Patient.Messaging.Main`, `Pages.Admin.Inventory.Main`, `Pages.Dashboard.ProgramDashboard.Main`

A page on the site, which has its own URL. The module name should be roughly equivalent to the URL. This should not include reusable components that are shared between pages - it should only be logic specific to the page.

#### Examples
- `Pages.Admin.Inventory.Main` corresponds to the URL `/admin/inventory`. In particular, the `Admin.Inventory` part corresponds to the `/admin/inventory` part. The `Inventory.Main` part is subject to [How to Structure Modules for A Page](#how-to-structure-modules-for-a-page).

### `Api.`
`Api.Therapy`, `Api.Patient`, `Api.MessageTemplate`

Provide wrappers around web requests to our api. Each call should use `RemoteData.WebData` (old api functions may still use results and should be updated/removed as possible).

All functions should follow the convention of `[method][dataType] : [id|dataType] -> (RemoteData.WebData [dataType] -> msg) -> Cmd msg` (e.g. `fetchPatient : PatientId -> (RemoteData.WebData Patient -> msg) -> Cmd msg`)

#### Examples
- Api calls to fetch, update, and delete patients

### `Routes.`
`Routes.Patient`, `Routes.Program`

Each `Routes` Module represents a base-level piece of a URL. Routes should be created when a page needs to create links between pages. Each route understands how to construct a valid URL to itself.

#### Examples
- An edit url for patients (belongs in `Routes.Patient`)

### Top-level modules
`Paging`

Something reusable that we might open source, that aren't tied directly to any SMRxT stuff. Name it what we'd name it if we'd already open-sourced it.

Make as much of this opensource-ready as possible:

- Must have simple documentation explaining how to use the component. No need to go overboard, but it needs to be there. Imagine you're publishing the package on elm-package! Use `--warn` to get errors for missing documentation.
- Expose Model and the Msg constructors.
- Use `type alias Model a = { a | b : c }` to allow extending of things.
- Provide an API file as example usage of the module.
- Follow either the [elm-api-component](https://github.com/NoRedInk/elm-api-components) pattern, or the [elm-html-widgets](https://github.com/NoRedInk/elm-html-widgets) pattern

#### Examples
- Paging

## How to Structure Modules for a Page

Our Elm apps generally take this form:

- Main.elm
- Model.elm
- Update.elm
- View.elm
- Flags.elm

Inside **`Model.elm`**, we contain the actual model for the view state of our program. Note that we generally don't include non-view state inside here, preferring to instead generalize things away from the view where possible. For example, we might have a record with a list of assignments in our `Model` file, but the assignment type itself would be in a module called `Data.Assignment`.

**`Update.elm`** contains our update code. This includes the `Msg` types for our view. Inside here most of our business logic lives.

Inside **`View.elm`**, we define the view for our model and set up any event handlers we need.

**`Flags.elm`** contains a decoder for the flags of the app. We aim to keep our decoders basic and so decode into a special `Flags` type that mirrors the structure of the raw JSON instead of the structure of the `Model` type. The `Flags` and `Model` modules should not depend on each other.

**`Main.elm`** is our entry file. Here, we import everything from the other files and actually connect everything together.

It calls `Html.programWithFlags` with:
- `init`, defined in `Main`, runs `Flags.decodeFlags` and turns the resulting `Flags` type into a `Model`.
- `update` is `Update.update >> batchUpdate`. See [NoRedInk/rocket-update](https://github.com/NoRedInk/rocket-update) for details on `batchUpdate`.
- `view` is simply `View.view`.
- `subscriptions`, defined in `Main`, contains any subscriptions this app relies on.

Additionally we setup ports for interop with JS in this file. We run elm-make on this file to generate a JS file that we can include elsewhere.

To summarize:

- Main.elm
    - Our entry point. Decodes the flags, creates the initial model, calls `Html.programWithFlags` and sets up ports.
    - Compile target for `elm-make`
    - Imports `Model`, `Update`, `View` and `Flags`.

- Model.elm
    - Contains the `Model` type for the view alone.
    - Imports nothing but generalized types that are used in the model
    - Exports `Model`

- Update.elm
    - Contains the `Msg` type for the view, and the update function.
    - Imports `Model`
    - Exports `update : Msg -> Model -> (Model, List (Cmd Msg))` and `Msg`

- View.elm
    - Contains the view code
    - Imports `Model` and `Update` (for the `Msg` types)
    - Exports `view : Model -> Html Msg`

- Flags.elm
    - Contains the flags decoder
    - Imports nothing but generalized decoders.
    - Exports `Flags`, `decodeFlags : String -> Result String Flags`

## Ports

### All ports should bring things in as `Json.Value`

The single source of runtime errors that we have right now are through ports receiving values they shouldn't. If a `port something : Signal Int` receives a float, it will cause a runtime error. We can prevent this by just wrapping the incoming things as `Json.Value`, and handle the errorful data through a `Decoder` result instead.

### Ports should always have documentation

I don't want to have to go out from our Elm files to find where a port is being used most of the time. Simply adding a line or two explaining what the port triggers, or where the values coming in from a port can help a lot.


## Model

### Model shouldn't have _any_ view state within them if they aren't tied to views

For example, an assignment should not have a `openPopout` attribute. Doing so means we can't use that type again in another situation.


## Naming

### Use descriptive names instead of tacking on underscores

Instead of this:

```elm
-- Don't do this --
markDirty model =
  let
    model_ =
      { model | dirty = True }
  in
    model_
```

...just come up with a name.

```elm
-- Instead do this --
markDirty model =
  let
    dirtyModel =
      { model | dirty = True }
  in
    dirtyModel
```

## Function Composition

### Prefer forward application [`|>`](http://package.elm-lang.org/packages/elm-lang/core/latest/Basics#|>) "pipelines" to backward application

Instead of this:
```elm
-- Don't do this --
Maybe.withDefault "default" <| Maybe.map String.toUpper <| patient.alias
```
...prefer using parentheses
```elm
-- Instead do this --
patient.alias
    |> Maybe.map String.toUpper
    |> Maybe.withDefault "default
```


### Only use backward function application [`<|`](http://package.elm-lang.org/packages/elm-lang/core/latest/Basics#<|) when parens would be awkward

Instead of this:

```elm
-- Don't do this --
foo <| bar <| baz qux
```

...prefer using parentheses, because they'd look fine:

```elm
-- Instead do this --
foo (bar (baz qux))
```

However this would be awkward:

```elm
-- Don't do this --
customDecoder string
  (\str ->
    case str of
      "one" ->
        Result.Ok 1

      "two" ->
        Result.Ok 2

      "three" ->
        Result.Ok 3
    )
```

...so prefer this instead:

```elm
-- Instead do this --
customDecoder string
  <| \str ->
      case str of
        "one" ->
          Result.Ok 1

        "two" ->
          Result.Ok 2

        "three" ->
          Result.Ok 3
```

### Keep function composition going the same direction

Instead of this:
```elm
-- Don't do this --
List.head patients
    |> Maybe.map (String.toUpper << .alias)
    |> Maybe.withDefault "default
```
...prefer using parentheses
```elm
-- Instead do this --
List.head patients
    |> Maybe.map (.alias >> String.toUpper)
    |> Maybe.withDefault "default
```

### Always use [`Json.Decode.Pipeline`](https://github.com/NoRedInk/elm-decode-pipeline) instead of [`mapN`](http://package.elm-lang.org/packages/elm-lang/core/latest/Json-Decode#map2)

Even though this would work...

```elm
-- Don't do this --
algoliaResult : Decoder AlgoliaResult
algoliaResult =
  map6 AlgoliaResult
    (field "id" int)
    (field "name" string)
    (field "address" string)
    (field "city" string)
    (field "state" string)
    (field "zip" string)
```

...it's inconsistent with the longer decoders, and must be refactored if we want to add more fields.

Instead do this from the start:

```elm
-- Instead do this --
import Json.Decode.Pipeline exposing (required, decode)

algoliaResult : Decoder AlgoliaResult
algoliaResult =
  decode AlgoliaResult
    |> required "id" int
    |> required "name" string
    |> required "address" string
    |> required "city" string
    |> required "state" string
    |> required "zip" string
```

This will also make it easier to add [optional fields](http://package.elm-lang.org/packages/NoRedInk/elm-decode-pipeline/latest/Json-Decode-Pipeline#optional) where necessary.

[json2elm](http://json2elm.org/) can generate pipeline-style decoders from
raw JSON and convert from other/older styles of decoders.

## Syntax

### Use `case..of` over `if` where possible

`case..of` is clever as it will generate more efficent JS, and it also allows you to catch unmatched patterns at compile time. It's also cheap to extend this data with something more useful later on, like if you need to add another branch. This saves code diffs.


## Consider refactoring if...

### ...a module has a looong list of imports

Having complicated imports hurts our compile time! I don't know what to say about this other than if you feel that there's something wrong with the top 40 lines of your module because of imports, then it might be time to move things out into another module. Trust your gut.

### ...a function can be pulled outside of a let binding

Giant let bindings hurt readability and performance. The less nested a function, the less functions are used in generated code.

The update function is especially prone to get longer and longer: keep it as small as possible. Smaller functions that don't live in a let binding are more reusable.

### ...your application has too many constructors for your Msg type

Large `case..of` statements hurts compile time. It might be possible that some of your constructors can be combined, for example `type Msg = Open | Close` could actually be `type Msg = SetOpenState Bool`


## Tooling

### Use [`elm-ops-tooling`](https://github.com/NoRedInk/elm-ops-tooling) to manage your projects

In particular, use `elm_deps_sync` to keep your main elm-package.json in sync with your test elm-package.json.

### Use [`elm-format`](https://github.com/avh4/elm-format) on all files

We run the latest version of [`elm-format`](https://github.com/avh4/elm-format) to get uniform syntax formatting on our source code.

This has [several benefits](https://github.com/avh4/elm-format#elm-format),
not the least of which is that it renders many potential style discussions moot,
making it easier to spend more time building things!