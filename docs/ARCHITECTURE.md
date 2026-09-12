# Architecture: Sovereign Spatial LiDAR

## Overview

**Package ID:** `PKG-023`  
**Domain:** Spatial Computing & Apple LiDAR  
**Microservice Port:** `8803`  
**n8n Webhook Path:** `spatial-lidar-trigger`  
**GitHub:** [BlackFoxgamingstudio/spatial-lidar](https://github.com/BlackFoxgamingstudio/spatial-lidar)

Apple LiDAR and Intel RealSense point cloud processing engine. Performs room scanning, object segmentation, dimensional measurement, and 3D model export.

---

## System Architecture

```
                     ┌──────────────────────────────────┐
                     │       Sovereign Spatial LiDAR       │
                     │       Port: 8803            │
                     ├──────────────┬───────────────────┤
   n8n Webhook ────▶ │  REST API    │   Core Engine     │
   HTTP POST         │  /api/v1/*   │   Dispatcher      │
                     └──────┬───────┴────────┬──────────┘
                            │                │
              ┌─────────────▼────────────────▼─────────┐
              │          Component Layer                 │
              │  PointCloudCaptu | ObjectSegmentor | DimensionalM  │
              └────────────────────────┬────────────────┘
                                       │
              ┌────────────────────────▼────────────────┐
              │      n8n Central Event Bus (:5678)       │
              └─────────────────────────────────────────┘
```

## Core Components

### `PointCloudCapture`
Handles all pointcloudcapture operations. Exposes async methods callable from the core dispatcher.

### `ObjectSegmentor`
Handles all objectsegmentor operations. Exposes async methods callable from the core dispatcher.

### `DimensionalMeasurer`
Handles all dimensionalmeasurer operations. Exposes async methods callable from the core dispatcher.

### `MeshReconstructor`
Handles all meshreconstructor operations. Exposes async methods callable from the core dispatcher.

### `ModelExporter`
Handles all modelexporter operations. Exposes async methods callable from the core dispatcher.

---

## API Contract

All interactions follow the SBB standard envelope:

```http
POST /api/v1/execute
Content-Type: application/json
X-SBB-API-Key: <api-key>

{
  "action": "<operation>",
  "payload": {},
  "trace_id": "optional-uuid"
}
```

**Success Response (HTTP 200):**
```json
{
  "status": "success",
  "data": {},
  "trace_id": "...",
  "timestamp": "2025-01-01T00:00:00Z"
}
```

**Health Check:**
```http
GET /health
→ {"status": "healthy", "service": "sovereign-spatial-lidar", "port": 8803}
```

## Integration Matrix

| System | Protocol | Direction | Purpose |
|--------|----------|-----------|---------|
| n8n Event Bus (:5678) | HTTP POST | Outbound | Event forwarding |
| n8n Webhook | HTTP POST | Inbound | Trigger execution |
| SBB Codebase Vault (:8766) | HTTP | Outbound | Code analysis |
| SBB Patterns Bible (:8794) | HTTP | Outbound | Standards validation |
| External APIs | HTTPS | Outbound | Domain-specific data |

## Deployment Architecture

```yaml
# docker-compose excerpt
sovereign-spatial-lidar:
  image: sovereign-spatial-lidar:latest
  ports: ["8803:8803"]
  healthcheck:
    test: curl -f http://localhost:8803/health
    interval: 30s
```

## Security Model

| Control | Implementation |
|---------|---------------|
| Authentication | `X-SBB-API-Key` header (env: `SBB_API_KEY`) |
| Rate Limiting | 100 req/min per client IP |
| Input Validation | Pydantic models (strict mode) |
| Container Security | Non-root user (`appuser:1001`) |
| Secrets | Environment variables only (never hardcoded) |
| TLS | Terminate at reverse proxy (nginx/caddy) |

## Tags
`lidar`, `point-cloud`, `ar`, `spatial`, `apple`
