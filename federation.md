[comment]: # (Copyright © 2022, Chris Moser - all rights reserved)
[comment]: # (Released under the Creative Commons Attribution-ShareAlike 4.0 International License)

# Yuforium Community Federation With ActivityPub
This document defines standard practices to federate communities using ActivityPub.

## Background
Federation with ActivityPub focuses on a social network model, which uses a user


There is no universally adopted method using ActivityPub for the inverse model -  to connect community content (forums, groups, etc.).

## Goals
- **Standardization & Simplicity**<br>
  Stay within the existing ActivityPub spec as much as possible and keep the implementation simple by adding a minimum amount of overhead

- **Discoverability**<br>
  Allow authoritative capabilities for discovery while providing an on-ramp to decentralization

- **Integration With Existing ActivityPub Services**<br>
  Compatibility with services like Mastodon for quick adoption

## Terminology
The terminology defined here is not expressly used to propose new Object types, but could be...

- `Topic` is the primary building block of Yuforium's federation model, and represents the subject matter of connection, such as as a favorite hobby or interest.

- `Community` represents an aggregatation of people, relationships, and content.  Usually this may be centered on various topics,   It can be an explicit representation or a broader network of connected communities.

- `Forum` represents a service endpoint that federates community activity and manages a set of users that can post to the forum and broadcast messages out to the larger community.

## Implementation

### Grouping Content with the Context Field
Yuforium uses the `context` field for federation, which is described in the Activity Streams `Object` specification as follows:

> The notion of "context" used is intentionally vague. The intended function is to serve as a means of grouping objects and activities that share a common originating context or purpose. An example could be all activities relating to a common project or event.

The `context` field is well suited for federating community content, because grouping of content is what a Community or a Forum does.

```json
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "id": "https://textiverse.com/object/123456",
  "type": "Note",
  "content": "Context groups objects and activities",
  "context": [
    {
      "type": "Actor",
      "id": "https://yuforium.com/topic/activitypub"
    }
  ]
}
```

Here we use `Actor` here to define a topic, but it could just as easily be an additional type, such as a `Topic`:

```json
{
  "@context": "https://yuforium.com/ns/activitypub",
  "id": "https://textiverse.com/topic/dodgeball",
  "name": "Dodgeball",
  "summary": "Anything related to the sport of Dodgeball"
}
```

Multiple contexts can be used, enabling cross network federation.  The following `Note` will span across community networks that discuss dodgeball and wrenches:

```json
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "id": "https://textiverse.com/object/5678",
  "type": "Note",
  "content": "If you can dodge a wrench, you can dodge a ball!",
  "context": [
    {
      "type": "Topic",
      "id": "https://yuforium.com/topic/dodgeball"
    },
    {
      "type": "Topic",
      "id": "https://yuforium.com/topic/wrenches"
    }
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

Or specific contexts may be appended or modified as needed when submitting content through a instance:

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

### Federation Amongst Instances
Now that we have defined the purpose of the context field, we can discuss federation across instances using this model.

```json
{
  "type": "Service",
  "name": "Piano Players Forum LA",
  "id": "https://textiverse.com/forum/piano-players",
  "context": "https://textiverse.com/community/
}
```

#### Topic and Community Followers

### Authoritative Topics

### Non Authoritative Topics


_Post to a forum from your Activity Stream by addressing the forum_
```json
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "id": "https://mastodon.social/cpmoser/note/some-id",
  "type": "Note",
  "name": "First Post by Chris",
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

_You can also POST directly to the forum outbox like your own and it will be excluded from your Activity Stream (should require authentication)_
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

### Resource Types TL;DR
- A topic is a focal point (purpose) for information exchange
- A forum is a public endpoint on a server and distributes content with other forums and the larger community
- A community is an ephemeral representation of all forums that are joined together by a topic

### Topics
Topics are the glue that forums use to share information.  They can be considered loosely authoritative entities for the context that they represent:

```json
{
  "id": "https://yuforia.com/topic/anything",
  "summary": "Only ANYTHING can be discussed here"
}
```

The concept of authoritative entities may go against the idea of federation, therefore:
- Forums can have multiple contexts
- Forums aren't required to follow the rules of their respective contexts - the caveat is that other forums (or the authoritative topic) may see them as "unofficial" or unvetted participants.

A topic can be completely unauthoritative, for example:
```json
{
  "id": "https://yuforia.com/forum/anything",
  "context": {
    "name": "anything"
  }
}
```
Without an authoritative context, however, a forum will need to manually `follow` another forum to distribute content.


### Forums
Forums are conduits to the larger, distributed community.  They are endpoints that can be a source

## Notes
- A history of a community should be maintained so that content made in the community represents the `topics` of the community at the time it is made.
- No consideration as yet had been made to <b>Private Communities</b> but we could use public key encryption to pass content around

## About
Yuforium is built on the excellent and Angular friendly NestJS framework.
