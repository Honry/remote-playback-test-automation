# WebDriver Extension API for Remote Playback

The [Remote Playback API] poses a challenge to test authors, as fully exercising those interfaces requires remote playback devices that respond in predictable ways. To address this challenge this document defines a number of extension commands to the [WebDriver] specification for controlling fake remote playback devices on the host that the user agent is running on. With these extension commands, devices with particular properties can be created and their responses to requests are well defined.

## Fake remote Playback device

A `fake remote playback device` simulates the behavior of a `remote playback device` in controlled ways. Each `fake remote playback device` has a unique identifier name.

**Note**: A `fake remote playback device` is initially available, no `fake remote playback device` is added means the `list of available remote playback devices` is empty.

A `fake remote playback device` have associated `fake remote playback state`, which should be in sync with the media element state when `fake remote playback device` is connected.

**Note**: The `fake remote playback device` defined in this specification is not intended be used by non-testing-related web content. The user agent MAY choose to expose fake remote playback device interface only when a runtime or compile-time flag has been set.

When a user agent chooses to expose `fake remote playback device` interface, it MUST reconfigure its internal implementation of the RemotePlayback object in the current global object so that it MUST detect availability of, connect to and control playback on `fake remote playback devices` in following ways:
- Once a `fake remote playback device` is added, it SHOULD be added to the `list of available remote playback devices`.
- When invoke `watchAvailability()` method, the UA MUST monitor the `list of available remote playback devices`.
- When invoke `prompt()` method, no UI will be prompted, the UA MUST connect to selected fake remote playback.
- If a `fake remote playback device` is added with `selected` set to `true`, that it will become the selected `fake remote playback device` for the next request to `establish a connection with the remote playback device`.
- If multiple `fake remote playback devices` are added with `selected` set to `true`, then the implementation acts as if no `fake remote playback device` has been selected (i.e., the `prompt()` Promise will remain unresolved).
- If no `fake remote playback device` has `selected` set to `true`, the UA will behave as if the user has denied permission to use any `fake remote playback device`.
- When the `state` of a `RemotePlayback` object is `connected`, the UA SHOULD satisfy conditions relate the `local playback state`, the `media element state`, and the `fake remote playback state` described in [`Media commands and media playback state`](https://w3c.github.io/remote-playback/#media-commands-and-media-playback-state).

### FakeRemotePlayback dictionary
The FakeRemotePlayback dictionary provides information about a `fake remote playback device`.

```
dictionary FakeRemotePlayback {
  DOMString name;
  boolean selected;
  FakeRemotePlaybackState state;
}
```

### FakeRemotePlaybackInit dictionary
The FakeRemotePlaybackInit dictionary is used to initialize a `fake remote playback device`.
```
dictionary FakeRemotePlaybackInit {
  required DOMString name;
  boolean selected = true;
  FakeRemotePlaybackState state;
}
```

### FakeRemotePlaybackState enumeration
A FakeRemotePlaybackState enumeration represents the state of connection to some `fake remote playback device`.
```
enum FakeRemotePlaybackState {
  "playing"
  "paused"
  "end"?
  …
}
```

## Extension Commands
### Add fake remote playback device
|HTTP Method|URI Template|
|----|----|
|POST|/session/{session id}/fakeremoteplayback/|

The `add fake remote playback device` `extension command` adds a new `fake remote playback device`.

TODO: Define the remote end steps.

### Get fake remote playback device
|HTTP Method|URI Template|
|----|----|
|GET|/session/{session id}/fakeremoteplayback/{name}|

The `get fake remote playback device` `extension command` retrieves information about a given name of `fake remote playback device`.

TODO: Define the remote end steps.

### Update fake remote playback device
//e.g. To change the selected attribute to test disconnection, change fake remote playback state to test reflection on local playback state.
|HTTP Method|URI Template|
|----|----|
|POST|/session/{session id}/fakeremoteplayback/{name}|

The `update fake remote playback device` `extension command` updates properties about a given name of `fake remote playback device`.

TODO: Define the remote end steps.

### Remove fake remote playback device
|HTTP Method|URI Template|
|----|----|
|DELETE|/session/{session id}/fakeremoteplayback/{name}|

The `remove fake remote playback device` `extension command` removes a given name of `fake remote playback device`.

TODO: Define the remote end steps.

### Remove all fake remote playback devices
|HTTP Method|URI Template|
|----|----|
|DELETE|/session/{session id}/fakeremoteplayback/|

The `remove all fake remote playback devices` `extension command` removes all existing `fake remote playback devices`.

TODO: Define the remote end steps.

## Handling WebDriver Errors
This section extends the `Handling Errors` and defines extended `WebDriver error codes` specific for `fake remote playback device` in following table.

|Error Code|HTTP Status|JSON Error Code|Description|
|----------|-----------|---------------|-----------|
|***no such fake remote playback device***|404|`no such fake remote playback device`|no fake remote playback device matching the given name was found.|
|***fake remote playback already created***|500|`fake remote playback device already created`|A command to create a fake remote playback device could not be satisfied because the given name of fake remote playback device is already existed.|


## Open:
1. Media element has a couple of [playback state](https://html.spec.whatwg.org/multipage/media.html#htmlmediaelement), which should be tested in remote playback?
```
   // playback state 
	attribute double currentTime;
	readonly attribute unrestricted double duration;
	readonly attribute boolean paused; 
	attribute double defaultPlaybackRate;
	attribute double playbackRate;
	readonly attribute TimeRanges played;
	readonly attribute TimeRanges seekable;
	readonly attribute boolean ended;
	[CEReactions] attribute boolean autoplay;
	[CEReactions] attribute boolean loop;
```