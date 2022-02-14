---
title: UniRx — PowerObservables
date: 2021-07-21
categories: [Unity]
tags: [Unity, UniRx, Intermediate]
---

<img src="/assets/img/posts/power-observables/01.webp"/>

## Prerequisites
- Familiar with Observer pattern
- Hands on any Rx library

## Introduction
PowerObservables are set of time-based operations with ability to pause and possibility to manage tick freqeuency

<img src="/assets/img/posts/power-observables/02.png"/>

## Why Power Observables
- Easy access to time-based operations
- Timers / Countdowns based on TimeSpans
- Ability to pause an observable using another observable (i.e. BoolReactiveProperty)

## Power Observables

### Countdown
Observable sequence that produces a differed time value for certain duration with ability to pause and resume. Completes when there is no time left.

It’s pretty usefull when it comes to
- Limit time in Arcade Games
- Countdown before race start when you want to prompt “3, 2, 1, GO”
- Damage overtime — every tick deal 10 damage to player, on complete remove debuff

{% highlight csharp %}
PowerObservable
	.Countdown(Time)
	.Subscribe(OnTick, OnComplete)
	.AddTo(this);
{% endhighlight %}

<center>

Sequence: Countdown(3) → 2 → 1 → done with 3rd tick

</center>

<img src="/assets/img/posts/power-observables/03.gif"/>

### TimerInterval 
Observable sequence that produces a value after each period for certain duration with ability to pause.

{% highlight csharp %}
PowerObservable
	.TimerInterval(Time)
	.Subscribe(OnTick, OnComplete)
	.AddTo(this);
{% endhighlight %}

<center>

Sequence: TimerInterval(3) → 1 → 1 → done with 3rd tick

</center>

### CountedInterval  
Returns an observable sequence with aggregated time since subscribe with pause option

{% highlight csharp %}
PowerObservable
	.TimerInterval(Time)
	.Subscribe(OnTick, OnComplete)
	.AddTo(this);
{% endhighlight %}

<center>

Sequence: CountedInterval() → 1 → 2 → 3 → 4 → 5…

</center>

<img src="/assets/img/posts/power-observables/04.gif"/>

### TimerCountedInterval   
Returns an observable sequence with aggregated time since subscribe. Lasts for certain duration. It’s the same concept as Countdown but reversed. Instead times go up from zero. It completes after certain duration.

{% highlight csharp %}
PowerObservable
	.TimerInterval(Time)
	.Subscribe(OnTick, OnComplete)
	.AddTo(this);	   
{% endhighlight %}

<center>

Sequence: TimerCountedInterval(3) → 1 → 2 → done with 3rd tick

</center>

## How To Use
Simple use case scenario below:

{% highlight csharp %}
var pause = new BoolReactiveProperty(); // create UniRx ReactiveProperty to manage pause

PowerObservables
	.CountedInterval(pause, tick: 1f) // you can provide the freqeuency of ticks
	.Subscribe(time => 	
	{ 
		// every tick 
		// 00:00 ...
		// 00:01 ...
		// 00:02
	})
	.AddTo(this); 

pause.Value = true; // interval will stops
pause.Value = false; // interval will resume from previous point
{% endhighlight %}

## Unity Installation
Since unity doesn’t support git dependencies in package.json you have to install [UniRx](https://github.com/neuecc/UniRx#upm-package) manually.
Edit manifest.json file in your Unity Packages directory

{% highlight javascript %}
{
	"dependencies": {
		"com.gbros.tools.unirx.powerobservables": "[<https://github.com/GbrosGames/Tools.git?path=Assets/UniRx/PowerObservables>](<https://github.com/GbrosGames/Tools.git?path=Assets/UniRx/PowerObservables>)",
		"com.neuecc.unirx": "[<https://github.com/neuecc/UniRx.git?path=Assets/Plugins/UniRx/Scripts>](<https://github.com/neuecc/UniRx.git?path=Assets/Plugins/UniRx/Scripts>)"
	}
}
{% endhighlight %}

## Unity Examples
All code samples can be found on our GitHub [page](https://github.com/GbrosGames/Tools/tree/main/Assets/UniRx/PowerObservables)

### [Example 1](https://github.com/GbrosGames/Tools/blob/main/Assets/UniRx/PowerObservables/Samples~/Example%201/CountedInterval.cs) - Counted Interval
This example shows how to create Interval observable that will return TimeSpan in the cumulative way. It shows the way of refreshing the UI. It also shows use case for BoolReactiveProperty from UniRx library to manage pause/resume functionality.

### [Example 2](https://github.com/GbrosGames/Tools/blob/main/Assets/UniRx/PowerObservables/Samples~/Example%202/Countdown.cs) — Countdown
This example shows how to create, restart and refresh Countdown and how to manage subscription and completion methods in observable.

## Where To Go From Here
[MessageBroker](https://www.gbrosgames.com/post/unirx-series-part-1-messagebroker)

## Resources
[UniRx Library](https://github.com/neuecc/UniRx)