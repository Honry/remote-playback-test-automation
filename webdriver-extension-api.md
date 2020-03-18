# WebDriver Extension API for Remote Playback

The [Remote Playback API] poses a challenge to test authors, as fully exercising those interfaces requires `remote playback devices` that respond in predictable ways. To address this challenge this document defines a number of `extension commands` to the [WebDriver] specification for controlling `fake remote playback devices` on the host that the user agent is running on. With these `extension commands`, devices with particular properties can be created and their responses to requests are well defined.

## Fake remote Playback device

A `fake remote playback device` simulates the behavior of a `remote playback device` in controlled ways. Each `fake remote playback device` has a unique identifier name.

**Note**: A `fake remote playback device` is initially available, no `fake remote playback device` is added means the `list of available remote playback devices` is empty.

A `fake remote playback device` have associated `fake remote playback state`, which should be in sync with the media element state when `fake remote playback device` is connected.

**Note**: The `fake remote playback device` defined in this specification is not intended be used by non-testing-related web content. The user agent MAY choose to expose `fake remote playback device` interface only when a runtime or compile-time flag has been set.

When a user agent chooses to expose `fake remote playback device` interface, it MUST reconfigure its internal implementation of the `RemotePlayback` object in the current global object so that it MUST detect availability of, connect to and control playback on `fake remote playback devices` in following ways:
- Once a `fake remote playback device` is added, it SHOULD be added to the `list of available remote playback devices`.
- The `list of available remote playback devices` SHOULD only contains `fake remote playback devices`.
- When invoke `watchAvailability()` method, the UA MUST monitor the `list of available remote playback devices`.
- When invoke `prompt()` method, no UI will be prompted, the UA MUST connect to the selected `fake remote playback device`.
- If a `fake remote playback device` is added with `selected state` set to `true`, that it will become the selected `fake remote playback device` for the next request to `establish a connection with the remote playback device`.
- If multiple `fake remote playback devices` are added with `selected state` set to `true`, then the implementation acts as if no `fake remote playback device` has been selected (i.e., the `prompt()` Promise will remain unresolved).
- If no `fake remote playback device` has `selected state` set to `true`, the UA will behave as if the user has denied permission to use any `fake remote playback device`.
- When the `state` of a `RemotePlayback` object is `connected`, the UA SHOULD satisfy conditions relate the `local playback state`, the `media element state`, and the `fake remote playback state` described in [`Media commands and media playback state`](https://w3c.github.io/remote-playback/#media-commands-and-media-playback-state).

### FakeRemotePlayback dictionary
The `FakeRemotePlayback` dictionary provides information about a `fake remote playback device`.

```
dictionary FakeRemotePlayback {
  DOMString name;
  boolean selected;
  FakeRemotePlaybackState state;
}
```
- name: a string representing a `fake remote playback device`'s name.
- selected: a boolean that indicates a `fake remote playback device`'s `selected state`. When set to true, this `fake remote playback device` is a selected `fake remote playback device`.
- state: A `FakeRemotePlaybackState` representing a `fake remote playback device`'s `fake remote playback state`.

A `serialized fake remote playback device` is a JSON `Object` where a `fake remote playback`'s fields listed in the `FakeRemotePlayback` dictionary are mapped using the *JSON Key* and the associated field's value from the `fake remote playback devices` in the `current browsing context`.

### FakeRemotePlaybackInit dictionary
The `FakeRemotePlaybackInit` dictionary is used to initialize a `fake remote playback device`.
```
dictionary FakeRemotePlaybackInit {
  required DOMString name;
  boolean selected = true;
  FakeRemotePlaybackState state;
}
```
- name: a string that is used to set a `fake remote playback device`'s name.
- selected: a boolean that is used to set a `fake remote playback device`'s `selected state`.
- state: a `FakeRemotePlaybackState` that is used to set a `fake remote playback device`'s `fake remote playback state`.

### FakeRemotePlaybackState enumeration
A `FakeRemotePlaybackState` enumeration represents the `fake remote playback state`.
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

The `remote end steps` are:
1. Let *init* be the *init* parameter, `converted to an IDL value` of type `FakeRemotePlaybackInit`. If this throws an exception, return a `WebDriver error` with `WebDriver error code` `invalid argument`.
2. Let *name* be the *init*.`name`. If the `current browsing context` already has this *name* of `fake remote playback device`, return a `WebDriver error` with `WebDriver error code` `fake remote playback device already exists`.
3. If the `current browsing context` is `no longer open`, return a `WebDriver error` with `WebDriver error code` `no such window`.
4. `Handle any user prompts`, and return its value if it is a `WebDriver error`.
5. Run these sub-steps `in parallel` to add a `fake remote playback device` in `the current browsing context`:
    1. Let *fakeDevice* be a new `fake remote playback device`.
    2. Set *fakeDevice*'s `fake remote playback device name` to *name*.
    3. If *init*.`selected` is `true`, set this *fakeDevice* `selected state` to true.
    4. If *init*.`state` is present, set *fakeDevice*'s `fake remote playback state` to *init*.`state`.
    5. Otherwise, set *fakeDevice*'s `fake remote playback state` to `null`.
6. Return `success` with `data` *null*.

**Example:**
> To add a `selected fake remote playback device` named "ChromeCast" in the `current browsing context` of the session with ID 12, the `local end` would POST to `/session/12/fakeremoteplayback` with the body:
>```
>{
>  "name": "ChromeCast",
>  "selected": true,
>  "state": "playing"
>}
>```

### Get fake remote playback device
|HTTP Method|URI Template|
|----|----|
|GET|/session/{session id}/fakeremoteplayback/{name}|

The `get fake remote playback device` `extension command` retrieves information about a given name of `fake remote playback device`.

The `remote end steps` are:

1. If `the current browsing context` is `no longer open`, return a `WebDriver error` with `WebDriver error code` `no such window`.
2. `Handle any user prompts`, and return its value if it is a `WebDriver error`.
3. If the `url variable` *name* is not equal to a `fake remote playback device`'s name amongst all associated `fake remote playback devices` of `the current browsing context`, return a `WebDriver error` with `WebDriver error code` `no such fake remote playback device`.
4. Return `success` with the `serialized fake remote playback device` as data.



### Update fake remote playback device
// e.g. To change the selected attribute to test disconnection, change fake remote playback state to test reflection on local playback state.
|HTTP Method|URI Template|
|----|----|
|POST|/session/{session id}/fakeremoteplayback/|

The `update fake remote playback device` `extension command` updates properties about a given name of `fake remote playback device`.

The `remote end steps` are:

1. Let *init* be the *init* parameter, `converted to an IDL value` of type `FakeRemotePlaybackInit`. If this throws an exception, return a `WebDriver error` with `WebDriver error code` `invalid argument`.
2. If the *init*.`name` is not equal to a `fake remote playback device`'s name amongst all associated `fake remote playback devices` of `the current browsing context`, return a `WebDriver error` with `WebDriver error code` `no such fake remote playback device`.
3. Otherwise, let *fakeDevice* be the `fake remote playback device` with name *init*.`name`.
4. If the `current browsing context` is `no longer open`, return a `WebDriver error` with `WebDriver error code` `no such window`.
5. `Handle any user prompts`, and return its value if it is a `WebDriver error`.
6. Set *fakeDevice*'s `selected state` to *init*.`selected`.
7. If *init*.`state` is present, set *fakeDevice*'s `fake remote playback state` to *init*.`state`.
8. Return `success` with data *null*.


### Remove fake remote playback device
|HTTP Method|URI Template|
|----|----|
|DELETE|/session/{session id}/fakeremoteplayback/{name}|

The `remove fake remote playback device` `extension command` removes a given name of `fake remote playback device`.

The `remote end steps` are:

1. Let *name* be a `url variable`.
2. If the `current browsing context` is `no longer open`, return a `WebDriver error` with `WebDriver error code` `no such window`.
3. `Handle any user prompts`, and return its value if it is a `WebDriver error`.
4. If *name* is not equal to a `fake remote playback device`'s name amongst all associated `fake remote playback devices` of `the current browsing context`, return a `WebDriver error` with `WebDriver error code` `no such fake remote playback device`.
5. Delete *name* of `fake remote playback device` in the `current browsing context`.
6. Return `success` with data *null*.

### Remove all fake remote playback devices
|HTTP Method|URI Template|
|----|----|
|DELETE|/session/{session id}/fakeremoteplayback/|

The `remove all fake remote playback devices` `extension command` removes all existing `fake remote playback devices`.

The `remote end steps` are:

1. If the `current browsing context` is `no longer open`, return a `WebDriver error` with `WebDriver error code` `no such window`.
2. `Handle any user prompts`, and return its value if it is a `WebDriver error`.
3. Delete all `fake remote playback devices` in the `current browsing context`.
4. Return `success` with data *null*.

## Handling Errors
This section extends the `Errors` and defines extended `WebDriver error codes` specific for `fake remote playback device` in following table.

|Error Code|HTTP Status|JSON Error Code|Description|
|----------|-----------|---------------|-----------|
|***no such fake remote playback device***|404|`no such fake remote playback device`|no fake remote playback device matching the given name was found.|
|***fake remote playback already exists***|500|`fake remote playback device already exists`|A command to add a `fake remote playback device` could not be satisfied because the given name of `fake remote playback device` is already existed.|


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
