---
title: "DataStore Code Doc"
---

The `DataStore` allows for reliable and consistent access to app state by UI elements. It is designed to make UI implementation easy and reliable. It decouples the the UI from the code, and allows data to be accessed with a simple path. GPF automatically stores `ServerObjects` state and static data inside the `DataStore`. Additional app state can be stored in the `DataStore` to take advantage of all the benefits the `DataStore` can provide.

Key Features:

- Binding to variables
- Data is always ready
- Automatic type conversion
- Variable name aliasing

### Binding to Variables

Every piece of data in the `DataStore` is given change events for when that piece of data is changed or updated. This allows scripts to bind to a piece of data, and get notified when any change is made. This keeps the UI in sync with the app state at all times.

### Data is Always Ready

The `DataStore` eliminates common race conditions that occur when game objects and scripts are initialized before the app has completed loading. Often this can occur on a scene load, when Unity runs start and awake calls. The `DataStore` will always return a non-null value when a variable is read. This makes code more simple by eliminating the need for null checks. And it removes the order dependency between when a UI element can be created and when the data for the app is loaded and ready.

### Automatic Type Conversion

The `DataStore` will convert data to the desired type automatically when it is read, regardless of the data type that is being stored. This enables reused of UI components. For example, a simple script to display a string on the screen can be used on any variable, because the `DataStore` will turn ints, bools, DateTimes, etc to strings without the script needing to know the original data type.

### Variable Name Aliasing

Variables are stored at a path within the `DataStore`. However, sometimes the same piece of data needs to be accessed from multiple different locations. The `DataStore` has the ability to link variables, and link parts of the path, in order to make access simpler. An example of this would be if a user is controlling 3 units: units.marine, units.robot, units.cyborg. But at any given moment, they can only select 1 unit. The UI can be constructed to show all the data from units.current. If the user selects the marine, we just need link units.current to units.marine, and the UI will automatically show the values for marine.

## `Namespace: GPF`

---

`Instance`

Provides a reference to `DataStore` singelton.

```csharp
public static DataStore Instance
```

---

`DataStore`

Constructor. Provides a named instance to `DataStore`

```csharp
public DataStore(string name)

```

---

`Set`

Store a piece of data

```csharp
    public void Set(string key, Object value)
```

Store function backed data. Reading data at this key will execute the function and return the result

```csharp
   public void Set<T>(string key, Func<T> value)
```

---

`Get`

Get a piece of data

```csharp
 public T Get<T>(string key)
```

```csharp
  SingleVar Get(Address path)
```

---

`Link`

Redirect access to a variable (and all children) to another part of the DataStore

```csharp
  public void Link(string source, string alias)
```

---

`Unlink`

Remove a link created at the target

```csharp
  public void Unlink(string alias)
```

---

`AddListener`

Subscribe to changes in this value.

The action will be called any time the variable is set or notified

init: Should the action be run immediately to initialize your class

```csharp
  public void AddListener(string key, Action listener, bool init = true)
```

The action will be called any time the variable is set or notified and passed the latest value

```csharp
   public void AddListener<T>(string key, Action<T> listener, bool init = true)
```

```csharp
   public void AddListener<T>(string key, Action<T, T> listener, bool init = true)
```

---

`HasListener`

Check if a specific listener has been added

```csharp
  public bool HasListener(string key, Action listener)

```

```csharp
    public bool HasListener<T>(string key, Action<T> listener)

```

```csharp
   public bool HasListener<T>(string key, Action<T, T> listener)

```

---

`RemoveListener`

Unsubscribe to changes in this variable

```csharp
 public void RemoveListener(string key, Action listener)
```

```csharp
 public void RemoveListener<T>(string key, Action<T> listener)
```

```csharp
 public void RemoveListener<T>(string key, Action<T, T> listener)
```

---

`Notify`

Run all listeners associated with this variable manually.
It is not necessary to run Notify() on most data. Listeners will be run automatically any time the variable is changed.
This can be useful to run listeners on function backed data.

```csharp
 public void Notify(string key)
```

---

`Delete`

Reset variable value and mark for deletion. The variable will be completely removed once all the listeners have unsubscribed

```csharp
 public void Delete(string key)
```

---

`DeleteTree`

Recursiveley reset and mark all variables under that path for deletion. The variables will be completely removed once all the listeners have unsubscribed

```csharp
  public void DeleteTree(string key)
```

---

`ToString`

```csharp
public override string ToString()
```
