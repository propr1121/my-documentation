---
title: API Quick Start Guide
layout: default
nav_order: 3
last_modified_date: 2025-07-17
---

{% include freshness_indicator.html %}

# API Quick Start Guide

{: .fs-3 }
Get started with **Stream Systems APIs** in minutes
{: .fs-6 .fw-300 }

---

## Authentication

{: .highlight-box }
All **Stream Systems API endpoints** require **valid authentication tokens**. Contact your **account manager** to obtain **API credentials**.

### API Token Setup

<div class="content-section" markdown="1">

#### **1. Obtain Your Credentials**
Contact **Stream Systems support** to get your:
- **API Key**: Your unique identifier
- **Secret Token**: For request signing  
- **Environment URLs**: Production and sandbox endpoints

#### **2. Authentication Headers**
```http
Authorization: Bearer YOUR_API_TOKEN
Content-Type: application/json
X-API-Version: 2024-01
```

</div>

---

## Example API Calls

### Market Data Request

{: .financial-data }
**Endpoint**: `GET /api/v1/market-data`

<div class="content-section" markdown="1">

**Request Example:**
```json
{
  "symbol": "EURUSD",
  "timeframe": "1M", 
  "start_date": "2024-01-01T00:00:00Z",
  "end_date": "2024-01-01T23:59:59Z"
}
```

**Response Example:**
```json
{
  "status": "success",
  "data": {
    "symbol": "EURUSD",
    "prices": [
      {
        "timestamp": "2024-01-01T00:00:00Z",
        "bid": 1.1045,
        "ask": 1.1047,
        "mid": 1.1046
      }
    ]
  },
  "timestamp": "2024-01-15T10:30:00Z",
  "request_id": "req_12345"
}
```

</div>

### Trading Operations

<div class="content-section" markdown="1">

| Operation | Method | Endpoint | Description |
|-----------|--------|----------|-------------|
| **Place Order** | POST | `/api/v1/orders` | Submit new trading order |
| **Get Positions** | GET | `/api/v1/positions` | Retrieve current positions |
| **Order History** | GET | `/api/v1/orders/history` | View historical orders |
| **Account Info** | GET | `/api/v1/account` | Account details and balance |

#### **Place Order Example**

**Endpoint**: `POST /api/v1/orders`

```json
{
  "symbol": "EURUSD",
  "side": "buy",
  "quantity": 100000,
  "order_type": "limit",
  "price": 1.1050,
  "time_in_force": "GTC"
}
```

</div>

---

## Response Format

All **Stream Systems API responses** follow this **standard format**:

<div class="content-section" markdown="1">

```json
{
  "status": "success",
  "data": {
    // Response data here
  },
  "timestamp": "2024-01-15T10:30:00Z",
  "request_id": "req_12345"
}
```

**Response Fields:**
- **status**: Request status (`success`, `error`)
- **data**: Response payload containing requested information
- **timestamp**: Server timestamp in ISO 8601 format
- **request_id**: Unique identifier for request tracking

</div>

---

## Error Handling

{: .note }
**Stream Systems APIs** use **standard HTTP status codes**. Always check the **response status** before processing data.

### Common Error Codes

<div class="content-section" markdown="1">

| Status Code | Error Type | Description | Action Required |
|-------------|------------|-------------|-----------------|
| **400** | Bad Request | Invalid parameters | Check request format |
| **401** | Unauthorized | Invalid or expired token | Refresh authentication |
| **403** | Forbidden | Insufficient permissions | Contact account manager |
| **429** | Rate Limited | Too many requests | Implement rate limiting |
| **500** | Server Error | Internal system error | Contact support team |

**Error Response Format:**
```json
{
  "status": "error",
  "error": {
    "code": "INVALID_SYMBOL",
    "message": "The specified symbol is not supported",
    "details": "Supported symbols: EURUSD, GBPUSD, USDJPY"
  },
  "timestamp": "2024-01-15T10:30:00Z",
  "request_id": "req_12345"
}
```

</div>

---

## Rate Limits

{: .financial-data }
**API rate limits** are applied **per account** to ensure **system stability** and **fair usage**.

<div class="content-section" markdown="1">

### Rate Limit Tiers

| API Category | Requests per Minute | Burst Limit |
|--------------|-------------------|-------------|
| **Market Data** | 1,000 requests/min | 100 requests/10sec |
| **Trading Operations** | 100 requests/min | 20 requests/10sec |
| **Account Queries** | 500 requests/min | 50 requests/10sec |

### Rate Limit Headers

All responses include rate limit information:
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 995
X-RateLimit-Reset: 1642248600
```

**Best Practices:**
- **Monitor rate limit headers** in responses
- **Implement exponential backoff** for rate limit errors
- **Cache responses** where appropriate to reduce API calls
- **Use webhooks** for real-time data instead of polling

</div>

---

## SDK and Libraries

<div class="content-section" markdown="1">

### Official SDKs

**Stream Systems** provides **official SDKs** for popular programming languages:

- **Python**: `pip install stream-systems-api`
- **JavaScript/Node.js**: `npm install @stream-systems/api`
- **Java**: Available via Maven Central
- **C#/.NET**: Available via NuGet

### Code Examples

**Python Example:**
```python
from stream_systems import StreamAPI

client = StreamAPI(api_key="your_key", secret="your_secret")
market_data = client.get_market_data("EURUSD", "1M")
print(market_data)
```

**JavaScript Example:**
```javascript
const StreamAPI = require('@stream-systems/api');

const client = new StreamAPI({
  apiKey: 'your_key',
  secret: 'your_secret'
});

const marketData = await client.getMarketData('EURUSD', '1M');
console.log(marketData);
```

</div>

---

## Support & Resources

<div class="content-section" markdown="1">

### Getting Help

{: .highlight-box }
**Need assistance?** Our **technical support team** is available **24/5** during market hours to help with API integration and troubleshooting.

### Quick Links

- 📞 **Technical Support**: Available during market hours
- 📚 **API Documentation**: Complete endpoint reference
- 💻 **Code Examples**: Sample implementations and best practices  
- 🔧 **Testing Environment**: Sandbox API for development
- 📧 **Developer Community**: Join our developer forum

### Testing Your Integration

1. **Start with Sandbox**: Use test credentials in sandbox environment
2. **Test Authentication**: Verify your API tokens work correctly
3. **Market Data**: Test market data retrieval with known symbols
4. **Paper Trading**: Execute test trades in sandbox mode
5. **Production**: Deploy to production with live credentials

</div>

---

*Need help getting started? Contact our **technical support team** during market hours or visit our **developer portal** for additional resources.*

---

**Last Updated**: July 17, 2025  
**Next Review**: January 17, 2026  
**API Version**: 2024-01  
**Support**: Available 24/5 during market hours
