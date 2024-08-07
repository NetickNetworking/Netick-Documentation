# Writing Client-Side Prediction Code

## Network Input 

Network Input describes what the player wants to do, which will be used to simulate the state of objects they want to control. This ensures that the client can’t directly change the state – the change happens by executing the input, which, even if tampered with, won’t be game-breaking.

## Defining Inputs

To define a new input, create a struct that implements `INetworkInput` interface:

```csharp
public struct MyInput : INetworkInput
{
    public NetworkBool ShootInput;
    public float       MoveDirX, MoveDirY;
}
```

### Network Input Constraints:
- Must not have class-based network collections as fields. Only `NetworkArrayStruct` variants are allowed as network input fields. 
- Must not have reference types as fields.
- Must not have `string` as a field. Instead, you can use `NetworkString` variants.

## Setting Inputs 

To set the fields of an input, you first need to acquire the input struct of the current tick, using `Sandbox.GetInput`.\
Then, you can set it inside `NetworkUpdate` on `NetworkBehaviour`:

```csharp
public override void NetworkUpdate()
{
    var input = Sandbox.GetInput<MyInput>();

    input.MoveDirX = Input.GetAxis("Horizontal");
    input.MoveDirY = Input.GetAxis("Vertical");

    Sandbox.SetInput(input);
}
```

You could also set them on `OnInput` of `NetworkEventsListener`, which is preferred.

## Simulating (Executing) Inputs

To drive the gameplay based on the input struct, you must do that in `NetworkFixedUpdate`:

```csharp
public override void NetworkFixedUpdate()
{
    if (FetchInput(out MyInput input))
    {
        // movement
        var movement = transform.TransformVector(new Vector3(input.MoveDirX, 0, input.MoveDirY)) * Speed;
        movement.y   = 0;
  	    _CC.Move(movement * Time.fixedDeltaTime);
	    // shooting
        if (input.ShootInput == true && !IsResimulating)
            Shot();
    }
}
```


> [!NOTE]
> Everything that is modified around `FetchInput`, and also affects the networked state, must be networked using `[Networked]`. For example, if you have a variable that is changing over time which can affect the speed of the player, it must be networked.

> [!NOTE]
> When `IsResimulating` equals to true, every `[Networked]` variable has an older value, since we are resimulating past ticks. The first resimulated tick will have the server state applied to every networked variable.

> [!WARNING]
> `Sandbox.GetInput` and `Sandbox.SetInput` are used to read and set the user inputs into the input struct. While `FetchInput` is used to actually use the input struct in the simulation.

FetchInput tries to fetch an input for the state/tick being simulated/resimulated. It only returns true if either:

1. We are providing inputs to this object – meaning we are the Input Source of the object.
2. We are the owner (the server) of this object – receiving inputs from the client who’s the Input Source. And only if we have an input for the current tick being simulated. If not, it would return false. Usually, that happens due to packet loss.

And to avoid the previous issue we talked about, we make sure that we are only shooting if we are simulating a new input, by checking `IsResimulating`.

## Input Source

For a client to be able to provide inputs to be used in an Object’s `NetworkFixedUpdate`, and hence take control of it, that client must be the Input Source of that object. Otherwise, `FetchInput` will return false. To check if you are the Input Source, use `IsInputSource`.

The server can also be the Input Source of objects, although it won’t do any CSP, since it needs not to, after all, it’s the server.

You can set the Input Source of an object when instantiating it:

```csharp
sandbox.NetworkInstantiate(PlayerPrefab, spawnPos, Quaternion.identity, client);
```

### Changing the Input Source

To set the Input Source of the object (must only be called on the server):

```csharp
Object.InputSource = newInputSource;
```

To remove the Input Source of the object (must only be called on the server):

```csharp
Object.InputSource = null;
```

## Callbacks

There are two methods you can override to run code when Input Source has changed or left the game:

1. `OnInputSourceChanged`: called on the Input Source and server when the Input Source changes.
2. `OnInputSourceLeft`: called on the owner (server) when the Input Source client has left the game.

## Framerate Lower than Tickrate

When the framerate is lower than the tickrate, there will be more ticks than frames. Therefore, two or more ticks need to use the same input of one frame. Not handling this can cause the player character to move slower during very low FPS. 

There are two ways to handle this issue:


### Enabling `Input Reuse At Low FPS` in Netick Settings

Enabling this option would automatically let Netick reuse/duplicate the same input of one frame to one or more ticks. You can also know if the input fetched is a duplicated input or not as follows:

```csharp
public override void NetworkFixedUpdate()
{
    if (FetchInput(out MyInput input, out bool isDuplicated))
    {
        if (!isDuplicated)
        {
            // do stuff when this input is not a duplicate.
        }
    }
}
```



### Networking the last fetched input

Or you can handle this yourself, by storing and networking the last fetched input:

```csharp

[Networked(relevancy: Relevancy.InputSource)]
private FirstPersonInput LastInput { get; set; }

public override void NetworkFixedUpdate()
{
    if (FetchInput(out MyInput input))
    {
        LastInput = input;
    }

    if (IsPredicted) // returns true on the client (if this object is predicted), and on the server.
    {
        // do movement stuff using LastInput.
    }
}
```
