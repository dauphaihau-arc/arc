# Realtime Chat

## Overview

Realtime Chat lets storefront and seller clients receive new conversation messages over the authenticated Socket.IO gateway.

The feature uses REST for authoritative message fetches and mutations, then uses WebSocket events for low-latency delivery after messages are created.

Chat conversations are scoped to a buyer and a shop. A product opened from storefront chat is represented as a generated `product_reference` message instead of a conversation-level product field.

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
- storefront and seller chat inboxes
- conversation creation and product reference messages
- cursor-based message history pagination
- unread count projection for buyer and seller views
- chat message fan-out after REST message creation
- Redis-backed room fan-out across API instances

Out of scope:

- admin chat inbox UX
- push notification delivery

## Client Contract

The client is responsible for:

- connecting to `/ws` with the current authenticated session
- subscribing to the active conversation before expecting conversation-scoped events
- unsubscribing when the active conversation changes or the chat surface unmounts
- using REST for authoritative message fetches and writes
- using REST cursors to load older messages before the currently loaded window
- using websocket events for incremental realtime updates
- deduplicating messages received from both REST and websocket paths
- patching conversation previews and unread counts from created-message events until the next authoritative refetch
- refetching message history after product-reference realtime events so product-card pricing comes from the REST read model

## Authorization Model

The websocket flow has two distinct authorization steps:

1. Socket connection authentication
2. Conversation room authorization

The first step proves who the user is. The second step proves that this user is allowed to join a specific chat conversation room.

Conversation subscription is the critical security boundary for room membership. A client cannot join an arbitrary conversation room just by knowing a conversation id. The socket user must be the buyer, the seller for that shop conversation, or an admin.

## REST Contract Notes

Conversation summaries include buyer, shop, latest message, read markers, and unread counts:

```json
{
  "id": "conversation-id",
  "buyer_user_id": "buyer-user-id",
  "buyer": {
    "id": "buyer-user-id",
    "display_name": "Buyer Name",
    "avatar": null
  },
  "shop": {
    "id": "shop-id",
    "owner_user_id": "seller-user-id",
    "shop_name": "Shop Name",
    "slug": "shop-slug"
  },
  "status": "open",
  "last_message": {
    "id": "message-id",
    "body_preview": "Hello",
    "sender_user_id": "buyer-user-id",
    "message_type": "text",
    "created_at": "2026-06-04T00:00:00.000Z"
  },
  "last_message_at": "2026-06-04T00:00:00.000Z",
  "last_message_sender_user_id": "buyer-user-id",
  "buyer_last_read_at": "2026-06-04T00:00:00.000Z",
  "seller_last_read_at": null,
  "buyer_unread_count": 0,
  "seller_unread_count": 1,
  "created_at": "2026-06-04T00:00:00.000Z",
  "updated_at": "2026-06-04T00:00:00.000Z"
}
```

Message history uses cursor pagination:

- request the latest message window with `limit`
- request older messages with `before=<page_info.before_cursor>`
- responses return messages in ascending display order
- `page_info.has_more_before` tells the client whether another older page exists
- product-reference message responses may overlay stored snapshot money with viewer-specific display pricing

```json
{
  "conversation": {},
  "results": [],
  "limit": 50,
  "page_info": {
    "has_more_before": true,
    "before_cursor": "cursor"
  }
}
```

Conversation list pagination still uses page-based metadata.

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
    "message_type": "text",
    "sender_user_id": "user-id",
    "occurred_at": "2026-06-04T00:00:00.000Z",
    "metadata": null
  }
}
```

Storefront and seller clients map that payload into local chat message state.

For product references, the API emits a normal `chat.message.created` event with `message_type: "product_reference"` and product snapshot metadata:

```json
{
  "event_type": "chat.message.created",
  "conversation_id": "conversation-id",
  "shop_id": "shop-id",
  "message": {
    "id": "message-id",
    "body": "Product title",
    "message_type": "product_reference",
    "sender_user_id": "buyer-user-id",
    "occurred_at": "2026-06-04T00:00:00.000Z",
    "metadata": {
      "product_reference": {
        "product_id": "product-id",
        "snapshot": {
          "title": "Product title",
          "shop_slug": "shop-slug",
          "product_slug": "product-slug",
          "image_storage_key": "products/card.jpg",
          "amount_minor": 1299,
          "original_amount_minor": 1599,
          "currency": "USD"
        },
        "current": {
          "status": "published",
          "in_stock": true,
          "stock": 12
        }
      }
    }
  }
}
```

Product-reference snapshot metadata is fallback context, not the final pricing authority for rendered product cards.

REST message reads are authoritative for product-card display money:

- storefront/buyer chat reads resolve product-card money using buyer presentment currency
- seller/shop chat reads resolve product-card money using seller-authored base catalog pricing
- order, payment, refund, and support workflows must use order money snapshots instead of live product-reference pricing

The websocket event carries the newly created message quickly, including the stored snapshot metadata. Because that event is shared across buyer and seller recipients, clients must not treat event snapshot money as final viewer-specific display pricing. Product-reference realtime events should invalidate or refetch the active message history so the rendered card uses the REST-enriched message response.

## Why Both REST And WebSocket Are Used

Chat clients typically use both:

- REST for authoritative fetch and mutation
- WebSocket for incremental realtime delivery

REST is still responsible for:

- loading message history
- loading conversation lists and unread counts
- creating or opening a conversation
- sending a message
- marking a conversation as read

WebSocket is responsible for:

- pushing newly created messages without waiting for polling
- keeping active message panes, conversation previews, and unread badges current between REST refetches
- triggering an authoritative REST refetch when a product-reference event needs viewer-specific product-card pricing

Some clients may still keep periodic refetches as a fallback, but the websocket path is the low-latency update channel.

## Related Documents

- [Flow](flow.md)
