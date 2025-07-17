---
title: Maintenance Guide
layout: default
nav_order: 13
last_modified_date: 2025-07-17
---

{% include freshness_indicator.html %}

# Stream Systems Maintenance Guide

{: .fs-3 }
Comprehensive maintenance documentation for Stream Systems financial technology platform, covering preventive maintenance, system updates, and operational procedures for inter-dealer brokerage services.

---

## 🕒 Maintenance Schedule

### Regular Maintenance Windows

#### Weekly Maintenance
- **Time**: Sundays 2:00 AM - 4:00 AM EST
- **Duration**: Maximum 2 hours
- **Scope**: Minor updates, security patches, routine optimization
- **Impact**: Minimal service disruption

#### Monthly Maintenance
- **Time**: First Saturday 11:00 PM - 3:00 AM EST
- **Duration**: Maximum 4 hours
- **Scope**: Major updates, infrastructure changes, performance tuning
- **Impact**: Planned service downtime

#### Quarterly Maintenance
- **Time**: Scheduled 2 weeks in advance
- **Duration**: Maximum 8 hours
- **Scope**: Major platform upgrades, disaster recovery testing
- **Impact**: Extended service downtime with advance notice

### Emergency Maintenance
- **Timing**: As needed for critical security or stability issues
- **Authorization**: CTO approval required
- **Notification**: Minimum 2-hour advance notice when possible
- **Documentation**: Post-maintenance report required

---

## 🔧 Preventive Maintenance

### System Health Checks

#### Daily Automated Checks
```bash
#!/bin/bash
# daily-health-check.sh

echo "=== Daily System Health Check - $(date) ==="

# Check service availability
kubectl get pods --all-namespaces | grep -v Running | grep -v Completed

# Database health
psql -h postgres-service -U stream_user -d streamsystems -c "SELECT version();"

# Redis connectivity
redis-cli -h redis-service ping

# Disk space monitoring
df -h | awk '$5 > 80 {print "WARNING: " $0}'

# Memory utilization
free -m | awk 'NR==2{printf "Memory Usage: %s/%sMB (%.2f%%)\n", $3,$2,$3*100/$2}'

# Check certificate expiration
kubectl get secrets -o jsonpath='{.items[*].metadata.name}' | xargs -I {} kubectl get secret {} -o jsonpath='{.data.tls\.crt}' | base64 -d | openssl x509 -noout -dates

echo "=== Health Check Complete ==="
```

#### Weekly Performance Review
```sql
-- Weekly database performance analysis
SELECT 
  schemaname,
  tablename,
  n_tup_ins as inserts,
  n_tup_upd as updates,
  n_tup_del as deletes,
  n_live_tup as live_tuples,
  n_dead_tup as dead_tuples,
  last_vacuum,
  last_autovacuum,
  last_analyze
FROM pg_stat_user_tables
WHERE schemaname = 'public'
ORDER BY n_dead_tup DESC;

-- Check for long-running queries
SELECT 
  pid,
  now() - pg_stat_activity.query_start AS duration,
  query,
  state
FROM pg_stat_activity
WHERE (now() - pg_stat_activity.query_start) > interval '5 minutes'
  AND state = 'active';
```

### Resource Monitoring

#### Disk Space Management
```bash
# Automated log rotation
/usr/sbin/logrotate -f /etc/logrotate.d/streamsystems

# Clean up old Docker images
docker system prune -f --volumes

# Database vacuum and analyze
psql -h postgres-service -U stream_user -d streamsystems -c "VACUUM ANALYZE;"

# Clear Redis cache if memory usage > 80%
redis-cli -h redis-service --eval "
  local memory = redis.call('INFO', 'memory')
  local used = tonumber(memory:match('used_memory:(%d+)'))
  local max = tonumber(memory:match('maxmemory:(%d+)'))
  if used/max > 0.8 then
    redis.call('FLUSHDB')
    return 'Cache cleared due to high memory usage'
  end
  return 'Memory usage OK'
"
```

#### Network Performance Monitoring
```bash
# Monitor network latency between services
kubectl exec -it app-pod -- ping -c 5 database-service
kubectl exec -it app-pod -- ping -c 5 redis-service

# Check service mesh metrics (if using Istio)
kubectl exec -it istio-proxy -- curl localhost:15000/stats/prometheus | grep "outbound.*response_time"

# Monitor external API latency
curl -w "@curl-format.txt" -o /dev/null -s "https://api.marketdata.com/health"
```

---

## 📦 Software Updates

### Application Updates

#### Deployment Process
```bash
#!/bin/bash
# update-application.sh

set -e

NEW_VERSION=$1
ENVIRONMENT=${2:-staging}

if [ -z "$NEW_VERSION" ]; then
  echo "Usage: $0 <version> [environment]"
  exit 1
fi

echo "Deploying Stream Systems v$NEW_VERSION to $ENVIRONMENT"

# Pre-deployment checks
echo "Running pre-deployment validation..."
kubectl get pods -n $ENVIRONMENT | grep stream-systems

# Database migration (if needed)
if [ -f "migrations/v$NEW_VERSION.sql" ]; then
  echo "Running database migrations..."
  psql -h postgres-service -U stream_user -d streamsystems -f migrations/v$NEW_VERSION.sql
fi

# Blue-green deployment
echo "Starting blue-green deployment..."
./scripts/deploy-blue-green.sh $NEW_VERSION $ENVIRONMENT

# Post-deployment validation
echo "Running post-deployment tests..."
./scripts/smoke-tests.sh $ENVIRONMENT

# Update monitoring
echo "Updating monitoring dashboards..."
./scripts/update-dashboards.sh $NEW_VERSION

echo "Deployment complete!"
```

#### Rollback Procedures
```bash
#!/bin/bash
# rollback-application.sh

ENVIRONMENT=${1:-production}
PREVIOUS_VERSION=$(kubectl get deployment stream-systems-app -n $ENVIRONMENT -o jsonpath='{.metadata.annotations.previous-version}')

echo "Rolling back to version $PREVIOUS_VERSION in $ENVIRONMENT"

# Rollback deployment
kubectl rollout undo deployment/stream-systems-app -n $ENVIRONMENT

# Verify rollback
kubectl rollout status deployment/stream-systems-app -n $ENVIRONMENT

# Database rollback (if needed)
if [ -f "rollbacks/from-v$CURRENT_VERSION.sql" ]; then
  echo "Rolling back database changes..."
  psql -h postgres-service -U stream_user -d streamsystems -f rollbacks/from-v$CURRENT_VERSION.sql
fi

echo "Rollback complete to version $PREVIOUS_VERSION"
```

### Security Updates

#### Patch Management Process
1. **Security Assessment**: Evaluate criticality and impact
2. **Testing**: Apply patches in testing environment
3. **Staging Validation**: Verify functionality in staging
4. **Production Deployment**: Deploy during maintenance window
5. **Monitoring**: Enhanced monitoring post-deployment

#### Critical Security Patches
```bash
# Emergency security patch deployment
kubectl set image deployment/stream-systems-app app=streamsystems/platform:v2.1.1-security-patch -n production

# Monitor deployment
kubectl rollout status deployment/stream-systems-app -n production --timeout=600s

# Immediate security scan
trivy image streamsystems/platform:v2.1.1-security-patch

# Verify security controls
./scripts/security-validation.sh
```

---

## 🗄️ Database Maintenance

### Regular Database Tasks

#### Daily Operations
```sql
-- Update table statistics
ANALYZE;

-- Check for bloated tables
SELECT 
  schemaname,
  tablename,
  n_dead_tup,
  n_live_tup,
  ROUND((n_dead_tup::float / GREATEST(n_live_tup, 1)) * 100, 2) as bloat_percent
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY bloat_percent DESC;

-- Monitor replication lag (if applicable)
SELECT 
  client_addr,
  state,
  pg_wal_lsn_diff(pg_current_wal_lsn(), flush_lsn) as lag_bytes
FROM pg_stat_replication;
```

#### Weekly Maintenance
```bash
# Vacuum full on heavily updated tables (during maintenance window)
psql -h postgres-service -U stream_user -d streamsystems << EOF
VACUUM FULL trades;
VACUUM FULL positions;
VACUUM FULL market_data;
ANALYZE;
EOF

# Reindex if needed
psql -h postgres-service -U stream_user -d streamsystems -c "REINDEX SCHEMA public;"

# Update sequence values
psql -h postgres-service -U stream_user -d streamsystems << EOF
SELECT setval('trades_id_seq', (SELECT MAX(id) FROM trades));
SELECT setval('positions_id_seq', (SELECT MAX(id) FROM positions));
EOF
```

### Backup Procedures

#### Automated Backup Script
```bash
#!/bin/bash
# backup-database.sh

BACKUP_DIR="/backup/$(date +%Y%m%d)"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

# Full database backup
pg_dump -h postgres-service -U stream_user -d streamsystems | gzip > $BACKUP_DIR/streamsystems_full_$TIMESTAMP.sql.gz

# Schema-only backup
pg_dump -h postgres-service -U stream_user -d streamsystems --schema-only > $BACKUP_DIR/streamsystems_schema_$TIMESTAMP.sql

# Data-only backup for critical tables
pg_dump -h postgres-service -U stream_user -d streamsystems --data-only --table=trades --table=positions | gzip > $BACKUP_DIR/critical_data_$TIMESTAMP.sql.gz

# Upload to cloud storage
aws s3 sync $BACKUP_DIR s3://streamsystems-backups/database/

# Cleanup old local backups (keep 7 days)
find /backup -type d -mtime +7 -exec rm -rf {} \;

echo "Database backup completed: $TIMESTAMP"
```

#### Backup Verification
```bash
# Test backup integrity
gunzip -c /backup/streamsystems_full_latest.sql.gz | head -n 100

# Restore test in isolated environment
kubectl run backup-test --image=postgres:15 --rm -it -- psql -h test-postgres -U test_user -d test_db -f /backup/streamsystems_full_latest.sql

# Verify critical data
psql -h test-postgres -U test_user -d test_db -c "SELECT COUNT(*) FROM trades WHERE created_at >= CURRENT_DATE;"
```

---

## 🔄 System Optimization

### Performance Tuning

#### Application Optimization
```yaml
# JVM tuning for better performance
apiVersion: apps/v1
kind: Deployment
metadata:
  name: stream-systems-app
spec:
  template:
    spec:
      containers:
      - name: app
        env:
        - name: JAVA_OPTS
          value: "-Xms4g -Xmx8g -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:+UseStringDeduplication"
        resources:
          requests:
            memory: "4Gi"
            cpu: "2000m"
          limits:
            memory: "8Gi"
            cpu: "4000m"
```

#### Database Optimization
```sql
-- Optimize PostgreSQL configuration
ALTER SYSTEM SET shared_buffers = '2GB';
ALTER SYSTEM SET effective_cache_size = '6GB';
ALTER SYSTEM SET work_mem = '256MB';
ALTER SYSTEM SET maintenance_work_mem = '1GB';
ALTER SYSTEM SET checkpoint_completion_target = 0.9;
ALTER SYSTEM SET wal_buffers = '64MB';
ALTER SYSTEM SET default_statistics_target = 500;

-- Reload configuration
SELECT pg_reload_conf();

-- Create performance-critical indexes
CREATE INDEX CONCURRENTLY idx_trades_created_at_client_id ON trades(created_at, client_id);
CREATE INDEX CONCURRENTLY idx_positions_client_instrument ON positions(client_id, instrument);
CREATE INDEX CONCURRENTLY idx_market_data_timestamp ON market_data(timestamp DESC);
```

### Cache Management

#### Redis Optimization
```bash
# Redis configuration optimization
redis-cli -h redis-service CONFIG SET maxmemory-policy allkeys-lru
redis-cli -h redis-service CONFIG SET maxmemory 4gb
redis-cli -h redis-service CONFIG SET save "900 1 300 10 60 10000"

# Monitor Redis performance
redis-cli -h redis-service --latency-history -i 5

# Analyze Redis memory usage
redis-cli -h redis-service --bigkeys

# Clear expired keys
redis-cli -h redis-service EVAL "return redis.call('DEL', unpack(redis.call('KEYS', ARGV[1])))" 0 "session:expired:*"
```

#### Application Cache Tuning
```java
// Optimize Spring Cache configuration
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        RedisCacheManager.Builder builder = RedisCacheManager
            .RedisCacheManagerBuilder
            .fromConnectionFactory(redisConnectionFactory())
            .cacheDefaults(cacheConfiguration());
        return builder.build();
    }
    
    private RedisCacheConfiguration cacheConfiguration() {
        return RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(30))
            .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
    }
}
```

---

## 📊 Monitoring & Alerting Maintenance

### Monitoring System Maintenance

#### Prometheus Maintenance
```bash
# Clean up old metrics data
kubectl exec -it prometheus-0 -- promtool tsdb list-blocks /prometheus
kubectl exec -it prometheus-0 -- promtool tsdb delete-series --dry-run /prometheus

# Compact Prometheus data
kubectl exec -it prometheus-0 -- promtool tsdb create-snapshot /prometheus

# Update Prometheus rules
kubectl apply -f monitoring/prometheus-rules.yaml
kubectl delete pod -l app=prometheus  # Restart to reload rules
```

#### Grafana Dashboard Updates
```bash
# Export current dashboards
for dashboard in $(curl -s -u admin:$GRAFANA_PASSWORD http://grafana:3000/api/search | jq -r '.[].uid'); do
  curl -s -u admin:$GRAFANA_PASSWORD http://grafana:3000/api/dashboards/uid/$dashboard | jq .dashboard > dashboards/$dashboard.json
done

# Import updated dashboards
for dashboard in dashboards/*.json; do
  curl -X POST -u admin:$GRAFANA_PASSWORD \
    -H "Content-Type: application/json" \
    -d @$dashboard \
    http://grafana:3000/api/dashboards/db
done
```

### Alert Rule Maintenance

#### Review and Update Alerts
```yaml
# Update alert thresholds based on historical data
groups:
- name: stream-systems.rules
  rules:
  - alert: HighCPUUsage
    expr: cpu_usage_percent > 85
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High CPU usage detected"
      
  - alert: DatabaseConnectionPoolExhausted
    expr: hikaricp_connections_active / hikaricp_connections_max > 0.9
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "Database connection pool nearly exhausted"

  - alert: TradingSystemDown
    expr: up{job="stream-systems"} == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Trading system is down"
```

---

## 🔐 Security Maintenance

### Security Hardening

#### Regular Security Tasks
```bash
# Update container base images
docker pull openjdk:17-jre-alpine
docker pull postgres:15-alpine
docker pull redis:7-alpine
docker pull nginx:1.24-alpine

# Scan for vulnerabilities
trivy image streamsystems/platform:latest
trivy k8s --report summary cluster

# Update SSL certificates
certbot renew --nginx
kubectl create secret tls stream-systems-tls --cert=new-cert.pem --key=new-key.pem --dry-run=client -o yaml | kubectl apply -f -

# Review and rotate secrets
kubectl delete secret api-keys
kubectl create secret generic api-keys --from-literal=market-data-key=$NEW_MARKET_DATA_KEY
```

#### Access Control Review
```bash
# Review user permissions
kubectl get rolebindings,clusterrolebindings -o wide

# Audit service account permissions
kubectl auth can-i --list --as=system:serviceaccount:production:stream-systems

# Review network policies
kubectl get networkpolicies -o yaml

# Check for unused secrets and configmaps
kubectl get secrets --no-headers | awk '{print $1}' | xargs -I {} kubectl describe secret {} | grep "Used By"
```

### Compliance Maintenance

#### Regular Compliance Checks
```bash
# Generate compliance report
./scripts/compliance-report.sh > reports/compliance-$(date +%Y%m%d).txt

# Check audit log integrity
kubectl logs audit-logger --since=24h | jq '.verb' | sort | uniq -c

# Verify data retention policies
psql -h postgres-service -U stream_user -d streamsystems << EOF
SELECT COUNT(*) as old_records 
FROM trades 
WHERE created_at < NOW() - INTERVAL '7 years';

SELECT COUNT(*) as old_logs 
FROM audit_logs 
WHERE created_at < NOW() - INTERVAL '10 years';
EOF

# Export regulatory reports
./scripts/generate-regulatory-reports.sh $(date +%Y%m)
```

---

## 📅 Maintenance Calendar

### Annual Maintenance Tasks

#### Q1 (January - March)
- **DR Testing**: Full disaster recovery exercise
- **Security Audit**: Annual penetration testing
- **Performance Review**: Annual capacity planning
- **License Renewal**: Software license renewals

#### Q2 (April - June)
- **Infrastructure Refresh**: Hardware/VM upgrades
- **Compliance Review**: Regulatory compliance assessment
- **Backup Testing**: Full backup and restore validation
- **Staff Training**: Technical team certification updates

#### Q3 (July - September)
- **Platform Upgrades**: Major platform version updates
- **Security Hardening**: Security configuration review
- **Monitoring Enhancement**: Monitoring system improvements
- **Process Review**: Maintenance procedure updates

#### Q4 (October - December)
- **Year-end Processing**: Special year-end maintenance
- **Budget Planning**: Next year's maintenance budget
- **Vendor Review**: Third-party vendor assessments
- **Documentation Updates**: Annual documentation review

### Maintenance Checklist Template

#### Pre-Maintenance
- [ ] Notify stakeholders (48 hours advance)
- [ ] Create maintenance ticket
- [ ] Backup critical data
- [ ] Prepare rollback plan
- [ ] Test procedures in staging
- [ ] Verify maintenance window availability

#### During Maintenance
- [ ] Follow change control process
- [ ] Document all changes made
- [ ] Monitor system metrics
- [ ] Test critical functionality
- [ ] Verify security controls
- [ ] Update configuration management

#### Post-Maintenance
- [ ] Validate all services operational
- [ ] Check monitoring alerts
- [ ] Run smoke tests
- [ ] Update documentation
- [ ] Send completion notification
- [ ] Create post-maintenance report

---

## 📞 Maintenance Support

### Emergency Procedures

#### Escalation During Maintenance
1. **Immediate**: On-duty engineer
2. **15 minutes**: Maintenance lead
3. **30 minutes**: Engineering manager
4. **1 hour**: CTO/VP Engineering

#### Emergency Contacts
- **Maintenance Lead**: Mike Chen - +1 (212) 555-0201
- **Database Admin**: Lisa Park - +1 (212) 555-0202
- **Security Officer**: Alex Rivera - +1 (212) 555-0203
- **Network Admin**: Tom Wilson - +1 (212) 555-0204

### Documentation Requirements

#### Maintenance Log Template
```
Maintenance ID: MAINT-2025-0717-001
Date: July 17, 2025
Window: 02:00 - 04:00 EST
Lead Engineer: John Doe

Planned Changes:
- Application update to v2.1.1
- Database index optimization
- SSL certificate renewal

Actual Changes:
- [List all changes made]

Issues Encountered:
- [Any problems during maintenance]

Resolution:
- [How issues were resolved]

Validation:
- [Tests performed post-maintenance]

Status: COMPLETED
Duration: 1 hour 45 minutes
```

---

*Maintenance Guide last updated: {{ page.last_modified_date | date: "%B %d, %Y" }}*  
*Platform Version: v2.1.0*  
*Next scheduled review: {{ page.last_modified_date | date: "%s" | plus: 2592000 | date: "%B %d, %Y" }}* 