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

### `Views.`
`Views.Form`, `Views.DailyCalendar`, `Views.ExplainerWell`

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

Something reusable that we might open source, that aren't tied directly to any SMRxT stuff. Give it a name that would make sense for the package as if we have already open-sourced it.

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

Don't break out large files into Model, Update, and View gratuitously. They should exist together in one file. Decompose problems on the boundaries of types. If a particular type has operations around it, break those out into a separate file. Large Elm files aren't as unwieldy as large files in other languages.


#### Resources
[Evan - Life of a file](https://www.youtube.com/watch?v=XpDsk374LDE&feature=youtu.be)


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

### JSON decoders for the type of the file name (ie, the decoder for a Device in Types.Device) should just be named decoder. Helper decoders and other decoders in the file should begin with the type being decoded followed by `decoder`.

```
...Types.Device...
decoder : Decoder Device

attributesDecoder : Json.Decoder Attributes
```

### Do not create decoders with pluralized names for decoding lists, the calling function should explicitly use Json.list.

Instead of this:

```elm
-- Don't do this --
devicesDecoder : Json.Decoder (List Device)
devicesDecoder =
    Json.List ...
```

...do this:

```elm
-- Instead do this --
deviceDecoder : Json.Decoder Device
deviceDecoder =
    ...

...
    |> required "list_of_devices" (Json.list deviceDecoder)
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

### Always use [`Json.Decode.Pipeline`](https://package.elm-lang.org/packages/NoRedInk/elm-json-decode-pipeline/latest)

Even though this would work...

```elm
-- Don't do this --
algoliaResult : Decoder AlgoliaResult
algoliaResult =
  succeed AlgoliaResult
    |> (field "id" int)
    |> (field "name" string)
    |> (field "address" string)
    |> (fied "city" string)
    |> (field "state" string)
    |> (field "zip" string)
```

Instead do this from the start:

```elm
-- Instead do this --
import Json.Decode.Pipeline exposing (required)

algoliaResult : Decoder AlgoliaResult
algoliaResult =
  succeed AlgoliaResult
    |> required "id" int
    |> required "name" string
    |> required "address" string
    |> required "city" string
    |> required "state" string
    |> required "zip" string
```


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
