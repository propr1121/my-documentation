---
title: CalServer - Calendar Services
layout: default
parent: Platform Components
nav_order: 7
last_modified_date: 2025-07-17
---

{% include freshness_indicator.html %}

# CalServer - Financial Calendar Services

{: .fs-3 }
Centralized financial calendar and date calculation services for FX options trading and settlement
{: .fs-6 .fw-300 }

---

## What It Does

CalServer provides **essential financial calendar services** for the Stream Systems platform, managing holiday calendars, business day conventions, and settlement date calculations across multiple currency pairs and market types. This microservice ensures accurate trade dating and settlement coordination.

### Core Business Functions

<div class="content-section" markdown="1">

#### 📅 **Financial Calendar Management**
- **Holiday Calendars**: Comprehensive holiday calendars for major financial centers
- **Business Day Conventions**: Following, Modified Following, Preceding day rules
- **Currency-Specific**: Separate calendars for each currency pair's settlement centers
- **Market Types**: Different calendar types for spot, forward, and options markets

#### 🗓️ **Date Calculation Services**
- **Settlement Dates**: T+2 settlement date calculations for FX trades
- **Business Day Adjustments**: Automatic adjustment for weekends and holidays
- **Tenor Calculations**: Forward date calculations (1W, 1M, 3M, 6M, 1Y)
- **Maturity Dates**: Option expiry and delivery date calculations

#### 🌍 **Multi-Market Support**
- **Global Coverage**: Major financial centers (NYC, London, Tokyo, Frankfurt)
- **Cross-Currency**: Calendar intersections for currency pair settlements
- **Real-Time Updates**: Holiday calendar updates and corrections
- **API Services**: RESTful endpoints for date calculation requests

#### 🔄 **Integration Services**
- **MLOEngine**: Settlement date calculations for post-trade processing
- **PrePricer**: Business day adjustments for pricing calculations
- **TRTN AppServer**: Date validation for external trade reporting

</div>

---

## Current Status & Health

### ✅ **Production Strengths**

{: .highlight-box }
**Comprehensive Business Logic**: CalServer successfully provides **accurate financial calendar services** with well-structured **domain modeling** and **DDD principles**.

**Key Capabilities:**
- **Accurate Calculations**: Reliable business day and settlement date calculations
- **Domain Modeling**: Well-structured calendar and date calculation logic
- **Multi-Currency Support**: Comprehensive coverage of major currency pairs
- **Clean Architecture**: Proper layered architecture with separation of concerns

### 🔴 **Critical Issues Requiring Attention**

{: .financial-data }
**Security Risk**: **CRITICAL** - **Hardcoded database credentials** and **missing authentication** expose the system to **unauthorized access**.

| Risk Category | Impact | Status | Resolution Timeline |
|---------------|--------|--------|-------------------|
| **Hardcoded DB Credentials** | Critical | Active | 1 week |
| **Missing Authentication** | Critical | Active | 2 weeks |
| **Outdated Spring Boot** | High | Active | 4-6 weeks |
| **N+1 Query Problems** | High | Active | 2-3 weeks |

### Required Improvements

**🔴 MUST DO (4-6 weeks | $75K-125K):**
- **Security Hardening**: Remove hardcoded credentials, implement authentication
- **Technology Stack Update**: Upgrade Spring Boot 2.1.6 to 3.3.x, Java 8 to Java 21
- **Performance Optimization**: Fix N+1 queries with batch loading
- **Error Handling**: Replace generic exceptions with specific types

**🟡 SHOULD DO (6-10 weeks | $100K-150K):**
- **Testing Implementation**: Add comprehensive unit and integration tests
- **Production Deployment**: Docker containerization and CI/CD pipeline
- **Monitoring**: Application health checks and performance metrics

*For detailed technical analysis, see [Technical Risk Assessment](../technical-risk-assessment.html)*

---

## Performance Characteristics

| Metric | Current | Target | Improvement Needed |
|--------|---------|--------|-------------------|
| **Date Calculation** | <100ms | <50ms | Query optimization |
| **Calendar Loading** | Variable | <1 second | Batch loading |
| **Availability** | 99% | 99.9% | Security hardening |
| **Test Coverage** | <20% | >80% | Testing implementation |

---

**Component Owner**: Infrastructure Team  
**Technology Stack**: Java 8, Spring Boot 2.1.6, PostgreSQL, Gradle  
**Status**: 🔴 Critical security updates required 