# Introduction

The New Relic Video Agent is a set of libraries for **video telemetry**. It works on top of the New Relic Mobile Agent, observing events generated in the video player and sending custom events using `recordCustomEvent`, that are registered under the event type `MobileVideo`.

It is a modular toolkit. The base classes and common behaviour is located inside the **NewRelicVideoCore** library. And there are other libraries for player specific support (trackers).

# Events

The data model revolves around the concept of **Event**. An event is composed by an **Action** and a set of **Attributes**. The action tells what is the event about, and the attributes gives context.

The most common actions currently supported are:

| Action | Sent when... |
| ------ | ------------ |
| `TRACKER_READY` | The tracker is started. |
| `PLAYER_READY` | The tracker got a valid player instance. |
| `CONTENT_REQUEST` | A video stream is requested. |
| `CONTENT_START` | Video started, first frame shown. |
| `CONTENT_BUFFER_START` | Video started buffering. |
| `CONTENT_BUFFER_END` | Video ended buffering. |
| `CONTENT_PAUSE` | Video paused. |
| `CONTENT_RESUME` | Video resumed after a pause. |
| `CONTENT_END` | Video ended. |
| `CONTENT_ERROR` | An error happened. |
| `CONTENT_HEARTBEAT` | Every 30 seconds between `CONTENT_START` and `CONTENT_END`.  |
| `CONTENT_RENDITION_CHANGE` | Stream quality changed. |

All the `CONTENT_*` actions come in `AD_` flavor for Ads trackers.

And some of the most prominent attributes are:

| Attribute | Description |
| ------ | ----------- |
| `contentTitle` | Title of the video. |
| `contentDuration` | Video duration. |
| `contentPlayhead` | Current playback position. |
| `contentSrc` | Stream source (URL). |
| `contentBitrate` | Video bitrate. |
| `contentRenditionWidth` | Video width. |
| `contentRenditionHeight` | Video height. |
| `contentFps` | Video frames per second. |
| `contentLanguage` | Video language. |
| `contentIsMuted` | Video is muted. |
| `contentIsLive` | Video is a live stream. |
| `playerName` | Name of the video player. |
| `playerVersion` | Version of the video player. |
| `viewId` | ID of current playback. |
| `totalPlaytime` | Total time played. |
| `playtimeSinceLastEvent` | Time played since last event sent. |

Again, the `content*` attributes come in `ad` flavor for Ads trackers.

There are also action specific attributes. These are attributes that are only included in events with a certain action. Of those, the most important ones are the **TimeSince Attributes**.

The TimeSince attributes are timers, they mark the time elapsed since a certain event happened. The most common ones are:

| Attribute | Description | Included in |
| ------ | ----------- | ----------- |
| `timeSinceTrackerReady` | Time since `TRACKER_READY` was sent. | All `CONTENT_` events. |
| `timeSinceRequested` | Time since `CONTENT_REQUEST` was sent. | All `CONTENT_` events. |
| `timeSinceStarted` | Time since `CONTENT_START` was sent. | All `CONTENT_` events. |
| `timeSincePaused` | Time since `CONTENT_PAUSE` was sent. | `CONTENT_RESUME` |
| `timeSinceSeekBegin ` | Time since `CONTENT_SEEK_START` was sent. | `CONTENT_SEEK_END` |
| `timeSinceBufferBegin ` | Time since `CONTENT_BUFFER_START` was sent. | `CONTENT_BUFFER_END` |
| `timeSinceLastRenditionChange` | Time since `CONTENT_RENDITION_CHANGE` was sent. | `CONTENT_RENDITION_CHANGE` |
| `timeSinceLastHeartbeat` | Time since `CONTENT_HEARTBEAT` was sent. | All `CONTENT_` events. |
| `timeSinceLastAd` | Time since last Ad was played. | All `CONTENT_` events. |

Once again, we have the Ad versions, like `timeSinceLastAdHeartbeat`, `timeSinceAdStarted`, `timeSinceAdRequested`, etc.

### Ad specific events

There are some actions that are specific of ads. These are:

| Action | Sent when... |
| ------ | ------------ |
| `AD_BREAK_START` | An ad break starts. |
| `AD_BREAK_END` | And ad break ends. |
| `AD_QUARTILE` | Every quarter of the ad viewed. |
| `AD_CLICK` | User clicked on the ad. |

The `AD_BREAK_` block can contain multiple consecutive ads (signaled with `AD_START` and `AD_END`).

The `AD_QUARTILE` is sent 3 times within an ad, one when the first quarter of the ad is viewed, then when the second quarter (the half), and finally when the third quarter. The fourth quarter is the `AD_END`, so no quartile event is sent.

# Trackers

Trackers are the classes used to capture data from a player and generate the events. A tracker for a video player extends the class `NRVideoTracker`. This class extends `NRTracker`, that only contains the most essential functionalities, like event and attribute generation.

### Start the New Relic Video Agent

Starting the New Relic Video Agent implies creating a tracker. For iOS the default video player is AVPlayer, and Android is ExoPlayer. The New Relic Video Agent provides modules for with trackers for both players, the `NRAVPlayerTracker` library for iOS and the `NRExoPlayerTracker` library for Android.

To start it, we will call the start method passing a tracker instance to it. And the tracker instance is (usually) created by passing the player instance:

<details>
<summary>iOS</summary>
<p>

```Objective-C
// Start the New Relic Video Agent with the tracker
NSNumber *trackerId = [[NewRelicVideoAgent sharedInstance] startWithContentTracker:[[NRTrackerAVPlayer alloc] initWithAVPlayer:player]];
```

</p>
</details>
<details>
<summary>Android</summary>
<p>

```Java
// Start the New Relic Video Agent with the tracker
Integer trackerId = NewRelicVideoAgent.getInstance().start(new NRTrackerExoPlayer(player));
```

</p>
</details>

### Using a tracker

When started, the New Relic Video Agent will return a tracker ID, that can be used later to obtain the tracker instance:

<details>
<summary>iOS</summary>
<p>

```Objective-C
NRTrackerAVPlayer *tracker = (NRTrackerAVPlayer *)[[NewRelicVideoAgent sharedInstance] contentTracker:trackerId];
```

</p>
</details>
<details>
<summary>Android</summary>
<p>

```Java
NRTrackerExoPlayer tracker = (NRTrackerExoPlayer)NewRelicVideoAgent.getInstance().getContentTracker(trackerId);
```

</p>
</details>

We can use this instance to manually send events, or in general call the methods we desire:

<details>
<summary>iOS</summary>
<p>

```Objective-C
// Get duration of current video
NSNumber *duration = [tracker getDuration];
// Send a custom event
[tracker sendEvent:@"MY_TEST_ACTION"];
// Send a custom event with custom attributes
[tracker sendEvent:@"MY_TEST_ACTION" attributes:@{@"myAttr": @"myVal"}];
```

</p>
</details>
<details>
<summary>Android</summary>
<p>

```Java
// Get duration of current video
Long duration = tracker.getDuration();
// Send a custom event
tracker.sendEvent("MY_TEST_ACTION");
// Send a custom event with custom attributes
Map att = new HashMap();
att.put("myAttr", "myVal");
tracker.sendEvent("MY_TEST_ACTION", att);
```

</p>
</details>

And once the tracker is no longer neede, we can release it:

<details>
<summary>iOS</summary>
<p>

```Objective-C
[[NewRelicVideoAgent sharedInstance] releaseTracker:trackerId];
```

</p>
</details>
<details>
<summary>Android</summary>
<p>

```Java
NewRelicVideoAgent.getInstance().releaseTracker(trackerId);
```

</p>
</details>

### Using Ads trackers

An ads tracker is just a normal tracker. What makes it different is how it is initialized. Any `NRVideoTracker` can be an ads tracker, just by passing it in the correct argument to the New Relic Video Agent start method:

<details>
<summary>iOS</summary>
<p>

```Objective-C
trackerId = [[NewRelicVideoAgent sharedInstance]
				startWithContentTracker:myContentTracker
				adTracker:myAdTracker];
```

</p>
</details>
<details>
<summary>Android</summary>
<p>

```Java
trackerId = NewRelicVideoAgent.getInstance().start(myContentTracker, myAdTracker);
```

</p>
</details>

In the example above we are passing a content tracker and an ad tracker to start, but we could only pass an ad tracker, just by setting the content tracker null/nil.

Once started, the New Relic Video Agent sets an internal flag in the tracker state, that is `isAd`. It is set to `true` for the ads tracker, and `false` for the content tracker. This flag causes the tracker to automatically generate the `CONTENT_` actions or the `AD_` actions. The video agent also sets a property in the tracker, named `linkedTracker`. This property holds a reference to the partner, the content tracker will have a reference to the ads tracker, and vice versa. If there is no partner this property is null/nil.

Both, the ads and the contents tracker are tied to the same tracker ID. We already saw how to get the contents tracker instance using the tracker ID, we can do the same for the ads tracker:

<details>
<summary>iOS</summary>
<p>

```Objective-C
// Set the appropiate var type and cast for the ads tracker class we are using
NRTracker *adTracker = [[NewRelicVideoAgent sharedInstance] adTracker:trackerId];
```

</p>
</details>
<details>
<summary>Android</summary>
<p>

```Java
// Set the appropiate var type and cast for the ads tracker class we are using
NRTracker adTracker = NewRelicVideoAgent.getInstance().getAdTracker(trackerId);
```

</p>
</details>

### Creating a custom tracker

Let's guess we want to support a new player, or we just want to sense a default player in a different way. In this case we need a custom tracker that understands about the player, knows how to register listeners and read properties to generate the appropiate events. To create a custom tracker we usually estend the `NRVideoTracker` class.

The anatomy of a video tracker is as follows:

Every tracker should provide a constructor to pass the player instance. This constructor should call `setPlayer`. A tracker also should override the `setPlayer` to do whatever is necessary, like calling `registerListeners` method.

Every tracker should override `registerListeners` and `unregisterListeners`. These methods do what the name suggests: register player event observers and unregister them. The `registerListeners`, as already said, should be called after the player is set, and the `unregisterListeners` is automatically called when the tracker is released.

A tracker contains a sender method to generate each one of the events described in the chapter *Events*. For example, to generate a `CONTENT_REQUEST` / `AD_REQUEST`, we have the method `sendRequest`. These methods are inherited from `NRVideoTracker` class, and are used in the tracker to generate the events. The main job of a tracker is to handle the player event observers, understand what each observer means and call the appropiate senders to generate the events.

Every video tracker has a `state` property, inherited from `NRVideoTracker`. This property is an instance of `NRTrackerState` and holds the state of the tracker/player, and has flags to signal if the player is seeking, or paused, or buffering, etc. This state is updated with the sender methods.

A tracker also contains a getter method for each one of the attributes. For example to generate the `contentDuration` / `adDuration` we have the method `getDuration`. A tracker can override the getters for the standard attributes. The tracker must know how to obtain the requested information from the player.

Besides the standard attributes that are generated by overriding the getters, a tracker can generate specific attributes. To do so, it can override the `getAttributes` method. This method returns a dictionary/hashmap with all the attributes that will be included in a particular event.

Finally we have the TimeSince attributes. A tracker can register custom TimeSince attributes by overriding the `generateTimeSinceTable` method, and from it, call `addTimeSinceEntry` for each one of the attributes it wants to generate. This method takes 3 arguments. The first is the action that will trigger the timer (the reference to start counting the time). The second is the name of the TimeSince attribute. And the third is a regexp filter, to match the actions that will include the attribute. In the following example we see how one of the standard TimeSince attributes is generated, the `timeSinceLastHeartbeat`. The fist argument is `CONTENT_HEARTBEAT`, the action that will (re)start the timer. Next comes the attribute name. And finally in which events this attribute will be included, in this case all `CONTENT_` actions.

<details>
<summary>iOS</summary>
<p>

```Objective-C
[self addTimeSinceEntryWithAction:CONTENT_HEARTBEAT attribute:@"timeSinceLastHeartbeat" applyTo:@"^CONTENT_[A-Z_]+$"];
```

</p>
</details>
<details>
<summary>Android</summary>
<p>

```Java
addTimeSinceEntry(CONTENT_HEARTBEAT, "timeSinceLastHeartbeat", "^CONTENT_[A-Z_]+$");
```

</p>
</details>

# Data lifecycle

An event travels thought multiple steps from the moment it is generated till the moment it is sent to New Relic using the mobile agent `recordCustomEvent` method.

Everything starts with a call to a sender, for example `sendRequest`. A particular sender must check the tracker state to make sure we can send the event. For example, if a `CONTENT_REQUEST` was already sent, we don't send it again. Or, if we call `sendBufferEnd` but we are not buffering (we did't call `sendBufferStart`), it must be ignored. This is done by using the `go` methods inside the `state` instance, like `goRequest` or `goBufferEnd`. These methods returns `true` if the event can be sent, and also update the state.

Once the sender knows the event can be sent, it calls `sendEvent`, providing the action name, and optinally a list of attributes. This method generates the event, by calling `getAttributes`, merging the attributes into one dictionary/hashmap, and finally giving it the necessary format to call `recordCustomEvent`.

But before calling `recordCustomEvent`, it calls the method `preSend`. This method returns `true` if the event must be sent, or `false` if it must be discarded. The `NRTracker` class provides a default implementation of this method that always returns `true`. This is a mechanism for custom trackers that want to ignore certain events in certain moments, or they want to send the event in a different way (not using the default `recordCustomEvent`), or they want to tranform the events somehow.

If `preSend` returns `true`, the event is sent to `recordCustomEvent` and the journey ends here.