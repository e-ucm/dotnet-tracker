# DotNET tracker

![logo_rage_top-of-the-web](https://cloud.githubusercontent.com/assets/5657407/16461800/9ae669fc-3e2e-11e6-97f4-4e93f2c96dea.jpg)

This code belongs to [RAGE project](http://rageproject.eu/) and sends analytics information to a server; or, if the server is currently unavailable, stores them locally until it becomes available again.

After a game is developed, a common need is to know how the players play, what interactions they follow within the game and how much time they spend in a game session; collectively, these are known as game analytics. Analytics are used to locate gameplay bottlenecks and assess  game effectiveness and learning outcomes, among other tasks.

## Installation
1. Clone the repository with `git clone --recursive -j8 git://github.com/e-ucm/dotnet-tracker.git`
1. Include projects into your solution or open the existing solution found in the root of the repository:
  1. If you already have the AssetManager project, ignore it, if not, include it into your solution.
  1. Include TrackerAsset and make sure you reference AssetManager from TrackerAsset
  1. Reference AssetManager and TrackerAsset into your main project.
1. Create a new TrackerAsset instance and start configuring it by creating a new instance of `TrackerAssetSettings()`:
  1. **Host**: If storage type is set to `TrackerAsset.StorageTypes.net`, this should have the host for the analytics server 
  1. **Secure**: If the server uses a secure connection set it to true.
  1. **Port**: This should have the port of the analytics server. By default, port 80 is used.
  1. **BasePath**: Lets you set the route for the api path of the analytics server. Usually set it to "/api".
  1. **StorageType**: can be `TrackerAsset.StorageTypes.net`, to send traces to a server, o `TrackerAsset.StorageTypes.local`, to store them locally.
  1. **TraceFormat**: the format of the traces. Can be `TrackerAsset.TraceFormats.xapi` (Recommended), `TrackerAsset.TraceFormats.csv` or `TrackerAsset.TraceFormats.json` (Deprecated).
  1. **TrackingCode**: If storage type is set to `TrackerAsset.StorageTypes.net`, this is the [tracking code identifying the game](https://github.com/e-ucm/rage-analytics/wiki/Tracking-code)
  1. **BackupStorage**: If true, creates a traces backup file into the filesystem, formated in CSV
1. Set up a bridge for creating the connections with the server.
1. Send traces

<b>Note</b>: The traces file are saved in the path specified by the Bridge. 

## Usage Example

Using the tracker is a really simple proccess. For expanding the Installation guide here you can found some examples of how to setup and use the tracker.

### Tracker Instatiation

First, you need to create and manage an instance.

```c#

TrackerAssetSettings tracker_settings = new TrackerAssetSettings()
{
  Host = "https://rage.e-ucm.es/",
  TrackingCode = "OBTAINED_FROM_SERVER",
  BasePath = "/api",
  Port = 334,
  Secure = domain.Split('/')[0] == "https:",
  StorageType = TrackerAsset.StorageTypes.net,
  TraceFormat = TrackerAsset.TraceFormats.xapi,
  BackupStorage = true
};
```

## Detailed Feature List
1. Configurable flush intervals (via `T().SetFlushInterval()`; use `-1` to entirely avoid auto-flush). If flushing fails, for example due to transient network problems, the tracker will periodically attempt to re-send the data. 
1. Different storage types: 
	1. `net`: sends data to a trace-server, such as the [rage-analytics Backend](https://github.com/e-ucm/rage-analytics-backend). If set, a hostname should be specified via the `host` property.
	2. `local`, to store them locally for later retrieval. Un-sent traces are always persisted locally before being sent through the net, to support intermittent internet access.
1. Different trace formats:
	2. `csv`: allow processing in MS Excel or other spreadsheets. Also supported by many analytics environments.
	3. `json`: especially intended for programmatic analysis, for instance using python or java/javascript or
	4. `xapi`: an upcoming standard for student activity. Note that, if the tracker's storage type is `net` it is required to use the `xapi` trace format since the [rage-analytics Backend](https://github.com/e-ucm/rage-analytics-backend) expects xAPI Statements. The [xAPI tracking model] (https://github.com/e-ucm/xapi-seriousgames) that the backend expects is composed of [Completables](https://github.com/e-ucm/xapi-seriousgames/blob/master/README.md#1341-completable), [Reachables](https://github.com/e-ucm/xapi-seriousgames/blob/master/README.md#1341-reachable), [Variables](https://github.com/e-ucm/xapi-seriousgames/blob/master/README.md#1342-variables) and [Alternatives](https://github.com/e-ucm/xapi-seriousgames/blob/master/README.md#1343-alternatives). 
1. Tracker messages can be displayed in the Unity console by setting the `Debug` property
1. Uses Unity's in-built facilities to handle connections, files and timing.
 
### User Guide

The tracker requires (if `net` mode is on) the [RAGE Analytics](https://github.com/e-ucm/rage-analytics) infrastructure up and running. Check out the [Quickstart guide](https://github.com/e-ucm/rage-analytics/wiki/Quickstart) and follow the `developer` and `teacher` steps in order to create a game and [setup a class](https://github.com/e-ucm/rage-analytics/wiki/Set-up-a-class). It also requires a:

* **Host**: where the server is at. This value usually looks like `<rage_server_hostmane>/api/proxy/gleaner/collector/`. The [collector](https://github.com/e-ucm/rage-analytics/wiki/Back-end-collector) is an endpoint designed to retrieve traces and send them to the analysis pipeline.
* **Tracking code**: an unique tracking code identifying the game. [This code is created in the frontend](https://github.com/e-ucm/rage-analytics/wiki/Tracking-code), when creating a new game.


The tracker exposes an API designed to collect, analyze and visualize the data. The  API consists on defining a set of **game objects**. A game object represents an element of the game on which players can perform one or several types of interactions. Some examples of player's interactions are:

* start or complete (interaction) a level (game object)
* increase or decrease (interaction) the number of coins (game object)
* select or unlock (interaction) a power-up (game object)

A **gameplay** is the flow of interactions that a player performs over these game objects in a sequential order.

The main typed of game objects supported are:

* [Completable](https://github.com/e-ucm/unity-tracker/blob/master/Tracker/Format/CompletableTracker.cs) - for Game, Session, Level, Quest, Stage, Combat, StoryNode, Race or any other generic Completable. Methods: `Initialized`, `Progressed` and `Completed`.
* [Accessible](https://github.com/e-ucm/unity-tracker/blob/master/Tracker/Format/AccessibleTracker.cs) - for Screen, Area, Zone, Cutscene or any other generic Accessible. Methods: `Accessed` and `Skipped`.
* [Alternative](https://github.com/e-ucm/unity-tracker/blob/master/Tracker/Format/AlternativeTracker.cs) - for Question, Menu, Dialog, Path, Arena or any other generic Alternative. Methods: `Selected` and `Unlocked`.
* [TrackedGameObject](https://github.com/e-ucm/unity-tracker/blob/master/Tracker/Format/GameObjectTracker.cs) for Enemy, Npc, Item or any other generic GameObject. Methods: `Interacted` and `Used`.

##### Completable

Usage example for the tracking of an in-game quest. We decided to use a [Completable](https://github.com/e-ucm/unity-tracker/blob/master/Assets/Format/CompletableTracker.cs) game object for this use-case as the most suitable option:

```c#

	// Completable
	// Initialized
	Tracker.T.completable.Initialized("MyGameQuestId", Completable.Quest);
	
	//...
	
	// Progressed
	Tracker.T.completable.Progressed("MyGameQuestId", Completable.Quest, 0.8);
	
	//...
	
	// Progressed
	bool success = true;
	Tracker.T.completable.Completed("MyGameQuestId", Completed, success);

```

##### Accessible

Usage example for the tracking the player's movement through some in-game screens and skipping the `Intro` cutscene:

```c#
	
	// Accessible
	// The player accessed the 'MainMenu' screen
	Tracker.T.accessible.Accessed("MainMenu", Accessible.Screen);
	
	//...
	
	// The player skipped a cutscene
	Tracker.T.accessible.Skipped("Intro", Accessible.Cutscene);

```

##### Alternative

Usage example for the tracking the player's choices during a conversation:

```c#
	
	// Alternative
	// The player selected the 'Dan' answer for the question 'What's his name?'
	Tracker.T.alternative.Selected("What's his name?", "Dan", Alternative.Question);
	
	//...
	
	// The player selected 'OK' for the question 'Do you want it?'
	Tracker.T.alternative.Selected("Do you want to start right now?", "OK", Alternative.Question);

	//...
	
	// The player unlocked 'Combat Mode' for the menu 'Menues/Start'
	Tracker.T.alternative.Unlocked("Menues/Start", "Combat Mode", Alternative.Menu);
	
```

##### Tracked Game Object

Usage example for the tracking the player's with a NPC villager and using a health potion (item):

```c#
	
	// Tracked Game Object
	// The player interacted with a Non Playable Character
	Tracker.T.trackedGameObject.Interacted("NPC/Villager", TrackedGameObject.Npc);
	
	//...
	
	// The player used a health potion
	Tracker.T.trackedGameObject.Used("Item/HealthPotion", TrackedGameObject.Item);
	
```

Note that in order to track other type of user interactions it is required to perform a previous analysis to identify the most suitable game objects ([Completable](https://github.com/e-ucm/unity-tracker/blob/master/Assets/Format/CompletableTracker.cs), [Accessible](https://github.com/e-ucm/unity-tracker/blob/master/Assets/Format/AccessibleTracker.cs), [Alternative](https://github.com/e-ucm/unity-tracker/blob/master/Assets/Format/AlternativeTracker.cs), [TrackedGameObject](https://github.com/e-ucm/unity-tracker/blob/master/Assets/Format/GameObjectTracker.cs)) for the given case. For instance, in order to track conversations [Alternative](https://github.com/e-ucm/unity-tracker/blob/master/Assets/Format/AlternativeTracker.cs) is the best choice

### Tracker and Collector Flow
If the storage type is `net`, the tracker will try to connect to a `Collector` [endpoint](https://github.com/e-ucm/rage-analytics/wiki/Back-end-collector), exposed by the [rage-analytics Backend](https://github.com/e-ucm/rage-analytics-backend). 

More information about the tracker can be found in the [official documentation of rage-analytics] (https://github.com/e-ucm/rage-analytics/wiki/Tracker).

