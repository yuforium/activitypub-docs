[comment]: # (Copyright © 2022, Chris Moser - all rights reserved)
[comment]: # (Released under the Creative Commons Attribution-ShareAlike 4.0 International License)

# Yuforium Community Federation With Activity Pub
*Purpose statement here*

## Goals
- *Standardization*<br>
  Stay within the existing ActivityPub spec as much as possible

- *Simplicity*<br>
  Keep implementation simple by adding a minimum amount of overhead

- *Discoverability*<br>
  Allow authoritative capabilities for discovery while providing an on-ramp to decentralization

- *Integration*<br>
  Compatibility with existing services  like Mastodon for quick adoption

## Terminology
- `Topic` is the building block of Yuforium's federation model, and represents the subject matter of connection, such as as a favorite hobby or interest. It is used as `context` in Yuforium federation.

- `Community` represents an aggregate of people, relationships, and content based on a collection of topics.  It can be an explicit representation or a broader network of connected communities.  It is also be used as a `context` in Yuforium's implementation.

- `Forum`<br>

- `Thread`<br>

## Notes
- A history of a community should be maintained so that content made in the community represents the `topics` of the community at the time it is made.

## Implementation
Yuforium uses the `context` field for federation, which is described in the Activity Streams `Object` specification as follows:

> The notion of "context" used is intentionally vague. The intended function is to serve as a means of grouping objects and activities that share a common originating context or purpose. An example could be all activities relating to a common project or event.

A community context can be described in various ways.  For example, a community can be centered around discussion of several topics (e.g. a favorite hobby, a sport or team), or it can be centered around a specific group of people such as a neighborhood associaationspecific topics, or even a community itself, Yuforium adds a `/topic` endpoint to provide a shared context for federated communities.

Before we begin, let's clarify that:
- Yuforium is code and standards, Yuforia.com is a service
- `@context` is not the same as `context`

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
    "https://yuforia.com/forum/anything"
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


## Private Communities
We could use public key encryption to pass content around.

## About
Yuforium is built on the excellent and Angular friendly NestJS framework.

<p align="center">
  <a href="http://nestjs.com/" target="blank"><img src="https://nestjs.com/img/logo_text.svg" width="320" alt="Nest Logo" /></a>
</p>

[travis-image]: https://api.travis-ci.org/nestjs/nest.svg?branch=master
[travis-url]: https://travis-ci.org/nestjs/nest
[linux-image]: https://img.shields.io/travis/nestjs/nest/master.svg?label=linux
[linux-url]: https://travis-ci.org/nestjs/nest

  <p align="center">A progressive <a href="http://nodejs.org" target="blank">Node.js</a> framework for building efficient and scalable server-side applications, heavily inspired by <a href="https://angular.io" target="blank">Angular</a>.</p>