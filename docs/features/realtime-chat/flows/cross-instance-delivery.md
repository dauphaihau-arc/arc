# Cross-Instance Delivery

This flow describes how realtime chat events reach subscribers connected to a different API instance.

```mermaid
sequenceDiagram
    participant APIA as API Instance A
    participant Broker as Realtime Broker
    participant APIB as API Instance B
    participant Recipient as Recipient Storefront

    Recipient->>APIB: Connected and subscribed to conversation
    APIA->>APIA: Message is created
    APIA->>Broker: Publish conversation event
    Broker-->>APIB: Forward conversation event
    APIB-->>Recipient: Push new message event
```

Summary:

- realtime delivery must work when different users are connected to different API instances
- the broker carries room events across instances
- conversation delivery remains addressed by the logical conversation subscription, not by a specific server process
