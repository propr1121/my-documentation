---
title: Troubleshooting Guide
layout: default
nav_order: 12
last_modified_date: 2025-07-17
---

{% include freshness_indicator.html %}

# Stream Systems Troubleshooting Guide

{: .fs-3 }
Comprehensive troubleshooting documentation for Stream Systems financial technology platform, covering common issues, diagnostic procedures, and resolution steps for inter-dealer brokerage services.

---

## 🚨 Emergency Response

### Critical Issue Escalation

#### Severity Classification
1. **Critical (P0)**: Trading system down, data breach, regulatory compliance failure
2. **High (P1)**: Significant functionality impaired, performance degradation
3. **Medium (P2)**: Minor functionality issues, workaround available
4. **Low (P3)**: Documentation issues, enhancement requests

#### Emergency Contacts
- **24/7 Support Hotline**: +1 (212) 555-HELP (4357)
- **Critical Issue Email**: critical@streamsystems.io
- **Escalation Manager**: John Smith - +1 (212) 555-0101
- **Technical Lead**: Sarah Johnson - +1 (212) 555-0102

### Immediate Response Checklist

#### For Critical Issues (P0)
1. **Contact 24/7 support** immediately
2. **Document symptoms** with screenshots and error messages
3. **Check system status** at [status.streamsystems.io](https://status.streamsystems.io)
4. **Implement emergency procedures** if available
5. **Notify stakeholders** per communication plan

---

## 🖥️ Platform Access Issues

### Login & Authentication Problems

#### Issue: Cannot Login to Platform
**Symptoms:**
- Login page not loading
- Invalid credentials error
- Timeout during authentication

**Diagnostic Steps:**
```bash
# Check platform availability
curl -I https://platform.streamsystems.io

# Verify DNS resolution
nslookup platform.streamsystems.io

# Test network connectivity
ping platform.streamsystems.io
```

**Resolution:**
1. **Clear browser cache** and cookies
2. **Try incognito mode** to eliminate browser extensions
3. **Verify credentials** with IT administrator
4. **Check two-factor authentication** device time sync
5. **Contact support** if issues persist

#### Issue: Two-Factor Authentication Failure
**Symptoms:**
- TOTP codes rejected
- SMS codes not received
- Hardware token not working

**Resolution:**
1. **Sync device time** - Ensure accurate time on mobile device
2. **Use backup codes** provided during setup
3. **Try alternative method** (SMS if app fails, vice versa)
4. **Contact administrator** for MFA reset

### Session Management Issues

#### Issue: Frequent Session Timeouts
**Symptoms:**
- Logged out every few minutes
- Session expires during active use
- Multiple login prompts

**Diagnostic Steps:**
```javascript
// Check session storage in browser console
console.log(sessionStorage);
console.log(localStorage);

// Check cookies
document.cookie;
```

**Resolution:**
1. **Check system clock** - Ensure correct time/timezone
2. **Disable browser auto-refresh** extensions
3. **Close duplicate tabs** - Only one session per user
4. **Clear browser data** and restart browser
5. **Update browser** to latest version

---

## 💹 Trading System Issues

### Trade Execution Problems

#### Issue: Trades Failing to Execute
**Symptoms:**
- "Trade execution failed" error
- Orders stuck in pending status
- Timeout errors during execution

**Diagnostic Queries:**
```sql
-- Check recent failed trades
SELECT trade_id, client_id, status, error_message, created_at
FROM trades 
WHERE status = 'FAILED' 
  AND created_at > NOW() - INTERVAL '1 hour'
ORDER BY created_at DESC;

-- Check system capacity
SELECT COUNT(*) as pending_trades
FROM trades 
WHERE status = 'PENDING';
```

**Common Causes & Solutions:**

| Cause | Solution |
|-------|----------|
| **Insufficient client credit** | Contact operations to verify credit limits |
| **Market closed** | Check trading hours for instrument |
| **Price movement** | Update limit price to current market |
| **System overload** | Wait and retry, escalate if persistent |
| **Invalid instrument** | Verify instrument symbol and availability |

#### Issue: Delayed Trade Confirmations
**Symptoms:**
- Trades execute but confirmations delayed
- Missing trade notifications
- Inconsistent trade blotter updates

**Diagnostic Steps:**
```bash
# Check message queue status
kubectl exec -it kafka-0 -- kafka-topics.sh --list --bootstrap-server localhost:9092

# Check Redis connection
redis-cli -h redis-service ping

# Verify database connections
kubectl logs deployment/stream-systems-app | grep "database"
```

**Resolution:**
1. **Check message queue** health and backlog
2. **Verify database connectivity** and performance
3. **Restart notification service** if isolated issue
4. **Check network latency** between components

### Position Management Issues

#### Issue: Position Discrepancies
**Symptoms:**
- Position balances don't match expectations
- Missing positions in portfolio view
- Incorrect P&L calculations

**Diagnostic Procedure:**
```sql
-- Reconcile positions vs trades
SELECT 
  p.client_id,
  p.instrument,
  p.quantity as position_qty,
  SUM(CASE WHEN t.side = 'BUY' THEN t.quantity ELSE -t.quantity END) as trade_qty
FROM positions p
LEFT JOIN trades t ON p.client_id = t.client_id AND p.instrument = t.instrument
WHERE t.status = 'EXECUTED'
GROUP BY p.client_id, p.instrument, p.quantity
HAVING p.quantity != SUM(CASE WHEN t.side = 'BUY' THEN t.quantity ELSE -t.quantity END);
```

**Resolution Steps:**
1. **Run position reconciliation** process
2. **Check for pending settlements** 
3. **Verify trade data integrity**
4. **Contact operations** for manual reconciliation if needed

### Market Data Issues

#### Issue: Stale or Missing Prices
**Symptoms:**
- Prices not updating
- "No price available" errors
- Significant spread anomalies

**Diagnostic Commands:**
```bash
# Check market data feed
kubectl logs deployment/market-data-service -f

# Verify external data sources
curl -H "Authorization: Bearer $API_KEY" \
  https://api.marketdata.com/v1/quotes/EURUSD

# Check Redis cache
redis-cli -h redis-service get "price:EURUSD"
```

**Resolution:**
1. **Check external data provider** status
2. **Verify API credentials** and rate limits
3. **Restart market data service** if cache issues
4. **Switch to backup data provider** if primary fails

---

## 🗄️ Database Issues

### Connection Problems

#### Issue: Database Connection Failures
**Symptoms:**
- "Connection timeout" errors
- "Unable to acquire connection" messages
- Application startup failures

**Diagnostic Steps:**
```bash
# Check database pod status
kubectl get pods -l app=postgresql

# Check connection pool metrics
curl http://localhost:8080/actuator/metrics/hikaricp.connections.active

# Test direct database connection
psql -h postgres-service -U stream_user -d streamsystems -c "SELECT 1;"
```

**Resolution:**
1. **Check database pod health** and resources
2. **Verify connection pool configuration**
3. **Check network connectivity** between app and database
4. **Scale database resources** if CPU/memory constrained

### Performance Issues

#### Issue: Slow Database Queries
**Symptoms:**
- Application response delays
- Database CPU high utilization
- Query timeout errors

**Diagnostic Queries:**
```sql
-- Check slow queries
SELECT query, mean_exec_time, calls
FROM pg_stat_statements 
ORDER BY mean_exec_time DESC 
LIMIT 10;

-- Check database locks
SELECT 
  pl.pid,
  pl.mode,
  pl.locktype,
  pl.relation::regclass,
  now() - pa.query_start as duration
FROM pg_locks pl
JOIN pg_stat_activity pa ON pl.pid = pa.pid
WHERE NOT pl.granted;

-- Check index usage
SELECT 
  schemaname, 
  tablename, 
  idx_scan, 
  seq_scan,
  seq_scan::float / GREATEST(idx_scan, 1) as seq_idx_ratio
FROM pg_stat_user_tables
ORDER BY seq_idx_ratio DESC;
```

**Optimization Steps:**
1. **Add missing indexes** for frequently queried columns
2. **Update table statistics** with ANALYZE
3. **Optimize connection pool** settings
4. **Consider query rewriting** for complex operations

---

## 🚀 Performance Issues

### High CPU Usage

#### Issue: Application Server High CPU
**Symptoms:**
- Slow response times
- High CPU utilization (>80%)
- Application timeouts

**Diagnostic Commands:**
```bash
# Check pod resource usage
kubectl top pods

# Get CPU profiling data
curl http://localhost:8080/actuator/metrics/process.cpu.usage

# Java thread dump for analysis
kubectl exec -it pod-name -- jstack 1 > threaddump.txt

# Check garbage collection metrics
curl http://localhost:8080/actuator/metrics/jvm.gc.pause
```

**Resolution:**
1. **Scale horizontally** - Add more application pods
2. **Optimize JVM settings** - Tune heap size and GC parameters
3. **Review application logs** for inefficient operations
4. **Profile application** to identify CPU hotspots

### Memory Issues

#### Issue: Out of Memory Errors
**Symptoms:**
- "OutOfMemoryError" in logs
- Pod restarts due to memory limits
- Application becomes unresponsive

**Memory Analysis:**
```bash
# Check memory usage
kubectl top pods

# Java heap dump analysis
kubectl exec -it pod-name -- jcmd 1 GC.run_finalization
kubectl exec -it pod-name -- jcmd 1 VM.memory_summary

# Container memory statistics
docker stats container-name
```

**Resolution:**
1. **Increase memory limits** in Kubernetes deployment
2. **Tune JVM heap size** - Adjust -Xmx and -Xms parameters
3. **Check for memory leaks** in application code
4. **Optimize caching strategy** to reduce memory usage

### Network Issues

#### Issue: Network Connectivity Problems
**Symptoms:**
- Intermittent connection failures
- High network latency
- Service discovery issues

**Network Diagnostics:**
```bash
# Check pod-to-pod connectivity
kubectl exec -it app-pod -- curl http://database-service:5432

# DNS resolution testing
kubectl exec -it app-pod -- nslookup database-service

# Network policy verification
kubectl get networkpolicies

# Service endpoint validation
kubectl get endpoints
```

**Resolution:**
1. **Verify service definitions** and selectors
2. **Check network policies** for blocking rules
3. **Test DNS resolution** within cluster
4. **Monitor network metrics** for bottlenecks

---

## 📊 Monitoring & Alerting Issues

### Missing Metrics

#### Issue: Metrics Not Appearing in Dashboards
**Symptoms:**
- Empty Grafana dashboards
- Missing Prometheus targets
- No alerting notifications

**Diagnostic Steps:**
```bash
# Check Prometheus targets
curl http://prometheus:9090/api/v1/targets

# Verify metrics endpoint
curl http://stream-systems-service/actuator/prometheus

# Check Grafana data sources
curl -u admin:password http://grafana:3000/api/datasources
```

**Resolution:**
1. **Verify Prometheus configuration** and scrape configs
2. **Check application metrics** endpoint accessibility
3. **Restart Prometheus** to reload configuration
4. **Validate Grafana data source** connections

### Alerting Issues

#### Issue: Missing or Delayed Alerts
**Symptoms:**
- No notifications for known issues
- Delayed alert delivery
- False positive alerts

**Alert Diagnostics:**
```bash
# Check alert rules
curl http://prometheus:9090/api/v1/rules

# Verify alertmanager status
curl http://alertmanager:9093/api/v1/status

# Check notification channels
kubectl logs deployment/alertmanager
```

**Resolution:**
1. **Review alert rule syntax** and conditions
2. **Test notification channels** (email, Slack, etc.)
3. **Adjust alert thresholds** to reduce false positives
4. **Check alertmanager routing** configuration

---

## 🔧 Configuration Issues

### Environment Configuration

#### Issue: Incorrect Environment Settings
**Symptoms:**
- Application startup failures
- Invalid service connections
- Missing feature functionality

**Configuration Validation:**
```bash
# Check environment variables
kubectl exec -it pod-name -- env | grep STREAM

# Verify ConfigMap contents
kubectl get configmap stream-config -o yaml

# Check secret values (be careful with sensitive data)
kubectl get secret stream-secrets -o yaml
```

**Resolution:**
1. **Validate configuration syntax** (YAML, JSON)
2. **Check environment-specific** values
3. **Verify secret references** and permissions
4. **Restart pods** after configuration changes

### Service Discovery Issues

#### Issue: Services Cannot Find Each Other
**Symptoms:**
- "Service not found" errors
- Connection refused to internal services
- Inconsistent service availability

**Service Diagnostics:**
```bash
# List all services
kubectl get services

# Check service endpoints
kubectl describe service stream-systems-service

# Verify pod labels and selectors
kubectl get pods --show-labels

# Test service DNS
kubectl exec -it pod-name -- nslookup stream-systems-service
```

**Resolution:**
1. **Verify service selectors** match pod labels
2. **Check service ports** and target ports
3. **Ensure pods are ready** and healthy
4. **Review namespace** configurations

---

## 📝 Common Error Messages

### Application Errors

#### "Connection pool exhausted"
**Cause:** Too many concurrent database connections
**Solution:**
```yaml
# Increase connection pool size
spring:
  datasource:
    hikari:
      maximum-pool-size: 50
      minimum-idle: 10
```

#### "Rate limit exceeded"
**Cause:** Too many API requests to external services
**Solution:**
```java
// Implement exponential backoff
@Retryable(value = {RateLimitException.class}, 
           backoff = @Backoff(delay = 1000, multiplier = 2))
```

#### "Circuit breaker open"
**Cause:** Downstream service failures triggering circuit breaker
**Solution:**
1. Check downstream service health
2. Review circuit breaker thresholds
3. Implement proper fallback mechanisms

### Infrastructure Errors

#### "Pod in CrashLoopBackOff"
**Cause:** Application failing to start repeatedly
**Diagnostic:**
```bash
kubectl logs pod-name --previous
kubectl describe pod pod-name
```
**Solution:**
1. Check application logs for startup errors
2. Verify resource limits and requests
3. Review health check configurations

#### "ImagePullBackOff"
**Cause:** Cannot pull Docker image from registry
**Solution:**
1. Verify image name and tag
2. Check registry credentials
3. Ensure network access to registry

---

## 🛠️ Diagnostic Tools

### Log Analysis Tools

#### Centralized Logging Queries
```bash
# Elasticsearch queries for error investigation
curl -X GET "elasticsearch:9200/stream-systems-*/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        {"range": {"@timestamp": {"gte": "now-1h"}}},
        {"match": {"level": "ERROR"}}
      ]
    }
  },
  "sort": [{"@timestamp": {"order": "desc"}}]
}'

# Kibana dashboard for real-time monitoring
# Navigate to: http://kibana:5601/app/discover
```

### Performance Profiling

#### Java Application Profiling
```bash
# Generate heap dump
kubectl exec -it pod-name -- jcmd 1 GC.dump_heap /tmp/heapdump.hprof

# CPU profiling with async-profiler
kubectl exec -it pod-name -- java -jar async-profiler.jar -d 60 -f profile.html 1
```

### Network Troubleshooting

#### Network Connectivity Testing
```bash
# Port connectivity test
nc -zv hostname port

# Network trace
traceroute hostname

# Packet capture (if permitted)
tcpdump -i eth0 host hostname
```

---

## 📞 Support Procedures

### Escalation Matrix

| Issue Severity | Response Time | Resolution Time | Escalation |
|---------------|---------------|-----------------|------------|
| **Critical** | 15 minutes | 4 hours | CTO |
| **High** | 1 hour | 24 hours | Engineering Manager |
| **Medium** | 4 hours | 72 hours | Team Lead |
| **Low** | 24 hours | 1 week | Assigned Engineer |

### Information to Provide

#### When Contacting Support
1. **Issue Description** - Clear problem statement
2. **Steps to Reproduce** - Exact steps that cause the issue
3. **Error Messages** - Complete error text and stack traces
4. **System Information** - Environment, versions, configuration
5. **Impact Assessment** - Business impact and affected users
6. **Urgency Level** - Based on business criticality

#### Log Collection Commands
```bash
# Application logs
kubectl logs deployment/stream-systems-app --tail=1000 > app-logs.txt

# System metrics
kubectl top nodes > system-metrics.txt
kubectl top pods >> system-metrics.txt

# Configuration dump
kubectl get configmaps -o yaml > configmaps.yaml
kubectl get secrets -o yaml > secrets.yaml  # Be careful with sensitive data
```

---

*Troubleshooting Guide last updated: {{ page.last_modified_date | date: "%B %d, %Y" }}*  
*Platform Version: v2.1.0*  
*Next scheduled review: {{ page.last_modified_date | date: "%s" | plus: 2592000 | date: "%B %d, %Y" }}* 