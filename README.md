Game Plumbing Framework (GPF) is a server-authoritative framework for building games as well as any other high performance, realtime app in C#. It empowers game and app developers with a backend infrastructure and a reactive user interface so devs can focus on game logic by inheriting the infrastructure.  We call this style of coding "infraless". GPF has integrated support with Unity but should be able to be made to work with other C# frontends (eg. Godot Mono, Stride3D, Unreal with UnrealCLR, Blazor (WebAssembly), Xamarin (MAUI), Avalonia)

GPF is composed of:

1. A server authoritative construct called ServerObject
2. A global data store where all render-able data lives in the client called the DataStore that GPF automatically keeps in sync with the world state (which is determined on the server).
3. Highly reusable UI components that subscribe to and draw data from the datastore, called ViewBindings.
4. A test framework for developing ServerObjects.

# ServerObjects Primer

<center>
  <Video videoTitle="server_development" youtubeID="IFlMi8HI8vE" />
</center>

Building a game can be broken down into two fundamental responsibilities:

- Managing game state
- Managing UI

Within a synchronous, server authoritative architecture, the sequence can be broken down further:

```mermaid
    graph TD;
    subgraph Client
        id1([1.UI draws game state])
        --> id2([2.UI produces update])
        --> id3([3.Update sent to backend])
        end
    subgraph Backend
        id3
        --> id4([4.Update received])
        --> id5([5.Game state updated])
        --> id6([6.Game state persisted])
        --> id7([7.Game state pushed to clients])
  end
```

We have packed up all of the "backend" functionality in a construct we call a `ServerObject`, or `SO`.

When you extend `ServerObject` you are creating an object that manages a portion of the game state.

Examples of components you might write as an `SO` include:

- The player's account
- A leaderboard
- A matchmaker
- A match

The implementation of your `SO` determines what state it encompasses and how that state gets updated. To customize how this works for your SO, you need to:

- Give the SO fields that represent the state you want to manage
- Handle messages by updating those fields, sending messages to other `SOs`, or both

Once the SO is working, our `ViewBindings` let you simply hook the `SOs` fields up to UI, usually without needing to write any extra code.

`SO's` are simple to write. Our `CoinLeaderboardSO` below ([see Single-player+leaderboard demo](https://github.com/launch-it-labs/gameplumbingframework/wiki/leaderboard_walkthrough) was developed and tested in about 2 hours and only required 50 lines of code.

Here state is comprised of a `Dictionary` called `scores`. `scores` will be persisted and synced to all relevant clients for you automatically. Notice the functions named `Handler` handle messages by updating state and/or sending messages.

```csharp
[Register("coin_leaderboard")]
public class CoinLeaderboardSO : ServerObject
{
    public const int TOP_SCORE_RETURN_LENGTH = 3;

    public override SOID ID { get; set; }

    public Dictionary<string, Tuple<string, int>> scores = new Dictionary<string, Tuple<string, int>>();

    public class TopScores : ServerObjectMessage
    {
        public Tuple<string, int>[] scores;
    }

    public class SendScore : ServerObjectMessage
    {
        public string username;
        public int score;
    }
    void Handler(SendScore message)
    {
        scores[message.evt.source] = new Tuple<string, int>(message.username, message.score);
        var topScores = FindTopScores();
        Send(message.evt.source, new TopScores { scores = topScores });
    }

    public class GetTopScores : ServerObjectMessage {}
    void Handler(GetTopScores message)
    {
        var topScores = FindTopScores();
        Send(message.evt.source, new TopScores { scores = topScores });
    }

    Tuple<string, int>[] FindTopScores()
    {
        var count = TOP_SCORE_RETURN_LENGTH < scores.Count ? TOP_SCORE_RETURN_LENGTH : scores.Count;

        var topScores = scores.OrderByDescending(entry => entry.Value.Item2).Take(count).ToArray();
        var result = new Tuple<string, int>[count];
        for (int i = 0; i < topScores.Length; i++)
        {
            result[i] = topScores[i].Value;
        }
        return result;
    }
}
```

# Conclusion 

When writing SOs, developers focus can focus on server authoritative logic because GPF automatically handles: 

- Persistence
- Syncing state to subscribed clients
- Update UI as state changes

GPF also makes developing SOs easy in a number of ways:

- `MockCloud` enables `SOs` to be developed within Unity and without waiting for deploys
- `MockCloud` enables the debugger to oversee the full stack, so breakpoints in your `ServerObjects` can be triggered from gameplay
- `SOs` can be tested independently
- Auto-generate diagrams to illustrate how they are interact
- `SO's` can be deployed to a serverless AWS backend through the Unity Editor
- UI inspector that uses autocomplete to find game state

Finally, GPF handles deployment for you by including a stack management panel from inside of Unity.  Developers can easily spin up a stack for their branch and test multiplayer functionality with others.

[Get Started Now!](https://github.com/launch-it-labs/gameplumbingframework/wiki)
