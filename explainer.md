# Gesture Delegation Explained

## Objective
Some features on the platform require a user gesture in order to be triggered. Among these features, some of them require a
user gesture to have happened on the document. Gating features to user gestures is meant to reduce user annoyance but can
hurt the ecosystem when the features are triggered from an iframe. Indeed, iframes may be used as components of a websites
and can be used to sandbox third party code. Allowing these iframes to be able to use the user gesture information from the
embedder would still allow the embedder to limit the behaviour of an iframe without breaking entirely the intended
experience.

## Use Cases
The use cases are currently limited to features that require a user gesture on a document, not a user gesture on the stack of
the call. There are two cases of these on the platform at the moment:
- The Vibration API can only be triggered if the document was interacted with at some point in the past;
- Some browsers implement autoplay limitations based on the document being interacted with.

The API's intent is to allow both of these features to usable by an iframe if the embedder received a user gesture or if its
embedder also delegated user gesture and received user gesture.

For example, a website could embed a video player in an iframe and would like the player to start playback when the user
interact on the main frame.

## Proposed API
The API is meant to be future facing, allowing for new features to gate their behaviour on user gestures while staying
simple, in order to reduce the burden on the embedder.

The attribute will take the form of an attribute that could be set on the `iframe` element by the embedder. The attribute
will have known keywords that, if set, will allow the delegation of user gesture to the iframe for a given feature.

```javascript
partial interface HTMLIFrameElement {
  [SameObject, PutForwards=value] readonly attribute DOMTokenList allowedGestureDelegation;
};
```

The `allowedGestureDelegation` known keywords are:
`vibration`, which delegates user gesture to the iframe in the context of the Vibration API.
`media`, which delegates user gesture to the iframe in the context of media playback (ie. autoplay).

If a feature's associated keyword is not present, delegation will not apply. In other words, the API will not offer a way to
allow all types of user gesture delegation to an iframe.

## Example

### Allow an inserted iframe to play immediately

This example assumes that the user agent requires a user gesture on the document in order to allow for media playback.

```html
<button>play</button>
<div id='player-container'></div>
<script>
  document.querySelector('button').addEventListener('click', () => {
    var frame = document.createElement('iframe');
    frame.src = '/my_player.html';
    frame.allowedGestureDelegation = 'media';
    document.querySelector('#player-container').appendChild(frame);
  }, { once: true });
</script>
```
