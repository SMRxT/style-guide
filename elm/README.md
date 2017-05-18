# Elm Style Guide

Inspired by and based on NoRedInk's [Style Guide](https://github.com/NoRedInk/elm-style-guide/blob/master/README.md)

## Table of Contents

* [How to Namespace Modules](#how-to-namespace-modules)
* [How to Structure Modules for a Page](#how-to-structure-modules-for-a-page)
* [Project Structure for an SPA](#project-structure-for-an-spa)
* [Ports](#ports)
* [Model](#model)
* [Naming](#naming)
* [Function Composition](#function-composition)
* [Syntax](#syntax)
* [Consider refactoring if...](#Consider-refacotring-if...)
* [General Guidelines](#general-guidelines)
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

#### Examples
- Paging

## How to Structure Modules for a Page

### Smaller Pages

- [Page].elm

Just go ahead and grow the page inside a single file. For smaller pages, 
it makes perfect sense to just keep the `Model`, `Msg`, `update`, and `view` all nearby.
When it starts to feel unweildy and hard to find pieces of the page, break it out into
multiple pages.

### Larger pages

Our large Elm pages generally take this form:

- Model.elm
- Update.elm
- View.elm
- Main.elm (when using a polyglot)
- Flags.elm (for non-SPAs, can include in the Main)

Inside **`Model.elm`**, we contain the actual model for the view state of our program. Note that we generally don't include non-view state inside here, preferring to instead generalize things away from the view where possible. For example, we might have a record with a list of assignments in our `Model` file, but the assignment type itself would be in a module called `Data.Assignment`.

**`Update.elm`** contains our update code. This includes the `Msg` types for our view. Inside here most of our business logic lives.

Inside **`View.elm`**, we define the view for our model and set up any event handlers we need.

**`Main.elm`** is our entry file. Here, we import everything from the other files and actually connect everything together.

It calls `Html.programWithFlags` with:
- `init`, defined in `Main`, runs `Flags.decodeFlags` and turns the resulting `Flags` type into a `Model`.
- `update` is `Update.update`.
- `view` is `View.view`.
- `subscriptions`, defined in `Main`, contains any subscriptions this app relies on.

Additionally we setup ports for interop with JS in this file.

When building an SPA, there will only be a single `Main` and it will have a [slightly different role](#building-an-spa).

**`Flags.elm`** contains a decoder for the flags of the app. We aim to keep our decoders basic and so decode into a special `Flags` type that mirrors the structure of the raw JSON instead of the structure of the `Model` type. The `Flags` and `Model` modules should not depend on each other.

To summarize:

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

- Main.elm
    - Our entry point. Decodes the flags, creates the initial model, calls `Html.programWithFlags` and sets up ports.
    - Compile target for `elm-make`
    - Imports `Model`, `Update`, `View` and `Flags`.


- Flags.elm
    - Contains the flags decoder
    - Imports nothing but generalized decoders.
    - Exports `Flags`, `decodeFlags : String -> Result String Flags`

## Project Structure for an SPA

SPAs break the usual project structure. In general, we folow many of the 
principles from the [Real World Example](https://github.com/rtfeldman/elm-spa-example).
We handle a few things differently...


## Ports

### Ports should always have documentation

I don't want to have to go out from our Elm files to find where a port is being used most of the time. Simply adding a line or two explaining what the port triggers, or where the values coming in from a port can help a lot.


## Model

### Model shouldn't have _any_ view state within them if they aren't tied to views

For example, an assignment should not have a `openPopout` attribute. Doing so means we can't use that type again in another situation.


## Naming

### Use descriptive names instead of using underscores

Instead of this:

```elm
-- Don't do this --
updateModel model value =
  let
    model_ =
      { model | setting = value }
  in
    model_
```

...give it a meaningful name:

```elm
-- Instead do this --
updateModel model value =
  let
    updatedModel =
      { model | setting = value }
  in
    updatedModel
```

## Function Composition

### Prefer forward application [`|>`](http://package.elm-lang.org/packages/elm-lang/core/latest/Basics#|>) "pipelines" over backward application

Instead of this:
```elm
-- Don't do this --
Maybe.withDefault "default" <| Maybe.map String.toUpper <| patient.alias
```
...use pipelines:
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

...prefer using parentheses:

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

### Always use [`Json.Decode.Pipeline`](https://github.com/NoRedInk/elm-decode-pipeline)

Even though this would work...

```elm
-- Don't do this --
algoliaResult : Decoder AlgoliaResult
algoliaResult =
  succeed AlgoliaResult
    |: ("id" := int)
    |: ("name" := string)
    |: ("address" := string)
    |: ("city" := string)
    |: ("state" := string)
    |: ("zip" := string)
```

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

## General Guidelines

### Only expose types, not functions

Use fully-qualified names when calling functions. This helps with name collisions and it helps with navigation.

## Tooling

### Use [`elm-ops-tooling`](https://github.com/NoRedInk/elm-ops-tooling) to manage your projects

In particular, use `elm_deps_sync` to keep your main elm-package.json in sync with your test elm-package.json.

### Use [`elm-format`](https://github.com/avh4/elm-format) on all files

We run the latest version of [`elm-format`](https://github.com/avh4/elm-format) to get uniform syntax formatting on our source code.

This has [several benefits](https://github.com/avh4/elm-format#elm-format),
not the least of which is that it renders many potential style discussions moot,
making it easier to spend more time building things!