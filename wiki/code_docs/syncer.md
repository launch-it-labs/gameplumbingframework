---
title: "Syncer Code Doc"
---

`Syncer` enables syncing to SOs as well as sending messages to them.

`Syncer` objects are generally created within a controller or a test script. You will create a single `Syncer` in your `AppController`, however you can create multiple `Syncer` objects in tests to simulate interactions between players.

## `Namespace: GPF`

---

`CreateSyncer`

Provides a reference to `Syncer`.

```csharp
public static Syncer CreateSyncer()
```

For example

```csharp
syncer = Syncer.CreateSyncer();
```

---

`Sync`
Returns a reference to the SO that is automatically kept insync with the backend.

```csharp
public async Task<T> Sync<T>(SOID<T> soID) where T : RawServerObject
```

Example

```csharp
player = await syncer.Sync(playerID);
```

---

`PreSync`
Start syncing porcess on multiple server objects as a possible optimization to syncing objects in sequence. You can get a reference to the server object with Syncer.Sync

```csharp
public void PreSync(params SOID[] soIDs)
```

Example

```csharp
syncer.PreSync(hello1SOID,hello2SOID);
```
---

`Send`

Send messages to SOs through the send method.

```csharp
Send(SOID soID, ServerObjectMessage message)
```

Example:

```csharp
syncer.Send(player, new CoinPlayerSO.Flip());
```

---

`SendWait`

Like `Send` but blocks until message consumed and processed.

```csharp
 public async Task SendWait(SOID soID, ServerObjectMessage message)
```

Example

```csharp
await syncer.SendWait(player, new CoinPlayerSO.GetTopScores());
```

`SendWait` is helpful when sending messages that will only be sent once per session, for example during initialization or testing because it saves you from setting up a listener. In either case, you need to make sure that the SO Handler for the message you are sending is marked with [FromClient], like so:

```csharp
[FromClient]
void Handler(GetTopScores message)
{
    Send(
        Registry.GetId<CoinLeaderboardSO>("0"),
        new CoinLeaderboardSO.GetTopScores()
        );
}
```

`Syncer` supports a series of "WaitFor" methods that allow us to wait for a field to have a particular value, or, more broadly, evaluate as true given an operation. These methods are particularly useful when writing unit tests. They throw a `TimeoutException` if the field doesn't change within the timeout.

| Method Signature                                                                                                                                    | Notes                                                                         |
| --------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| `public async Task WaitFor(RawServerObject so, string fieldname, IFieldOperation operation)`                                                        | The timeout will default to 15s.                                              |
| `public async Task WaitFor(RawServerObject so, string fieldname, IFieldOperation operation, TimeSpan timeout)`                                      | You configure your own timeout                                                |
| `public async Task WaitFor(RawServerObject so, string fieldname, IFieldOperation operation, CancellationToken cancellationToken)`                   | You can cancel the task through code using the cancelation token (no timeout) |
| `public async Task WaitFor(RawServerObject so, string fieldname, IFieldOperation operation, TimeSpan timeout, CancellationToken cancellationToken)` | Configure a timeout and set a cancelation token.                              |

Example

```csharp
// Wait for the username to have been changed to what we requested
await syncer.WaitFor(me, nameof(CoinPlayerSO.username), FieldIs.Equal(username));
```

```csharp
// Wait for the coin flip to resolve to heads or tails
await syncer.WaitFor(
        me,
        nameof(CoinPlayerSO.state),
        FieldIs.EqualToItemIn(CoinPlayerSO.CoinState.HEADS, CoinPlayerSO.CoinState.TAILS)
    );
```
---

`Alias`

Creates a link to an SO in the data store.

```csharp
 public void Alias(SOID soID, string alias)
```

Example

```csharp
syncer.Alias(table, "table");
```
Creates a link to an SO field in the data store.

```csharp
public void Alias(SOID soID, string field, string alias)
```

Example

```csharp
syncer.Alias(table, "game", "table_game");
```
---

`RemoveAlias`

Removes link a link in the datastore

```csharp
public void RemoveAlias(string alias)
```

Example

```csharp
  syncer.RemoveAlias("table");
```
---

`Disconnect`

Disconnects the connection between `Syncer` and the backend so the backend isn't unnecessarily overburdened.

```csharp
 public async Task Disconnect()
```

Example

```csharp
private async void OnDestroy()
{
    if (syncer != null)
    {
        await syncer.Disconnect();
    }
}
```
