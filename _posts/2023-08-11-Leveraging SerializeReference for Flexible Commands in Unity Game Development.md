---
title: Leveraging SerializeReference for Flexible Commands in Unity Game Development
date: 2023-08-11
categories: [Unity]
tags: [Unity, UniRx, Intermediate, Advanced]
---

## Introduction

In the fast-paced world of game development, time is often a precious commodity. Finding efficient solutions that offer quick testing and flexible configuration becomes paramount. We, too, faced this challenge and discovered a potent approach that not only met these demands but also allowed us to customize our domain's behavior extensively.

Our secret weapon? The [SerializeReference](https://docs.unity3d.com/ScriptReference/SerializeReference.html) attribute coupled with a tailored Condition/Command pattern. While our exploration of this technique is still ongoing, the preliminary results are incredibly promising. We're excited to share our insights and invite you to join the conversation.

<img src="/assets/img/posts/commands/Unity%20SerializeReference%20Commands.jpg"/>

If you're curious about the base capabilities of SerializeReference, check out this basic yet powerful example in [Reddit thread](https://www.reddit.com/r/Unity3D/comments/14y0c1q/serializereference_is_very_powerfull_why_is_no/).

Let's dive into the tools that helped us craft this solution:

- **UniTask**: A free library offering efficient allocation-free async/await integration for Unity.
- **OdinInspector**: A library that facilitates instance creation from dropdowns using the **ISerializeReference** attribute (You can also create custom editors or explore alternatives like [Unity-SerializeReferenceExtensions](https://github.com/mackysoft/Unity-SerializeReferenceExtensions)). These tools emphasize the use of Commands through **SerializeReference**, with a special nod to the added brilliance when employing async/await principles.

### Unleashing Commands:
But what exactly are these "commands" we keep referring to? In essence, commands are serializable configurations of actions that can be executed within a given context. They're like the building blocks of dynamic behavior in your game.

```
public interface ICommand<TContext>
{
    UniTask ExecuteAsync(TContext context, CancellationToken cancellationToken);
}
```

Let's explore some examples:

#### Simple Contextual Commands for Character Behavior:

```
public class RestoreHPCharacterCommand : ICharacterCommand
{
    public UniTask Execute(Character context, CancellationToken cancellationToken)
    {
        context.RestoreHP();
        return UniTask.CompletedTask;
    }
}
```

```
public class ModifyStatisticCharacterCommand : ICharacterCommand
{
    public Id StatisticId;
    public float Value;

    public UniTask Execute(Character context, CancellationToken cancellationToken)
    {
        context.GetStatistic(StatisticId).ModifyBy(Value);
        return UniTask.CompletedTask;
    }
}
```

#### Versatile Commands for Any GameObject:

Behold an asynchronous command example:

```
public class MoveGameObjectCommand : IGameObjectCommand
{
    public Vector3 Direction;
    public float Duration;
    public float Distance;

    public async UniTask Execute(GameObject context, CancellationToken cancellationToken)
    {
        var startTime = Time.time;
        var initialPosition = context.transform.position;
        var targetPosition = initialPosition + Direction.normalized * Distance;

        while (Time.time - startTime < Duration)
        {
            float progress = (Time.time - startTime) / Duration;
            context.transform.position = Vector3.Lerp(initialPosition, targetPosition, progress);

            await UniTask.Yield(PlayerLoopTiming.Update, cancellationToken: cancellationToken);
        }

        context.transform.position = targetPosition;
    }
}
```

#### Building Blocks with Boilerplate Commands:

Meet the Command Group—a highly useful construct enabling the execution of multiple commands in parallel or sequentially:

```
public class CommandGroup<TCommand, TContext> : ICommand<TContext> where TCommand : ICommand<TContext>
{
    public List<CommandSetup> Commands = new();
    public bool ExecuteInParallel = true;

    public async UniTask ExecuteAsync(TContext context, CancellationToken cancellationToken)
    {
        if (ExecuteInParallel)
        {
            await UniTask.WhenAll(Commands.Select(item => item.ExecuteAsync(context, cancellationToken)));
            return;
        }

        foreach (var item in Commands)
        {
            await item.ExecuteAsync(context, cancellationToken);
        }

    }
}
```

Enter the Delay Command—perfect for orchestrating pauses between command executions:

```
public class DelayCommand<TContext> : ICommand<TContext>
{
    public float DelayMs;
    public UniTask ExecuteAsync(TContext context, CancellationToken cancellationToken)
    {
        return UniTask.Delay(TimeSpan.FromMilliseconds(DelayMs), cancellationToken: cancellationToken);
    }
}
```

Bridge the Unity world with the UnityEventCommand, facilitating actions like starting particle systems or toggling game objects:

```
public class UnityEventCommand<TCommand, TContext> : ICommand<TContext> where TCommand : ICommand<TContext>
{
    public UnityEvent Event;
    public UniTask ExecuteAsync(TContext context, CancellationToken cancellationToken)
    {
        Event?.Invoke();
        return UniTask.CompletedTask;
    }
}
```

## Conditions: Expanding Behavior Horizons

Conditions are an integral piece of the puzzle, perfectly complementing the concept of commands. With conditions, we can create complex behaviors by weaving together simple atomic actions. This enables us to craft dynamic and nuanced gameplay experiences, allowing our games to react intelligently to a variety of scenarios.

### What Are Conditions in the Context of Commands?

Conditions are logic checks that determine whether a certain command or set of commands should be executed based on the state of the game. This powerful addition allows us to create decision branches that respond dynamically to the gameplay context.

```
public interface ICondition<TContext>
{
    UniTask<bool> ExecuteAsync(TContext context, CancellationToken cancellationToken);
}
```

#### Examples of Condition-Driven Behavior:

- If a character's HP drops to 0, execute the PlayFeedbacksCommand.
- Depending on the damage severity from a DamageInfo context, trigger either a Critical Effect or a Default Effect.
- If the DamageType is Fire, apply fire-related effects.
- If the MagicShield's energy level is above 50%, enable the HighEnergy Effect and disable the LowEnergy Effect.

### Creating Condition Groups: All or Any

To harness the full potential of conditions, we can group them together to create more intricate decision-making structures. These groups allow us to determine whether all conditions must be met before executing a command or if just a single condition suffices.

```
public class ConditionGroup<TCondition, TContext> : ICondition<TContext> where TCondition : ICondition<TContext>
{
    public enum GroupType
    {
        All,
        Any
    }

    [EnumToggleButtons] public GroupType Type;
    [SerializeReference] public List<TCondition> Conditions = new();

    public async UniTask<bool> ExecuteAsync(TContext context, CancellationToken cancellationToken)
    {
        if (Type == GroupType.All)
        {
            foreach (var condition in Conditions)
            {
                if (!await condition.ExecuteAsync(context, cancellationToken))
                    return false;
            }
            return true;
        }
        else if (Type == GroupType.Any)
        {
            foreach (var condition in Conditions)
            {
                if (await condition.ExecuteAsync(context, cancellationToken))
                    return true;
            }
            return false;
        }

        return false;
    }
}
```

### Condition-Driven Command Execution

The synergy between conditions and commands allows us to craft more sophisticated behaviors. The `ConditionCommand` enables us to execute specific commands based on whether a condition is met or not. This adds a layer of intelligence to our gameplay, making it react dynamically to the context.

```
public class ConditionCommand<TCondition, TCommand, TContext> : ICommand<TContext>
        where TCommand : ICommand<TContext>
        where TCondition : ICondition<TContext>
{
    [SerializeReference] public TCondition Condition;
    [SerializeReference] public TCommand Command;
    [SerializeReference] public TCommand ElseCommand;

    public async UniTask ExecuteAsync(TContext context, CancellationToken cancellationToken)
    {
        var result = await Condition.ExecuteAsync(context, cancellationToken);

        if (result)
        {
            if (Command == null) return;
            await Command.ExecuteAsync(context, cancellationToken);
        }
        else
        {
            if (ElseCommand == null) return;
            await ElseCommand.ExecuteAsync(context, cancellationToken);
        }
    }
}
```

## Pros and Cons:

**Pros:**
- Modularity
- Reusability
- Easy configuration of new behaviors
- Exceptionally flexible solution accommodating various scenarios
- Leveraging presets for streamlined development

**Cons:**

- Potential complexity in inspector-side configuration
- Involvement of boilerplate code
- Dependency injection challenges

## Summary
In the dynamic realm of Unity game development, integrating **SerializeReference**, the Command pattern, and Conditions isn't just a fancy idea; it's a game-changer. This trifecta offers tangible benefits that can transform your game development process:

- **Flexibility Unleashed**: Embrace the power to mold behaviors on the fly. Whether it's character actions, environmental responses, or narrative twists, this approach adapts to your creative whims.
    
- **Efficiency Amplified**: Streamline your workflow with a unified toolkit. No need to reinvent the wheel for every system; these tools seamlessly integrate across components and ScriptableObjects.
    
- **Engagement Elevated**: Craft gameplay that reacts to players choices and actions. From immersive weather shifts to intricate dialogue trees, this approach enhances player immersion and investment.
    
- **Reusability Simplified**: Create components and behaviors that can be easily reused across your project. Save time and effort while maintaining consistent and polished gameplay elements.
    
- **Innovation Unlocked**: Experiment with combinations and possibilities that breathe life into your game. The synergy between these techniques sparks innovation and elevates your game above the ordinary.
    

So, as you embark on your next Unity adventure, remember that the fusion of **SerializeReference**, the Command pattern, and Conditions isn't just a theory—it's a practical gateway to a richer, more responsive, and thoroughly enjoyable game development journey. Start integrating these tools, watch your game thrive, and relish in the joy of creating something truly exceptional. Your game awaits its transformation!

## Resources

- [UniRx Library](https://github.com/neuecc/UniRx)
- [Odin Inspector](https://assetstore.unity.com/packages/tools/utilities/odin-inspector-and-serializer-89041)
