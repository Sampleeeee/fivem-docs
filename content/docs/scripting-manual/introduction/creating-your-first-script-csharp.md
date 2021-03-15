---
title: Creating your first script in C#
weight: 412
---

Getting started with scripting for FiveM might be a tad overwhelming, given the wide range of possibilities and the sparsely spread documentation. In this quick and simple guide, we'll try to show you how to get started with a quick resource in C#. We will be implementing a car spawner through a command.

## Prerequisites
Before creating your first script with C#, there are a couple of things you will need to set up and understand.

* [Creating a C# project and setup your environment](/docs/scripting-manual/runtimes/csharp)
* [Understanding of resources and manifest files](/docs/scripting-reference/resource-manifest/resource-manifest)

### Writing code
Now that you have set up your C# project and environment, you will have two projects; `MyResourceNameClient` and `MyResourceNameServer`.

Any C# class that handles FiveM scripting-related events must inherit from the `BaseScript` class. Lets do this by going to `Class1.cs` in your client project. At the same time, we will also define a constructor, which we will use further on. Make sure you have a using directive to `CitizenFX.Core`.
```csharp
using CitizenFX.Core;

namespace MyResourceNameClient
{
    public class Class1 : BaseScript
    {
    
    }
}
```

Easy right? But what about adding functionality? We will start by adding a command using various FiveM scripting concepts.

```csharp
using System;
using System.Collections.Generic;
using CitizenFX.Core;
using static CitizenFX.Core.Native.API;

namespace MyResourceNameClient
{
    public class Class1 : BaseScript
    {
        [Command("car", Restricted = false)]
        private async Task SpawnCar(string[] args)
        {
            // TODO: Make a vehicle! FUN!
            TriggerEvent("chat:addMessage", new
            {
                color = new[] {255, 0, 0},
                args = new[] {"[CarSpawner]", $"I wish I could spawn this {(args.Count > 0 ? $"{args[0]} or" : "")} adder but my owner was too lazy. :("}
            });
        }
    }
}
```

You might be overwhelmed at this point, but don't worry. We will go through everything bit by bit.

```csharp
[Command("car", Restricted = false)]
```

We can place the `Command` above any method with one of the following types: `void` or `async Task`. Placing this above a method will automatically register it as a command. By setting the `Restricted` parameter to false, we allow all players to use this command. Setting it to `true` requires the player to have the `command.[command name]` ace permission. This native internally calls the {{% native_link "REGISTER_COMMAND" %}} native.

```csharp
private async Task SpawnCar(string[] args)
```

This is the method we will use to spawn the car. The beatiful `Command` attribute automatically pushes the `string[] args` parameter. The `args` are basically what you enter after the command like `/car zentorno` making `args` end up being `new string[1] { "zentorno" }` or `/car zentorno unused` being `new string[2]{ "zentorno", "unused" }`.


```csharp
TriggerEvent("chat:addMessage", new
{
    color = new[] {255, 0, 0},
    args = new[] {"[CarSpawner]", $"I wish I could spawn this {(args.Count > 0 ? $"{args[0]} or" : "")} adder but my owner was too lazy. :("}
});
```
But what about `TriggerEvent()`? It's used to call the event `chat:addMessage`, which is part of the [chat](/docs/resources/chat/events/chat-addMessage/) resource. In our written example, we send the author name `[CarSpawner]` in red and a message as arguments.

At this point, you can build your client project, add/move it to your resource and run it. When typing `/car` in the chat box, you will see our command returning the chat message we defined. ![screenshot-1](/csharp-tut-6.png)

Hey! It's complaining in the chat box that you were too lazy to implement this. We'll show them that you're _absolutely not lazy_, and actually implement this now.

### Implementing a car spawner
You may have followed the Lua tutorial on creating your first script and remember that there was a lot of boilerplate code that might looked overwhelming. Fear not, FiveM provides an easy to use C# wrapper that will allow us to reduce the code.

```csharp
[Command("car", Restricted = false)]
private async Task SpawnCar(string[] args)
{
    string model = "adder";
    if (args.Length > 0)
        model = args[0];

    uint hash = (uint) GetHashKey(model);
    if (!IsModelInCdimage(hash) || !IsModelAVehicle(hash))
    {
        TriggerEvent("chat:addMessage", new
        {
            color = new[] { 255, 0, 0 },
            args = new[] { "[CarSpawner]", $"It might have been a good thing that you tried to spawn a {model}. Who even wants their spawning to actually ^*succeed?" }
        });

        return;
    }

    Vehicle vehicle = await World.CreateVehicle(model, Game.PlayerPed.Position, Game.PlayerPed.Heading);
    Game.PlayerPed.SetIntoVehicle(vehicle, VehicleSeat.Driver);

    TriggerEvent("chat:addMessage", new {
        color = new[] { 255, 0, 0 },
        args = new[] { "[CarSpawner]", $"Woohoo! Enjoy your new ^*{model}!" }
    });
}
```

This uses some natives and C# wrapper methods. We'll link a few of them and explain the hard parts.

#### Step 1: Validation
We started with checking the model. We set it to `adder`. If there are any arguments, we set the model to the first argument and cast it to a string.

Then, we check if the vehicle is in the CD image using {{% native_link "IS_MODEL_IN_CDIMAGE" %}}. This basically means 'is this registered with the game'. We also check if it's a vehicle using {{% native_link "IS_MODEL_A_VEHICLE" %}}. If either check fails, we tell the player and return from the command.

There may be C# wrapper here, but it's important to reify the use of natives as you will use them a lot when scripting. Make sure you have the `using static CitizenFX.Core.Native.API;` directive in your class.

#### Step 2: Creating the vehicle
Using the client side C# wrapper class `World`, we call the `CreateVehicle` method which takes a model, `Vector3` position, and `float` heading as arguments. This is the great thing about C#. You have access to a method supplied by _us_ such that you don't have to request and load a model like you would in Lua. This method returns us a `Vehicle` object. If you have experience with *ScriptHookV.NET* you may recognize these classes. The C# wrapper of FiveM is very similar.

#### Step 3: Setting the player into the vehicle
Since we have our ped and a vehicle now, using the C# wrapper with the `Game.PlayerPed` object, we can set ourselves into the vehicle's driver seat.

### Running this
Build your project and make sure the latest `MyResourceNameClient.net.dll` is in the folder of your resource. In your server console, type `restart mymode` (or whatever you named your resource), and try `/car voltic2` in the game client (which should by now be really bored of respawning). You'll now have your very own Rocket Voltic!

## Server scripts
You'll probably also want to write scripts that interact with the server. This section is still to be written. :-(
