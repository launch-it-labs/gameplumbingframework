---
title: "ServerObjectTest Code Doc"
---

`ServerObjectTest`

Provides a context to run unit tests that:

- Can be run from `Unity TestRunner`
- Reports an accurate stack trace
- Uses `Simulated Time` to be able to instantly run through code that would otherwise take time.

## `Namespace: GPFEditor`

---

`Run`

Override this task as the entry point of your test.

```csharp
public abstract Task Run();
```

---

`CreateSyncer`

Creates a test Syncer that uses `Simulated Time` and a `DataStore` configured for testing.

The `name` parameter is used as the name of the `DataStore` instance is used for the test.

```csharp
 protected Syncer CreateSyncer(string name)
```

### Example 

Your test class should extend `ServerObjectTest`, create and assert something.
This example is from our [HelloWorld Tutorial](../hello_world/part1).

```csharp
sealed class HelloWorldTest : ServerObjectTest
{
    public override async Task Run()
    {
        // Create a syncer
        var syncer = CreateSyncer("default");

        //get an ID
        var helloSOID =  Registry.GetId<HelloSO>();

        // get reference to helloSO
        HelloSO helloSO = await syncer.Sync(helloSOID);

        // Let's tell the SO our name so it can say hello to us.
        // Note  await syncer.SendUpdateWait doesn't return until the SO is updated.
        await syncer.SendWait(helloSO, new HelloSO.MyNameIs { name = "World" });

        // Now let's check to see things went as planned.
        Assert.That(helloSO.greeting, Is.EqualTo("Hello World") );
    }
}
```

In this example we need multiple `Syncers` to simulate a multiplayer experience,
so we use `ServerObjectTest.CreateSyncer`. Notice we use `ServerObjectTest.CreateSyncer` to simulate each player.

We did not include the code for the `PlayerActions` task. It simply runs through a script for each player. This example is from our [Matchmaker Example](../walkthroughs/matchmaker_walkthrough)

```csharp
public override async Task Run()
{
    var player1Name = "Gina";
    var player2Name = "Bob";
    var matchmakerId = Registry.GetId<MatchmakerSO>();

    var player1Task = PlayerActions(
        CreateSyncer("player1"),
        Registry.GetId<MatchPlayerSO>(),
        player1Name,
        MatchGameSO.Moves.ROCK,
        MatchPlayerSO.State.GAME_LOST,
        matchmakerId
        );

    var player2Task = PlayerActions(
        CreateSyncer("player2"),
        Registry.GetId<MatchPlayerSO>(),
        player2Name,
        MatchGameSO.Moves.PAPER,
        MatchPlayerSO.State.GAME_WON,
        matchmakerId
        );

    await player1Task;
    await player2Task;
}
```

`Delay`

Use this instead of Task.Delay because it will just call Task.Delay when `Backend Type` is set to `Cloud`, but it will run tests instantly when your `Backend Type` is set to `Simulated`.

```csharp
public async Task Delay(int milliseconds)
```

### Example 

```csharp
await Delay(SECONDS * 1000);
```
`LogAssert.Expect`

Expect a particular type and message being logged.

```csharp
public static void Expect(LogType type, string message);
public static void Expect(LogType type, Regex message);
```

### Example 

```csharp
LogAssert.Expect(LogType.Exception, "SystemException: " + message);
LogAssert.Expect(UnityEngine.LogType.Error, "Bad thing happend");
```
