---
title: "Guide to Building ViewBindings"
---

You can also build your own `ViewBindings`.

`ViewBindings` connect data in the `DataStore` to UI. There are a set of basic `ViewBindings` that are provided with GPF. The basic `ViewBindings` are very useful, however sometimes you will need UI for a data structure that there isn't an existing `ViewBinding` for, and it's in those instances where you will want to create your own.

This guide walks you through `LeaderboardVB`, a custom `ViewBinding` included with the [Single-Player / Leaderboard Demo](http://localhost:8000/leaderboard_walkthrough). On first glance, it might not seem nessary to create a new `ViewBinding` since a leaderboard is just an a list of objects, and `ListVB` can already handle that. However, we chose to use an unorthadox data structure for the leaderboard, specifically

```csharp
Tuple<string, int>[]
```

thus, we need to write a new `ViewBinding` that assumes `variable` is of this type.

Let's zoom in to perhaps the least obvious line of code in `LeaderBoardVB.cs`. The last line of the `Start()` function reads:

```csharp
props.AddListener<Tuple<string, int>[]>(variable, Listener);
```

Here, `props` gives us the ability to listen to the `DataStore` for changes to `variable`, and run `Listener` when those changes occur. Infact `Listener` will also be run as soon as the listener is added to `props`. The necisity for each `ViewBinding` to use `props` to gain access to the `DataStore` motivates the preceding line as well:

```csharp
 Props.Inject(this);
```

The code above causes `props` to be able to access the `DataStore` and forces our `LeaderboardVB` class to implement `IPropReceiver`.

The rest of the code is straight forward as the `Listener` function simply leverages it's intimate knowledge of the data structure used by `variables` as well as it's intimate knowledge of the UI in play ,`spots`, to set the UI approprietely.

Here is the complete `LeaderboardVB` class with comments:

```csharp

public class LeaderboardVB : MonoBehaviour, IPropReceiver
{
    // Props give this ViewBinding a unique view of data from the DataStore
    public Props props { get; set; }

    // A DataPath field enables autocomplete in the Unity Editor
    public DataPath variable;

    // The places on the leader board we want to display
    [Serializable]
    public class Spot
    {
        public Text username;
        public Text score;
    }
    public Spot[] spots = new Spot[0];

    void Start()
    {
        // Get our props. Props can be provided from parent objects in the hierarchy
        Props.Inject(this);

        // We want the UI to be updated every time the data changes
        // The change listener will be called whenever the variable value changes
        props.AddListener<Tuple<string, int>[]>(variable, Listener);
    }

    void Listener(Tuple<string, int>[] scores)
    {
        for (int i = 0; i < spots.Length; i++)
        {
            if (scores.Length > i)
            {
                spots[i].username.text = scores[i].Item1;
                spots[i].score.text = scores[i].Item2.ToString();
            }
            else
            {
                spots[i].username.text = "";
                spots[i].score.text = "";
            }
        }
    }
}
```
