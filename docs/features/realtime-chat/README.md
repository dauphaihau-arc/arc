# Realtime Chat

## Overview

Realtime Chat lets storefront clients receive new conversation messages over the authenticated Socket.IO gateway.

The feature uses REST for authoritative message fetches and mutations, then uses WebSocket events for low-latency delivery after messages are created.

For the detailed websocket lifecycle, see [flow.md](flow.md).

## Purpose

This document defines the realtime delivery contract for clients that connect to the chat gateway and subscribe to conversation rooms.

It covers:

- how a chat client connects to the websocket gateway
- how the API authenticates the socket connection
- how the API authorizes `conversation.subscribe`
- how the socket joins a conversation room
- how new chat messages are pushed back to subscribed clients in realtime

## Scope

In scope:

- Socket.IO namespace `/ws`
- cookie-authenticated websocket connections
- user-scoped websocket rooms
- conversation room subscription and authorization
- chat message fan-out after REST message creation
- Redis-backed room fan-out across API instances

Out of scope:

- message history pagination
- conversation creation
- read receipt design
- seller/admin chat inbox UX
- push notification delivery

## Client Contract

The client is responsible for:

- connecting to `/ws` with the current authenticated session
- subscribing to the active conversation before expecting conversation-scoped events
- unsubscribing when the active conversation changes or the chat surface unmounts
- using REST for authoritative message fetches and writes
- using websocket events for incremental realtime updates

## Authorization Model

The websocket flow has two distinct authorization steps:

1. Socket connection authentication
2. Conversation room authorization

The first step proves who the user is. The second step proves that this user is allowed to join a specific chat conversation room.

Conversation subscription is the critical security boundary for room membership. A client cannot join an arbitrary conversation room just by knowing a conversation id. The socket user must be the buyer, the seller for that shop conversation, or an admin.

## Event Shape

Realtime delivery publishes a websocket event of type `message` with a payload shaped like:

```json
{
  "event_type": "chat.message.created",
  "conversation_id": "conversation-id",
  "shop_id": "shop-id",
  "message": {
    "id": "message-id",
    "body": "Hello",
    "sender_user_id": "user-id",
    "occurred_at": "2026-06-04T00:00:00.000Z",
    "metadata": null
  }
}
```

Storefront maps that payload into its local chat message state.

## Why Both REST And WebSocket Are Used

Chat clients typically use both:

- REST for authoritative fetch and mutation
- WebSocket for incremental realtime delivery

REST is still responsible for:

- loading message history
- sending a message
- marking a conversation as read

WebSocket is responsible for:

- pushing newly created messages without waiting for polling

Some clients may still keep periodic refetches as a fallback, but the websocket path is the low-latency update channel.

## Related Documents

- [Flow](flow.md)
