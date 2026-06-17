# Camino Matrix App Service

The **Camino Matrix App Service** is an extension component of the Camino Messenger network. It runs alongside the Matrix homeserver (`camino-conduit`) to process, audit, and validate message transfers, focusing on fee compliance (Network Fees) and bot behavior.

---

## Architectural Role & Function

```
                  ┌────────────────────────┐
                  │   Matrix Homeserver    │
                  │   (camino-conduit)     │
                  └───────────┬────────────┘
                              │
               Relays ALL events (namespace: .*)
                              │
                              ▼
           ┌──────────────────────────────────┐
           │    Camino Matrix App Service     │
           │                                  │
           │  1. Verifies signed message sigs │
           │  2. Tracks message chunks        │
           │  3. Enforces bot compliance      │
           └──────────────────┬───────────────┘
                              │
                      If check fails
                              │
                              ▼
           ┌──────────────────────────────────┐
           │        Ban/Mute Action           │
           │   (Currently Not Enforced/Stub)   │
           └──────────────────────────────────┘
```

1. **Event Capture**:
   - The App Service is registered automatically by `camino-conduit` at startup with access to the global namespace (`.*`).
   - It captures `matrix.EventTypeSignedMessage` and `matrix.EventTypeMessageChunk` events from all rooms.
2. **Signature Verification**:
   - For every transaction message, the App Service calls `eventContent.Verify()` to validate the sender's signature.
3. **Chunk Tracking**:
   - Large payloads (such as extensive search results) are split and sent in chunks (`MessageChunk`). The App Service tracks chunk counts to prevent spam or misreporting.

---

## Current Enforcement Status

> [!NOTE]
> **Not Enforced Yet**: While the App Service processes and flags invalid signatures for user banning, the actual banning execution is currently a no-op stub:
> ```go
> // TODO @evlekht implement (next ticket) // persist with db, make it durable? not just call it from event receiver?
> func (s *service) banUser(_ context.Context, _ id.UserID) error {
>     return nil
> }
> ```
> Bots sending invalid or missing signatures will generate warnings in the logs, but are not actively muted or blocked on the network at this stage.

---

## Configuration & Deployment

### Docker Build
Build the Docker image with:
```bash
docker build -t c4tplatform/camino-matrix-app-service .
```

### Registration configuration
- **Conduit Integration**: When using `camino-conduit`, registration is automated at startup using settings in `conduit.toml` (`camino_app_service_url`, `camino_app_service_as_token`, `camino_app_service_hs_token`).
- **Synapse Integration**: If running with a standard Synapse server, register by placing the appservice registration yaml file at `files/matrix/.synapse/camino.yaml` (see [example/config/synapse/camino.yaml](example/config/synapse/camino.yaml)).

### App-Service configuration
The app-service expects its configuration yaml at `/camino-matrix-app-service/camino-matrix-app-service.yaml`.
Refer to [example/config/camino-matrix-app-service.yaml](example/config/camino-matrix-app-service.yaml) for a configuration template.
