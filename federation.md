[comment]: # (Copyright © 2022, Chris Moser - all rights reserved)
[comment]: # (Released under the Creative Commons Attribution-ShareAlike 4.0 International License)

# Yuforium Community Federation With ActivityPub
This document defines standard practices to federate communities using ActivityPub.

## Goals
- **Standardization & Simplicity**<br>
  Stay within the existing ActivityPub spec as much as possible and keep the implementation simple by adding a minimum amount of overhead

- **Discoverability**<br>
  Allow authoritative capabilities for discovery while providing an on-ramp to decentralization

- **Integration With Existing ActivityPub Services**<br>
  Compatibility with services like Mastodon for quick adoption

## Terminology
Terminology in this document is not expressly used to propose new types, and can be represented as `Actor` or `Service` types.  However for clarity we will define the following:

- `Community` is the primary building block of Yuforium's federation model, and represents the origination of an `Object` type through any outbox associated with federation.  Like the `context` field of an `Object`, the definition of `Community` is intentionally vague, and represents a subject matter of connection, such as as a favorite hobby or interest, or a group of friends.  Regardless of _how_ the community is defined, the **origination** represents the motivation for a person to contribute to the community.

- `Forum` represents a service endpoint that federates community activity and manages a set of users that can post content to the forum outbox and broadcast messages to the larger, federated `Community`.  This type of resource may also include moderators who manage the content that appears on the instance.

## Grouping Content with the Context Field
Yuforium uses the `context` field for federation, which is described in the Activity Streams `Object` specification as follows:

> The notion of "context" used is intentionally vague. The intended function is to serve as a means of grouping objects and activities that share a common originating context or purpose. An example could be all activities relating to a common project or event.

The `context` field is well suited for federating community content, because grouping of content is what a Community or a Forum does.

```json
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "id": "https://textiverse.com/object/123456",
  "type": "Note",
  "content": "The context field groups objects that have a common originating purpose together, change my mind.",
  "context": "https://yuforium.com/community/activitypub-developers"
}
```

Multiple contexts can be used, enabling cross network federation.  The following `Note` can span across multiple communities.  In this example, the Note is relevant to both the `ActivityPub Developers` community on Yuforium and the `Typescript Developers` community on Textiverse:

```json
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "id": "https://textiverse.com/object/5678",
  "type": "Note",
  "content": "Hi, I am implementing an ActivityPub server in Typescript",
  "context": [
    "https://yuforium.com/community/activitypub-developers",
    "https://textiverse.com/community/typescript-developers"
  ]
}
```

A `Community` encompasses a group of topics.  A community can include information that a topic on its own cannot provide, such as a `Place`, which may be relevant to the community but not necessarily to the group of topics it represents:

```json
{
  "type": "Community",
  "@context": "https://www.w3.org/ns/activitystreams",
  "id": "https://textiverse.com/community/piano-players",
  "name": "Santa Monica Piano Players",
  "context": [
    {
      "type": "Topic",
      "id": "https://yuforium.com/topic/music"
    },
    {
      "type": "Topic",
      "id": "https://yuforium.com/topic/piano-music"
    }
  ],
  "place": {
    "@context": "https://www.w3.org/ns/activitystreams",
    "type": "Place",
    "name": "Santa Monica, CA",
    "latitude": 34.765,
    "longitude": -118.832,
    "radius": 10,
    "units": "miles"
  }
}
```
Content created with the given `Community` may be assumed to be relevant to one or more of the grouped topics specified.  The `published` field would indicate the state of the community at that time, thus a `Community` should manage a history of its relevant topics should they change over time:

```json
{
  "type": "Note",
  "content": "My favorite piece to play",
  "context": "https://textiverse.com/community/piano-players",
  "published": "2022-04-20T03:13:37Z"
}
```

Or specific contexts may be automatically appended or user selected as needed when submitting content through an instance:

```json
{
  "type": "Note",
  "content": "My favorite piece to play",
  "context": [
    "https://textiverse.com/community/piano-players",
    "https://yuforium.com/topic/sheet-music"
  ]
}
```

## Discovery with Topics and Communities
Given that the `Topic` and `Community` types discussed earlier extend the `Actor` type, either one can contain followers:

```json
{
  "type": "Topic",
  "id": "https://yuforium.com/topic/hiking",
  "inbox": "https://yuforium.com/topic/hiking/inbox",
  "followers": "https://yuforium.com/hiking/followers"
}
```
A `Forum` or `Service` instance can follow a topic:

```json
{
  "type": "Follow",
  "summary": "Textiverse Hiking Forum follows the Hiking Topic",
  "actor": "https://textiverse.com/forum/hiking",
  "object": "https://yuforium.com/topic/hiking"
}
```

On accepting the follow, the `Topic` can notify its other followers of the newly added instance:

```json
{
  "type": "Add",
  "summary": "New instance added to the Hiking Topic",
  "actor": "https://yuforium.com/topic/hiking",
  "to": [
    "https://www.w3.org/ns/activitystreams#Public",
    "https://yuforium.com/topic/hiking/followers"
  ],
  "target": "https://yuforium.com/topic/hiking/followers",
  "object": "https://textiverse.com/forum/hiking"
}
```

## Federation
By following a `Topic`, a `Forum` now has the ability to connect with other instances that can federate content based on the `context` field.  For example, consider the following instance:

```json
{
  "type": "Forum",
  "id": "https://textiverse.com/forum/hiking",
  "summary": "Textiverse Hiking Forum",
  "context": [
    "https://yuforium.com/topic/hiking"
  ]
}
```

Through a broadcast of the `Add` activity from the `Topic`, this forum is now aware of another instance that also follows the `hiking` topic:

```json
{
  "type": "Forum",
  "id": "https://hiking-forum.io",
  "context": [
    "https://yuforium.com/topic/hiking",
    "https://yuforium.com/topic/backpacking"
  ]
}
```

And sends a follow request to the Hiking-Forum.io instance:

```json
{
  "type": "Follow",
  "summary": "Textiverse Hiking Forum follows Hiking-Forum.io",
  "actor": "https://textiverse.com/forum/hiking",
  "object": "https://hiking-forum.io",
  "context": "https://yuforium.com/topic/hiking"
}
```

Now when content is created on the `Hiking-Forum.io` instance, the `Textiverse Hiking Forum` will receive that content in its own inbox, and that content is now available on that instance.

There are two things to note about this specific `Follow` activity:

- The `Follow` activity itself contains a context.  While not required, it can be used to limit the scope of activities sent to the follower.  In the example above, `Hiking-Forum.io` should not send any activites that do not include the `hiking` topic context.
- Activities are not limited to content creation, and could include other activities of the instance, such as `Delete` and `Edit` activities.  Moderation of an instance itself could be considered a `Service`, and could be considered another part of the `Forum` instance that would be followed separately.

## Creating Activities on the Forum Instance
A `Forum` instance can manage its own users that should have the ability to post to the outbox of the instance.

_Post to a forum from your Activity Stream by addressing the forum_
```json
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "id": "https://mastodon.social/cpmoser/note/some-id",
  "type": "Note",
  "name": "First Post by Chris from Mastodon",
  "content": "Hi world!",
  "to": [
    "https://www.w3.org/ns/activitystreams#Public",
    "https://textiverse.com/forum/anything"
  ]
}
```
_A new Note is created in the forum, referencing the original via "delegatedFor" and adding the default context of the forum_
```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://yuforium.com/community/v1"
  ],
  "id": "https://yuforia.com/forum/anything/note/another-id",
  "type": "Note",
  "delegatedFor": "https://mastodon.social/cpmoser/note/some-id",
  "context": "https://yuforium.com/topic/anything"
}
```

It's not necessary to pass through the `delegatedFor` property through the community at large but it should remain with the forum's Note.

_You can also POST directly to the forum outbox like your own and it will be excluded from your Activity Stream (requires authentication through the Forum)_
```json
{
  "type": "Note",
  "name": "A note",
  "to": [
    "https://www.w3.org/ns/activitystreams#Public"
  ]
}
```

_Forums encapsulate the context_
```json
{
  "type": "Service",
  "name": "The anything community",
  "summary": "Representation of /topic/anything on a specific site",
  "context": [
    "https://yuforium.com/topic/anything",
    "https://yuforium.com/topic/general"
  ]
}
```

## Non Authoritative (Decentralized) Topics
Because a context can be represented by a base `Object` type, and not an `Actor`, we can make a topic completely unauthoritative, for example:

```json
{
  "id": "https://textiverse.com/forum/anything",
  "context": {
    "name": "anything"
  }
}
```
Without an authoritative context, however, a forum will need to manually `Follow` another forum to distribute content.

## Notes
- A history of a community should be maintained so that content made in the community represents the `topics` of the community at the time it is made.
- No consideration as yet had been made to *Private Communities* but we could use public key encryption to distribute content.
