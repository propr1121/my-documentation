---
title: API Documentation
layout: default
nav_order: 8
last_modified_date: 2025-07-17
---

{% include freshness_indicator.html %}

# Stream Systems API Documentation

{: .fs-3 }
Complete API reference for Stream Systems financial technology platform, covering authentication, endpoints, and integration patterns for inter-dealer brokerage services.

---

## 🚀 Quick Start

### Authentication
All API requests require authentication using API keys and OAuth2 tokens.

```bash
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
     -H "Content-Type: application/json" \
     https://api.streamsystems.io/v1/trades
```

### Base URL
```
Production: https://api.streamsystems.io/v1/
Sandbox: https://sandbox-api.streamsystems.io/v1/
```

---

## 📋 Core API Endpoints

### Trading Operations

#### POST /trades
Execute a new trade through the FXOptEngine.

**Request:**
```json
{
  "instrument": "EUR/USD",
  "side": "BUY",
  "quantity": 1000000,
  "price": 1.0850,
  "client_id": "CLIENT_001",
  "trade_type": "SPOT"
}
```

**Response:**
```json
{
  "trade_id": "TRD_20250717_001",
  "status": "EXECUTED",
  "execution_time": "2025-07-17T14:30:15Z",
  "execution_price": 1.0851,
  "fees": {
    "commission": 25.00,
    "spread": 0.0001
  }
}
```

#### GET /trades/{trade_id}
Retrieve trade details and execution status.

#### GET /trades
List all trades with filtering options.

**Query Parameters:**
- `client_id`: Filter by client
- `instrument`: Filter by currency pair
- `status`: Filter by execution status
- `date_from`: Start date (ISO 8601)
- `date_to`: End date (ISO 8601)

### Market Data

#### GET /market-data/prices
Real-time pricing from PrePricer engine.

**Response:**
```json
{
  "timestamp": "2025-07-17T14:30:15Z",
  "prices": [
    {
      "instrument": "EUR/USD",
      "bid": 1.0849,
      "ask": 1.0851,
      "mid": 1.0850,
      "volatility": 0.12
    }
  ]
}
```

#### GET /market-data/historical
Historical price data for analysis.

### Client Management

#### GET /clients
List all authorized clients.

#### POST /clients
Register a new client for trading access.

#### GET /clients/{client_id}/positions
Current positions for a specific client.

---

## 🔒 Security & Compliance

### API Rate Limits
- **Standard**: 1000 requests/hour
- **Premium**: 5000 requests/hour
- **Enterprise**: Unlimited

### Regulatory Compliance
All API endpoints automatically:
- Generate UTI (Unique Trade Identifiers)
- Submit required regulatory reports
- Maintain audit trails for MiFID II compliance

### Error Handling

#### Standard Error Response
```json
{
  "error": {
    "code": "INVALID_INSTRUMENT",
    "message": "The specified instrument is not supported",
    "details": "EUR/GBP not available during market hours",
    "timestamp": "2025-07-17T14:30:15Z"
  }
}
```

#### HTTP Status Codes
- `200`: Success
- `400`: Bad Request
- `401`: Unauthorized
- `403`: Forbidden
- `404`: Not Found
- `429`: Rate Limit Exceeded
- `500`: Internal Server Error

---

## 📊 Webhooks & Real-time Data

### Trade Execution Webhooks
Receive real-time notifications for trade executions.

**Configuration:**
```json
{
  "webhook_url": "https://your-system.com/webhooks/trades",
  "events": ["TRADE_EXECUTED", "TRADE_FAILED", "TRADE_SETTLED"],
  "secret": "your_webhook_secret"
}
```

**Payload:**
```json
{
  "event": "TRADE_EXECUTED",
  "trade_id": "TRD_20250717_001",
  "timestamp": "2025-07-17T14:30:15Z",
  "data": {
    "instrument": "EUR/USD",
    "side": "BUY",
    "quantity": 1000000,
    "execution_price": 1.0851
  }
}
```

### WebSocket Streaming
Real-time market data streaming.

```javascript
const ws = new WebSocket('wss://api.streamsystems.io/v1/stream');

ws.onmessage = function(event) {
  const data = JSON.parse(event.data);
  console.log('Price update:', data);
};

// Subscribe to EUR/USD prices
ws.send(JSON.stringify({
  "action": "subscribe",
  "channel": "prices",
  "instruments": ["EUR/USD", "GBP/USD"]
}));
```

---

## 🔧 SDK & Code Examples

### Python SDK
```python
import streamsystems

client = streamsystems.Client(
    api_key='your_api_key',
    secret='your_secret',
    environment='production'
)

# Execute a trade
trade = client.trades.create(
    instrument='EUR/USD',
    side='BUY',
    quantity=1000000,
    price=1.0850
)

print(f"Trade executed: {trade.trade_id}")
```

### JavaScript SDK
```javascript
const StreamSystems = require('@streamsystems/api');

const client = new StreamSystems({
  apiKey: 'your_api_key',
  secret: 'your_secret',
  environment: 'production'
});

// Get real-time prices
const prices = await client.marketData.getPrices(['EUR/USD']);
console.log('Current EUR/USD:', prices[0].mid);
```

### curl Examples
```bash
# Get current positions
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
     https://api.streamsystems.io/v1/clients/CLIENT_001/positions

# Execute trade
curl -X POST \
     -H "Authorization: Bearer $ACCESS_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"instrument":"EUR/USD","side":"BUY","quantity":1000000}' \
     https://api.streamsystems.io/v1/trades
```

---

## 📈 Testing & Development

### Sandbox Environment
Full-featured testing environment with:
- Simulated market data
- Test client accounts
- Full API functionality
- No actual trades executed

### API Testing Tools
- **Postman Collection**: [Download](https://api.streamsystems.io/postman)
- **OpenAPI Specification**: [View Swagger](https://api.streamsystems.io/docs)
- **Interactive API Explorer**: [Try API](https://api.streamsystems.io/explorer)

### Test Data
```json
{
  "test_clients": [
    "TEST_CLIENT_001",
    "TEST_CLIENT_002"
  ],
  "test_instruments": [
    "EUR/USD",
    "GBP/USD",
    "USD/JPY"
  ]
}
```

---

## 🚨 Support & Troubleshooting

### Common Issues

#### Invalid Authentication
```json
{
  "error": {
    "code": "INVALID_TOKEN",
    "message": "Access token has expired",
    "solution": "Refresh your access token using the OAuth2 refresh flow"
  }
}
```

#### Rate Limiting
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests",
    "retry_after": 60
  }
}
```

### API Support
- **Technical Support**: Available 24/5 during market hours
- **Email**: api-support@streamsystems.io
- **Documentation Issues**: [GitHub Issues](https://github.com/streamsystems/api-docs)

### Status Page
- **API Status**: [status.streamsystems.io](https://status.streamsystems.io)
- **Incident Reports**: Real-time system status
- **Maintenance Windows**: Scheduled downtime notifications

---

*API Documentation last updated: {{ page.last_modified_date | date: "%B %d, %Y" }}*  
*API Version: v1.2.0*  
*Next scheduled review: {{ page.last_modified_date | date: "%s" | plus: 2592000 | date: "%B %d, %Y" }}* 