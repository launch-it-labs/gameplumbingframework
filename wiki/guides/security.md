---
title: "Guide to Security"
---

### [FromClient]

By default, `ServerObject Handlers` will not receive messages from Clients (so they can’t be hijacked). To allow a `Handler` to receive messages from the Client apply the `[FromClient]` attribute to that handler. Sanitize all input from clients. If all messages from `[FromClient]` handlers are properly sanitized, you can trust any other messages from other SO's.

### [Syncable]

By default, `ServerObject` state can not be accessed by a client; however, you can give clients access to state with the `[Syncable]` attribute.

The `ServerObject` below is an example of `[Syncable]` and provides a few examples of `[FromClient]`:

```csharp
[DataStorePath("sync.coin_players")]
[Syncable]
[Register("coin_player")]
public class CoinPlayerSO : ServerObject
{
    public const int FLIP_SECONDS = 1;

    public override SOID ID { get; set; }

    public enum CoinState
    {
        HEADS,
        TAILS,
        FLIPPING
    }
    public CoinState state;

    public int currentStreak = 0;

    public int longestStreak = 0;

    public string username = "Anonymous";

    public Tuple<string,int>[] leaderboard;

    public class SetUsername : ServerObjectMessage
    {
        public string username;
    }
    [FromClient] // <- [FromClient] allows a user to send this type of message
    void Handler(SetUsername message)
    {
        username = message.username;
        Send(
            Registry.GetId<CoinLeaderboardSO>("0"),
            new CoinLeaderboardSO.SendScore {
                score = longestStreak,
                username = username }
            );
    }

    public class Flip : ServerObjectMessage {}
    [FromClient] // <- [FromClient] allows a user to send this type of message
    void Handler(Flip message)
    {
        if (state != CoinState.FLIPPING)
        {
            state = CoinState.FLIPPING;
            Random rand = new Random();
            var isHeads = rand.Next(2) == 0;
            Send(ID, new FlipResult { isHeads = isHeads }, FLIP_SECONDS);
        }
    }

    public class GetTopScores : ServerObjectMessage { }
    [FromClient] // <- [FromClient] allows a user to send this type of message
    void Handler(GetTopScores message)
    {
        Send(
            Registry.GetId<CoinLeaderboardSO>("0"),
            new CoinLeaderboardSO.GetTopScores()
            );
    }

    public class FlipResult : ServerObjectMessage
    {
        public bool isHeads;
    }
    // This handler does not have [FromClient] so only the server can send it
    void Handler(FlipResult message)
    {
        if (message.isHeads)
        {
            currentStreak++;
            state = CoinState.HEADS;
        }
        else
        {
            currentStreak = 0;
            state = CoinState.TAILS;
        }
        if (currentStreak > longestStreak)
        {
            longestStreak = currentStreak;
            Send(
                Registry.GetId<CoinLeaderboardSO>("0"),
                new CoinLeaderboardSO.SendScore {
                    score = longestStreak,
                    username = username }
                );
        }
    }

    // This handler does not have [FromClient] so only the server can send it
    void Handler(CoinLeaderboardSO.TopScores message)
    {
        leaderboard = message.scores;
    }

    public override string ToString()
    {
        string s = $"CoinPlayer: {username} currentStreak: {currentStreak} currentStreak: {longestStreak}";
        return s;
    }
}
```

### Encrypted IDs.

Using encrypted IDs, Client's are prevented from accessing an SO (either syncing to it or messaging it) unless the Client have a direct knowledge of that SO's ID. For example, if a client recieves an authenticated ID for it's `PlayerSO`, it can access that SO, but if it learns of a `PlayerSO's` ID from a `LeaderboardSO` or `LobbySO`, it will not be able to use that ID to access the playerSO.

### OAuth

While EncryptedIDs provide an out of the box minimal layer of security, it is highly recommended that you leverage GPF's OAuth integration.

