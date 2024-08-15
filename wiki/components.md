---
title: "Major Components"
---

import Video from "../components/Video";

GPF has five major components: `ServerObjects`, `Syncer`, `DataStore`, `View Bindings`, and `Controllers`. This article break those components down and illustrates how they are connected.

<center>
<Video videoTitle="components" youtubeID="a0JAHUnTon4" />
</center>

Below is a diagram with GPF's major components and a table describing those components.

<center><img src ="/images/components.png" alt="Components"/></center>

| Component                                                                                                                                                                                                                                                                                         | Description                                                                                                                                                                       | Status                                                     |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| You write custom ServerObjects to manage game state with server authoritaty. ServerObjects inherit persistence and synchronicity (subscribers are kept in-sync) so you only worry about what messages they should handle and how they should handle them. [Learn more here.](/serverobject_guide) | Base `ServerObject` provided along with many examples of `ServerObjects`.                                                                                                         |
| Syncer                                                                                                                                                                                                                                                                                            | `Syncer` lets the client sync to and message `ServerObjects`. [Learn more here.](/syncer)                                                                                         | Provided within framework.                                 |
| DataStore                                                                                                                                                                                                                                                                                         | A place to throw data so it can be easily accessed. Every SO value is stored here. These values can be bound to Views with ViewBindings. [Learn more here.](/data_store)          | Provided within framework.                                 |
| ViewBindings                                                                                                                                                                                                                                                                                      | Classes that make it easy to bind `gameobjects` to information in the DataStore. [Learn more here.](/provided_view_bindings)                                                      | Several examples provided. You can build your own as well. |
| Controllers                                                                                                                                                                                                                                                                                       | Glue logic for the application or a feature. Handles initialization and houses UI event handlers. This code is game specific and will be attached to a `gameobject` on the Scene. | One example provided in each example project.              |
