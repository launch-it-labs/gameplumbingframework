# Game Plumbing Framework (GPF)

**Game Plumbing Framework (GPF)** is a server-authoritative framework for building games and high-performance, real-time applications in C#. It empowers developers with a backend infrastructure and a reactive user interface, allowing them to focus on game logic by inheriting the infrastructure. We call this coding style **"infraless."** 

GPF has integrated support with Unity but can be adapted to work with other C# frontends such as Godot Mono, Stride3D, Unreal with UnrealCLR, Blazor (WebAssembly), Xamarin (MAUI), and Avalonia.

## Key Components of GPF

1. **ServerObject:** A server-authoritative construct that manages a portion of the game state.
2. **DataStore:** A global data store where all renderable data resides on the client. GPF automatically synchronizes the DataStore with the world state determined on the server.
3. **ViewBindings:** Highly reusable UI components that subscribe to and draw data from the DataStore.
4. **Test Framework:** A robust framework for developing and testing ServerObjects.

## ServerObjects Primer

<p align="center">
  <a href="https://www.youtube.com/watch?v=IFlMi8HI8vE" target="_blank">
    <img src="https://img.youtube.com/vi/IFlMi8HI8vE/0.jpg" alt="Server Development Video">
  </a>
</p>

Building a game can be broken down into two fundamental responsibilities:

- **Managing Game State**
- **Managing UI**

Within a synchronous, server-authoritative architecture, the sequence can be further broken down:

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
All the "backend" functionality is encapsulated in a construct we call a `ServerObject` (SO). When you extend `ServerObject`, you create an object that manages a portion of the game state.

### Examples of ServerObjects

Some components you might write as an SO include:

- **Player Account**
- **Leaderboard**
- **Matchmaker**
- **Match**

The implementation of your SO determines what state it manages and how that state is updated. To customize an SO, you need to:

- Define the SO fields representing the state you want to manage.
- Handle messages by updating those fields, sending messages to other SOs, or both.

Once the SO is functioning, our `ViewBindings` let you easily connect SO fields to the UI, often without needing to write additional code.

### Example: CoinLeaderboardSO

The `CoinLeaderboardSO` ([Single-player+leaderboard demo](https://github.com/launch-it-labs/gameplumbingframework/wiki/leaderboard_walkthrough)) was developed and tested in about 2 hours, requiring only 50 lines of code.

In this example, the state consists of a `Dictionary` called `scores`, which GPF automatically persists and syncs to all relevant clients. The `Handler` functions update the state and/or send messages.

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

[Get Started Now!](https://github.com/launch-it-labs/gameplumbingframework/wiki/Quick-Install)
