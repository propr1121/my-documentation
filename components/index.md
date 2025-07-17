---
title: Platform Components
layout: default
nav_order: 5
has_children: true
last_modified_date: 2025-07-17
---

{% include freshness_indicator.html %}

# Platform Components

{: .fs-3 }
Detailed documentation for each component of the Stream Systems trading platform
{: .fs-6 .fw-300 }

---

## Component Overview

The Stream Systems platform consists of **8 integrated components** that work together to deliver comprehensive inter-dealer brokerage services for FX options trading. Each component is designed for specific functionality while maintaining seamless integration with the overall system.

### Core Trading Components

- **[FXOptEngine](fxoptengine.html)** - Central order matching and execution engine
- **[MLOEngine](mloengine.html)** - Post-trade processing and settlement management
- **[PrePricer](prepricer.html)** - Real-time FX options pricing engine
- **[VolPricer](volpricer.html)** - Advanced volatility surface construction

### User Interface & Data

- **[FXOUI](fxoui.html)** - Modern web-based trading interface
- **[MktDataBridge](mktdatabridge.html)** - Market data acquisition and distribution
- **[CalServer](calserver.html)** - Financial calendar and business day services

### External Integration

- **[TRTN AppServer](trtn-appserver.html)** - Thomson Reuters Trading Network integration

---

## Component Health Summary

### ✅ **Production Ready**
- **FXOptEngine**: Core functionality operational with modernization needed
- **MLOEngine**: Post-trade processing working with compliance gaps
- **PrePricer**: Pricing accuracy maintained with security vulnerabilities

### ⚠️ **Requires Updates**
- **FXOUI**: Security patches and performance optimization needed
- **MktDataBridge**: Failover mechanisms and monitoring improvements
- **CalServer**: Technology stack upgrade and security hardening

### 🔧 **Development Focus**
- **VolPricer**: Performance optimization and caching implementation
- **TRTN AppServer**: Critical security fixes and operational readiness

---

## Quick Access

### By Business Function
- **Trading Operations**: FXOptEngine, FXOUI, PrePricer
- **Post-Trade Processing**: MLOEngine, TRTN AppServer, CalServer
- **Market Data & Analytics**: MktDataBridge, VolPricer, PrePricer
- **User Experience**: FXOUI, trading workspaces, reporting tools

### By Technology Stack
- **C++ Components**: FXOptEngine, MLOEngine, PrePricer
- **Java/Spring Boot**: VolPricer, MktDataBridge, CalServer, TRTN AppServer
- **Frontend**: FXOUI (React/TypeScript)

### By Priority Level
- **Critical Security Fixes**: All components require immediate security attention
- **Performance Optimization**: FXOptEngine, PrePricer, VolPricer
- **Operational Readiness**: MLOEngine, TRTN AppServer, MktDataBridge

---

*Each component page provides detailed technical analysis, current status, required improvements, and implementation roadmaps. For platform-wide technical risks, see [Technical Risk Assessment](../technical-risk-assessment.html).* 