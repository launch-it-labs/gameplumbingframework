# Game Plumbing Framework (GPF)

**Game Plumbing Framework (GPF)** is a server-authoritative framework for building games and high-performance, real-time applications in C#. It is the only technology that provides developers a fully extensible tripple A backend.  The backend is scalable, distributed, low latency, consistentent, and our middle where provides client side prediction.  These features are under the hood, so the developer can benifit from them without even thinking about it. GPF allows developers to focus on game logic while inheriting the infrastructure. We call this coding style **"infraless."** and it typically results in a reduction of the code by 80%, thereby significantly reducing engineering cost, time to market, and code complexity.

Every backend built on top of GPF will automatically be consistent, be horizontally scalable, with all communication being low latency.  Building a backend that accomplishes just 2 of these 3 requirements costs over a million dollars, but GPF achieves all three requirements while providing an easy way to tap into world data as well as robust testing tools.

GPF has integrated support with Unity but can be adapted to work with other C# frontends such as Godot Mono, Stride3D, Unreal with UnrealCLR, Blazor (WebAssembly), Xamarin (MAUI), and Avalonia.

GPF is not a BAAS; you host your data whereever you want: we just made the whole process as easy as possible.

## Key Components of GPF

1. **ServerObject:** A server-authoritative construct that manages a portion of the game state.
2. **DataStore:** A global data store where all renderable data resides on the client. GPF automatically synchronizes the DataStore with the world state determined on the server.
3. **ViewBindings:** Highly reusable UI components that subscribe to and draw data from the DataStore.
4. **Test Framework:** A robust framework for developing and testing ServerObjects.

## ServerObjects Primer
<!--
@TODO: Update Video
_Click the image below to watch the ServerObject primer video on YouTube._
[![Server Development Video](https://img.youtube.com/vi/IFlMi8HI8vE/0.jpg)](https://www.youtube.com/watch?v=IFlMi8HI8vE)
-->

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

The `CoinLeaderboardSO` is an example of a leaderboard used in our Coin Flipper game.

In this example, the state consists of a `Dictionary` called `scores`, which GPF automatically persists. The `Handler` functions handle messages and do so by updating the state and/or sending messages.

```csharp
// This class is not [Syncable] because the client should not have a copy of this class.
// Even with the id, users cannot access its contents
// No Handlers will have the [FromClient] on them
[Register("coin_leaderboard")]
public class CoinLeaderboardSO : ServerObject
{
    [ExpandData]
    public class LeaderboardRow
    {
        public string username;
        public int score;
    }

    // Use the same SOID to create a singleton
    public static readonly SOID<CoinLeaderboardSO> MainSoid = new("main");

    public const int TOP_SCORE_RETURN_LENGTH = 3;

    public const float MAX_UPDATE_RATE = 0.3f;

    public Dictionary<SOID<CoinPlayerSO>, LeaderboardRow> scores = new();

    public bool scoresChanged;

    public DateTime lastSendTime;

    public bool waitingForRefresh;

    public class SendScore : ServerObjectMessage, ITimeStampReceiver
    {
        public string Username { get; set; }
        public int Score { get; set; }

        public DateTime timestamp { get; set; }
    }

    // Add or update a user's score and username
    private void Handler(SendScore message)
    {
        this.scores[message.Source.GetSOID()] = new LeaderboardRow { username = message.Username, score = message.Score };
        this.scoresChanged = true;
        this.SendScoresToView(message.timestamp);
    }


    public class Refresh : ServerObjectMessage, ITimeStampReceiver
    {
        public DateTime timestamp { get; set; }
    }
    // Send the scores to the view after a delay, this allows for rate limiting and prevents flooding users with updates
    private void Handler(Refresh message)
    {
        this.waitingForRefresh = false;
        this.UpdateView(message.timestamp);
    }

    /// <summary>
    /// This SO is not syncable. Players see the scores through a separate, syncable, view SO
    /// This allows us to rate limit the updates that get sent to players
    /// It also allows us to keep player secret data hidden
    /// </summary>
    private void SendScoresToView(DateTime timestamp)
    {
        // Limit the rate that we update the viewable LeaderBoard to conserve bandwidth
        var secondsSinceLastUpdate = (timestamp - this.lastSendTime).TotalSeconds;
        if (secondsSinceLastUpdate >= MAX_UPDATE_RATE)
        {
            this.UpdateView(timestamp);
        }
        else
        {
            this.RefreshAfterDelay();
        }
    }

    /// <summary>
    /// Calculated the latest top scores and update all the player's top scores views
    /// </summary>
    private void UpdateView(DateTime timestamp)
    {
        // Send the top scores to the view
        var topScores = this.FindTopScores();
        var topScoresSoid = new SOID<CoinTopScoresSO>(this.Id);
        this.Send(topScoresSoid, new CoinTopScoresSO.SetTopScores { Scores = topScores });
        this.lastSendTime = timestamp;
        this.scoresChanged = false;
    }

    /// <summary>
    /// To prevent flooding, we want to limit how frequently we update the top score view
    /// Schedule a refresh for later, or wait for the refresh if it is already scheduled
    /// </summary>
    private void RefreshAfterDelay()
    {
        if (!this.waitingForRefresh && this.scoresChanged)
        {
            this.Send(this.Id, new Refresh(), MAX_UPDATE_RATE);
            this.waitingForRefresh = true;
        }
    }

    private LeaderboardRow[] FindTopScores()
    {
        var count = TOP_SCORE_RETURN_LENGTH < this.scores.Count ? TOP_SCORE_RETURN_LENGTH : this.scores.Count;

        var topScores = this.scores.OrderByDescending(entry => entry.Value.score).Take(count).ToArray();
        var result = new LeaderboardRow[count];
        for (var i = 0; i < topScores.Length; i++)
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
- Auto-generate diagrams to illustrate how they interact
- `SOs` can be deployed to a serverless AWS backend through the Unity Editor
- UI inspector that uses autocomplete to find game state

Finally, GPF handles deployment for you by including a stack management panel from inside of Unity.  Developers can easily spin up a stack for their branch and test multiplayer functionality with others.

[Jump on our Discord](https://discord.gg/bvPwWrwPDq)

[Component Overview](https://github.com/launch-it-labs/gameplumbingframework/wiki/Major-GPF-Components/)

[Quick Start](https://github.com/launch-it-labs/gameplumbingframework/wiki/Quick-Start-part-1)

