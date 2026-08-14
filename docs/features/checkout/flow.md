# Checkout Flow

This document is the entry point for end-to-end checkout lifecycle examples across cart checkout, buy-now checkout, buyer identity modes, payment returns, and recoverable failures.

For feature scope, user-visible states, invariants, and implementation notes, see [README.md](README.md).

## Main Flow

```mermaid
sequenceDiagram
    actor Buyer
    participant Web as Storefront Web
    participant Quote as Checkout Quote API
    participant Order as Order Creation API
    participant Payment as Payment Gateway

    Buyer->>Web: Start cart or buy-now checkout
    Buyer->>Web: Enter shipping and payment details
    Buyer->>Web: Review and confirm
    Web->>Quote: Create checkout quote
    Quote->>Quote: Resolve checkout currency, totals, and item snapshots
    Quote-->>Web: quote_id and quoted totals
    Web->>Order: Create order with quote_id and payment_type
    alt Card payment
        Order->>Payment: Create checkout session
        Payment-->>Order: Checkout session URL
        Order-->>Web: checkout_session_url
        Web-->>Buyer: Redirect to payment provider
    else Cash payment
        Order-->>Web: Created order shops
        Web-->>Buyer: Show success page
    end
```

Summary:

- checkout has one shared boundary for cart and buy-now: the storefront creates a quote first, then creates orders from the returned `quote_id`
- checkout quote creation owns checkout currency, totals, and item snapshots
- order creation consumes `quote_id` plus `payment_type`
- card payment redirects through a payment-provider checkout session
- cash payment confirms after local order creation returns created order shops

## Main Flow With Inventory Service

```mermaid
sequenceDiagram
    actor Buyer
    participant Web as Storefront Web
    participant Quote as Checkout Quote API
    participant Inventory as Inventory Service
    participant Order as Order Creation API
    participant Outbox as Order Inventory Outbox
    participant RabbitMQ
    participant Payment as Payment Gateway

    Buyer->>Web: Start cart or buy-now checkout
    Buyer->>Web: Enter shipping and payment details
    Buyer->>Web: Review and confirm
    Web->>Quote: Create checkout quote
    Quote->>Quote: Resolve checkout currency, totals, and item snapshots
    Quote->>Inventory: POST /inventory/reservations/quote
    alt Reservation accepted
        Inventory-->>Quote: reservationId and ACTIVE status
        Quote->>Quote: Store reservationId on checkout quote
        Quote-->>Web: quote_id, reservation-backed totals, expires_at
    else Reservation failed
        Inventory-->>Quote: RESERVATION_FAILED
        Quote-->>Web: Stock or reservation recovery error
    end

    Web->>Order: Create order with quote_id and payment_type
    Order->>Quote: Reload quote and reservationId
    Order->>Inventory: POST /inventory/reservations/validate
    alt Reservation is active
        Inventory-->>Order: valid ACTIVE reservation
        Order->>Order: Persist order shops
        Order->>Outbox: Persist order.created inventory event
        Outbox-->>RabbitMQ: Publish order.created
        RabbitMQ-->>Inventory: Deliver order.created
        Inventory->>Inventory: Mark reservation SOLD
        alt Card payment
            Order->>Payment: Create checkout session
            Payment-->>Order: Checkout session URL
            Order-->>Web: checkout_session_url
            Web-->>Buyer: Redirect to payment provider
        else Cash payment
            Order-->>Web: Created order shops
            Web-->>Buyer: Show success page
        end
    else Reservation invalid or unavailable
        Inventory-->>Order: invalid reservation status
        Order-->>Web: CHECKOUT_QUOTE_RESERVATION_UNAVAILABLE
        Web-->>Buyer: Show reservation recovery copy
    end
```

Summary:

- when `INVENTORY_RESERVATION_DRIVER=remote`, checkout uses the Go `inventory-service` as the stock reservation owner
- quote creation calls `POST /inventory/reservations/quote` and stores the returned `reservationId` on the checkout quote
- order creation validates the stored reservation through `POST /inventory/reservations/validate` before persisting order shops
- NestJS records an `order.created` inventory event in the order inventory outbox after quote-backed orders are created
- the outbox publisher publishes `order.created` through RabbitMQ, and `inventory-service` consumes it to transition the reservation to `SOLD`
- if the remote reservation is missing, inactive, expired, released, or mismatched, checkout returns the reservation-unavailable recovery path

## Detailed Flows

- [Cart Checkout](flows/cart-checkout.md)
- [Buy-Now Checkout](flows/buy-now-checkout.md)
- [Authenticated Buyer](flows/authenticated-buyer.md)
- [Guest Buyer](flows/guest-buyer.md)
- [Card Payment Return](flows/card-payment-return.md)
- [Cash Payment Success](flows/cash-payment-success.md)
- [Recoverable Failure Flow](flows/recoverable-failure.md)
- [Target Async Card Checkout](flows/target-async-card-checkout.md)
