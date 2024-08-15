---
title: "Guide to Provided ViewBindings"
---

import Video from "../../components/Video";

`ViewBindings` are UI components you can add to game objects on the `Hierarchy`. Every `ViewBinding` has a `variable` field on the `Inspector` panel that allows you to bind into any variable managed by the `DataStore`.

`ViewBindings` are responsible for updating UI automatically when the `variable` value changes. With the exception of `ListVB` all of the other `ViewBindings` expect `variable` to be cast-able to string.

Many `ViewBindings` requires another component to also be attached to the same `GameObject` because the `ViewBinding` will set the value of the component based on the `variable` field. We refer this as the "Companion" in the table below.

### ViewBindings Descriptions

`ViewBindings` provided by GPF are described below and live in the namespace `GPF.UI`

---

`TextVB`

Displays `variable` value inside Text Component. Also supports replacements with {} notation, so {0} will be replaced by the first "replacement", which is essentially another string `variable`.

---

`TextInputVB`

Displays `variable` value.

---

`ImageVB`

Displays an image based on a `variable` value representing the SpriteID.

---

`SelectChildVB`

Turns on one child at a time, based on the `variable` value. Ideal use cases include Menus and showing/hiding elements depending on the state. A feature of `SelectChildVB` is that if you an animation controller with `onEnter` and `onExit` animations for a child, then those animations will be played during state transitions. You can also easily create such a controller by right clicking inside `Assets` and going to `Create -> Game Plumbing Framework -> SelectChildVB Transition`.

---

`AnimationVB`

Controls the animations on a single game object. The value of the `variable` is directly passed to the `AnimationController` on that object in the form of a bool with the value as the name. For example, if the value of the variable in the DataStore is "highlight" then the AnimationVB will set "highlight" to true in the Animation Controller on the object. It will also set all other variables (not "highlight") to false. This is useful for any non-menu that needs to animate.

---

`ListVB`

Displays a list of objects. Maps properties of each item in list to a UI instance based on it's child. This is the only component that expects variable to be IEnumerable<string\>.

---

| VB Name       | Companion                      |
| ------------- | ------------------------------ |
| TextVB        | UnityEngine.UI.Text            |
| AnimationVB   | UnityEngine.Animator           |
| TextInputVB   | UnityEngine.UI.InputField      |
| ImageVB       | UnityEngine.UI.Image           |
| SelectChildVB | (optional)UnityEngine.Animator |
| ListVB        | GamePlumbing.Views.PropMapper  |

### Examples

This video demonstrates how `TextVB`, `AnimationVB` and `TextInputVB` are used to create our `Single-Player / Leaderboard` Demo Scene:

<center>
<Video videoTitle="leaderboardUI" youtubeID="XzPUo1-S0mc" />
</center>

This video demonstrates how `ListVB`, `ImageVB`, and `SelectChildVB` are used to create our `Hello Content` Demo Scene:

<center>
<Video videoTitle="hellocontentUI" youtubeID="UclNLCdEd90" />
</center>

This video provides a demonstration of how `SelectChildVB` manages the states and state transitions for our `Matchmaker` Demo:

<center>
<Video videoTitle="SelectChildVB" youtubeID="H6fzEYycHzA" />
</center>

### PropMapper

To use `ListVB`, you will need to add a `PropMapper` component to the GameObject `ListVB` is added to. Adding `Propmapper` will make a copy of the the GameObjects child for every element of the collection passed through the `props` field .

For example, we might want to display certain fields for our "characters" in a menu, so we create a menu, add a `ListVB` to it, and set the `variable` to "content.characters".

Then we can attach a `PropMapper` to the menu, and map "props" to "content.characters.\${id} "

`${id}` is a special notation that `PropMapper` supports by replacing `${id}` with the id for each item in the `ListVB` `variable` ( `content.characters` ).

Prop mapper will copy the child gameobject for each character, and set the `props` on each character.

### Other View Components

We have also included a couple of other view components you can attach to `MonoBehaviors`, but without binding to the `DataStore`.

| Name            | Description                                                                                   |
| --------------- | --------------------------------------------------------------------------------------------- |
| SafeArea        | Provides notch support.                                                                       |
| FlexableElement | Provides different anchors for portrait and landscape, allowing seamless editing in UI editor |
