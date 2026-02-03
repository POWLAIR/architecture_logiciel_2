# C4 - ITÉRATION 4 : Scalabilité et Observabilité

## Objectif IT4
Transformer le système **performant** (IT3) en système **production-grade enterprise** en ajoutant :
- ✅ Logs centralisés (ELK Stack).
- ✅ Métriques temps réel (Prometheus + Grafana).
- ✅ Alerting automatique (Slack/Email).
- ✅ Health checks (endpoints santé).
- ✅ Load balancer (scaling horizontal futur).
- ✅ Documentation API (Swagger/OpenAPI).

**Posture** : Excellence opérationnelle et **préparation expansion** (multi-restaurants).

---

## 4.1. Ajouts Architecturaux IT4

### Schéma IT4 : Architecture Finale Complète

```
┌─────────────────────────────────────────────────────────┐
│                   TIER 1 : PRÉSENTATION                 │
├─────────────────────────────────────────────────────────┤
│  Mobile Serveur x3  │  Caisse Web  │  🆕 Admin Dashboard│
└──────────┬──────────┴──────┬───────┴──────┬────────────┘
           │                 │              │              
           │   HTTPS/TLS 1.3 │              │              
           │                 │              │              
┌──────────▼─────────────────▼──────────────▼────────────┐
│  🆕 LOAD BALANCER (NGINX)                              │
│  - Round-robin distribution                            │
│  - Health checks (/health endpoint)                    │
│  - SSL termination                                     │
└──────────┬────────────────────────────────┬────────────┘
           │                                │             
┌──────────▼────────────────┐   ┌───────────▼──────────┐
│  API Backend #1           │   │  API Backend #2      │
│  (Node.js + Express)      │   │  (Node.js + Express) │
│                           │   │                      │
│  🆕 Winston Logger        │   │  🆕 Winston Logger   │
│  🆕 Prometheus Client     │   │  🆕 Prometheus Client│
│  🆕 /health endpoint      │   │  🆕 /health endpoint │
└───────┬───────────────────┘   └──────────┬───────────┘
        │                                  │            
        └──────────┬───────────────────────┘            
                   │                                     
    ┌──────────────┼──────────────────────┐             
    │              │                      │             
┌───▼────┐   ┌─────▼─────┐   ┌───────────▼──────────┐  
│ Redis  │   │PostgreSQL │   │ 🆕 ELK Stack         │  
│ Cache  │   │ Master    │   │ - Elasticsearch      │  
│        │   │           │   │ - Logstash           │  
└────────┘   └─────┬─────┘   │ - Kibana             │  
                   │         └──────────────────────┘  
            ┌──────▼─────┐                             
            │PostgreSQL  │   🆕 Monitoring Stack       
            │ Replica    │   ┌──────────────────────┐  
            └────────────┘   │ - Prometheus         │  
                             │ - Grafana            │  
                             │ - Alertmanager       │  
                             └──────────────────────┘  
```

**Légende** : 🆕 = Nouveauté IT4

---

## 4.2. Fonctionnalités Ajoutées IT4

### 4.2.1. Logs Centralisés (ELK Stack)

#### Architecture ELK

```
┌─────────────────────────────────────────────────┐
│  Backend API (Node.js)                          │
│                                                 │
│  🆕 Winston Transport → Logstash                │
│  {                                              │
│    level: 'info',                               │
│    timestamp: '2026-02-02T11:30:45Z',          │
│    message: 'Order created',                    │
│    order_id: 78,                                │
│    user_id: 12,                                 │
│    table_id: 5                                  │
│  }                                              │
└──────────────────┬──────────────────────────────┘
                   │ TCP 5000                      
                   │                               
┌──────────────────▼──────────────────────────────┐
│  Logstash (Ingestion + Parsing)                │
│                                                 │
│  filter {                                       │
│    json { source => "message" }                 │
│    grok { match => { ... } }                    │
│  }                                              │
└──────────────────┬──────────────────────────────┘
                   │                               
┌──────────────────▼──────────────────────────────┐
│  Elasticsearch (Stockage Indexé)               │
│  - Index: logs-restaurant-2026.02.02            │
│  - Retention: 90 jours                          │
│  - Full-text search                             │
└──────────────────┬──────────────────────────────┘
                   │                               
┌──────────────────▼──────────────────────────────┐
│  Kibana (Visualisation)                        │
│  - Dashboards (erreurs, latence, volumes)      │
│  - Alertes (> 10 erreurs/min)                  │
└─────────────────────────────────────────────────┘
```

#### Configuration Winston (Backend)

```javascript
const winston = require('winston');
const LogstashTransport = require('winston-logstash/lib/winston-logstash-latest');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    // Console (dev)
    new winston.transports.Console(),
    
    // 🆕 IT4 : Logstash (production)
    new LogstashTransport({
      host: 'logstash.restaurant.local',
      port: 5000,
      node_name: process.env.HOSTNAME || 'api-backend'
    })
  ]
});

// Utilisation
app.post('/api/orders', async (req, res) => {
  logger.info('Order creation started', { 
    user_id: req.user.id,
    table_id: req.body.table_id
  });
  
  try {
    const order = await OrderService.create(req.body);
    
    logger.info('Order created successfully', { 
      order_id: order.id,
      amount: order.total,
      duration_ms: performance.now()
    });
    
    res.status(201).json(order);
  } catch (err) {
    logger.error('Order creation failed', { 
      error: err.message,
      stack: err.stack,
      user_id: req.user.id
    });
    
    res.status(500).json({ error: 'Internal error' });
  }
});
```

**Exemples de Recherches Kibana** :
- `level: "error" AND timestamp:[now-1h TO now]` → Erreurs dernière heure.
- `order_id: 78` → Tracer toutes les étapes d'une commande.
- `user_id: 12 AND message: "payment"` → Historique paiements d'un serveur.

---

### 4.2.2. Métriques Temps Réel (Prometheus + Grafana)

#### Architecture Monitoring

```
┌─────────────────────────────────────────────────┐
│  Backend API (Node.js)                          │
│                                                 │
│  🆕 prom-client (Library)                       │
│  - Counter: http_requests_total                 │
│  - Histogram: http_request_duration_seconds     │
│  - Gauge: active_connections                    │
│  - Gauge: database_pool_size                    │
│                                                 │
│  GET /metrics endpoint                          │
└──────────────────┬──────────────────────────────┘
                   │ Scrape toutes les 15s         
                   │                               
┌──────────────────▼──────────────────────────────┐
│  Prometheus (Time-Series DB)                   │
│  - Collecte métriques                           │
│  - Stockage historique (30 jours)              │
│  - Requêtes PromQL                              │
└──────────────────┬──────────────────────────────┘
                   │                               
┌──────────────────▼──────────────────────────────┐
│  Grafana (Visualization)                       │
│  - Dashboard "Restaurant Overview"              │
│  - Graphs temps réel                            │
└─────────────────────────────────────────────────┘
```

#### Métriques Collectées

| Métrique | Type | Description |
| :--- | :--- | :--- |
| `http_requests_total` | Counter | Nombre total requêtes (label: method, route, status) |
| `http_request_duration_seconds` | Histogram | Latence requêtes (P50, P95, P99) |
| `orders_created_total` | Counter | Nombre commandes créées |
| `payments_total` | Counter | Encaissements (label: method = CB/Espèces) |
| `database_pool_active` | Gauge | Connexions BDD actives |
| `cache_hit_rate` | Gauge | Taux de hit Redis (%) |
| `websocket_connections` | Gauge | Connexions WebSocket actives |

#### Code Backend (Prometheus)

```javascript
const promClient = require('prom-client');

// Registre par défaut
const register = promClient.register;

// 🆕 Métriques personnalisées
const httpRequestsTotal = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status']
});

const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request latency',
  labelNames: ['method', 'route'],
  buckets: [0.01, 0.05, 0.1, 0.5, 1, 2, 5]
});

const ordersCreated = new promClient.Counter({
  name: 'orders_created_total',
  help: 'Total orders created'
});

const dbPoolActive = new promClient.Gauge({
  name: 'database_pool_active',
  help: 'Active database connections'
});

// Middleware instrumentation
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    
    httpRequestsTotal.labels(req.method, req.route?.path || 'unknown', res.statusCode).inc();
    httpRequestDuration.labels(req.method, req.route?.path || 'unknown').observe(duration);
  });
  
  next();
});

// Endpoint métriques
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

// Incrémentation métrique métier
app.post('/api/orders', async (req, res) => {
  const order = await OrderService.create(req.body);
  
  ordersCreated.inc(); // 🆕 Tracking
  
  res.status(201).json(order);
});

// Mise à jour gauge périodique
setInterval(() => {
  dbPoolActive.set(pool.totalCount - pool.idleCount);
}, 5000);
```

#### Configuration Prometheus

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'api-backend'
    static_configs:
      - targets: ['api-backend-1:3000', 'api-backend-2:3000']
        labels:
          environment: 'production'
          
  - job_name: 'postgresql'
    static_configs:
      - targets: ['postgres-exporter:9187']
      
  - job_name: 'redis'
    static_configs:
      - targets: ['redis-exporter:9121']
```

#### Dashboard Grafana (Exemple)

**Panel 1** : Requests per Second
```promql
rate(http_requests_total[1m])
```

**Panel 2** : P95 Latency
```promql
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

**Panel 3** : Error Rate
```promql
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])
```

**Panel 4** : Active Orders
```promql
orders_created_total - orders_completed_total
```

---

### 4.2.3. Alerting Automatique (Alertmanager)

#### Règles d'Alerte Prometheus

```yaml
# alerts.yml
groups:
  - name: restaurant_alerts
    interval: 30s
    rules:
      # Alerte : API down
      - alert: APIDown
        expr: up{job="api-backend"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "API Backend down"
          description: "Instance {{ $labels.instance }} is unreachable"
          
      # Alerte : Taux erreur élevé
      - alert: HighErrorRate
        expr: |
          rate(http_requests_total{status=~"5.."}[5m]) 
          / rate(http_requests_total[5m]) > 0.05
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }}"
          
      # Alerte : Latence excessive
      - alert: HighLatency
        expr: |
          histogram_quantile(0.95, 
            rate(http_request_duration_seconds_bucket[5m])
          ) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High API latency"
          description: "P95 latency is {{ $value }}s"
          
      # Alerte : BDD pool saturé
      - alert: DatabasePoolSaturated
        expr: database_pool_active / database_pool_max > 0.9
        for: 3m
        labels:
          severity: warning
        annotations:
          summary: "Database connection pool saturated"
          
      # Alerte : Espace disque faible
      - alert: LowDiskSpace
        expr: node_filesystem_avail_bytes / node_filesystem_size_bytes < 0.1
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
```

#### Configuration Alertmanager (Slack)

```yaml
# alertmanager.yml
global:
  slack_api_url: 'https://hooks.slack.com/services/XXX/YYY/ZZZ'

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'slack-notifications'
  
  routes:
    - match:
        severity: critical
      receiver: 'slack-critical'
      continue: true
      
    - match:
        severity: warning
      receiver: 'slack-warning'

receivers:
  - name: 'slack-critical'
    slack_configs:
      - channel: '#alerts-production'
        title: '🚨 CRITICAL: {{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
        
  - name: 'slack-warning'
    slack_configs:
      - channel: '#alerts-production'
        title: '⚠️ WARNING: {{ .GroupLabels.alertname }}'
        
  - name: 'slack-notifications'
    slack_configs:
      - channel: '#monitoring'
```

**Exemple Message Slack** :
```
🚨 CRITICAL: APIDown
Instance api-backend-1:3000 is unreachable
Firing since: 2026-02-02 11:34:22
```

---

### 4.2.4. Health Checks (Endpoints Santé)

#### Endpoints IT4

| Endpoint | Usage | Réponse |
| :--- | :--- | :--- |
| `GET /health` | Load balancer health check | 200 OK si système UP |
| `GET /ready` | Kubernetes readiness probe | 200 OK si prêt à recevoir trafic |
| `GET /live` | Kubernetes liveness probe | 200 OK si process vivant |

#### Implémentation

```javascript
// GET /health : Santé globale
app.get('/health', async (req, res) => {
  const checks = {
    uptime: process.uptime(),
    timestamp: Date.now(),
    status: 'healthy',
    checks: {}
  };
  
  // Vérifier BDD
  try {
    await pool.query('SELECT 1');
    checks.checks.database = { status: 'up' };
  } catch (err) {
    checks.status = 'unhealthy';
    checks.checks.database = { status: 'down', error: err.message };
  }
  
  // Vérifier Redis
  try {
    await redisClient.ping();
    checks.checks.cache = { status: 'up' };
  } catch (err) {
    checks.checks.cache = { status: 'down', error: err.message };
  }
  
  // Vérifier ERP (via circuit breaker)
  checks.checks.erp = {
    status: erpCircuitBreaker.opened ? 'degraded' : 'up'
  };
  
  const statusCode = checks.status === 'healthy' ? 200 : 503;
  res.status(statusCode).json(checks);
});

// GET /ready : Prêt à recevoir trafic
app.get('/ready', (req, res) => {
  // Vérifier que l'app a bien démarré
  if (!appInitialized) {
    return res.status(503).json({ ready: false });
  }
  
  res.status(200).json({ ready: true });
});

// GET /live : Process vivant
app.get('/live', (req, res) => {
  res.status(200).json({ alive: true });
});
```

**Configuration NGINX Load Balancer** :

```nginx
upstream backend {
    server api-backend-1:3000 max_fails=3 fail_timeout=30s;
    server api-backend-2:3000 max_fails=3 fail_timeout=30s;
    
    # 🆕 Health check
    check interval=3000 rise=2 fall=3 timeout=1000 type=http;
    check_http_send "GET /health HTTP/1.0\r\n\r\n";
    check_http_expect_alive http_2xx;
}
```

---

### 4.2.5. Documentation API (Swagger/OpenAPI)

#### Génération Auto Swagger

```javascript
const swaggerJsdoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

const swaggerOptions = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'Restaurant API',
      version: '2.0.0',
      description: 'API de gestion restaurant (Commandes, Paiements, Menu)',
      contact: {
        name: 'Support Technique',
        email: 'support@restaurant.com'
      }
    },
    servers: [
      { url: 'https://api.restaurant.com', description: 'Production' },
      { url: 'http://localhost:3000', description: 'Development' }
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT'
        }
      }
    },
    security: [{ bearerAuth: [] }]
  },
  apis: ['./routes/*.js']
};

const swaggerSpec = swaggerJsdoc(swaggerOptions);

// 🆕 IT4 : Endpoint documentation
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));
```

#### Annotations Routes

```javascript
/**
 * @swagger
 * /api/orders:
 *   post:
 *     summary: Créer une nouvelle commande
 *     tags: [Orders]
 *     security:
 *       - bearerAuth: []
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             properties:
 *               table_id:
 *                 type: integer
 *                 example: 5
 *               items:
 *                 type: array
 *                 items:
 *                   type: object
 *                   properties:
 *                     dish_id:
 *                       type: integer
 *                     quantity:
 *                       type: integer
 *     responses:
 *       201:
 *         description: Commande créée
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 order_id:
 *                   type: integer
 *                 status:
 *                   type: string
 *       401:
 *         description: Non autorisé
 *       403:
 *         description: Permissions insuffisantes
 */
app.post('/api/orders', authenticateJWT, checkRole(['Serveur', 'Admin']), OrderController.create);
```

**URL Documentation** : `https://api.restaurant.com/api-docs`

---

## 4.3. Load Balancer (Préparation Scaling)

### Configuration NGINX

```nginx
http {
    upstream backend_pool {
        least_conn; # Répartition par charge
        
        server api-backend-1:3000 weight=1;
        server api-backend-2:3000 weight=1;
        # Prêt pour api-backend-3 futur
        
        keepalive 32; # Connexions persistantes
    }
    
    server {
        listen 443 ssl http2;
        server_name api.restaurant.com;
        
        ssl_certificate /etc/ssl/certs/restaurant.crt;
        ssl_certificate_key /etc/ssl/private/restaurant.key;
        ssl_protocols TLSv1.3;
        
        location / {
            proxy_pass http://backend_pool;
            proxy_http_version 1.1;
            
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Connection "";
            
            # Timeouts
            proxy_connect_timeout 5s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
        }
        
        # WebSocket support
        location /socket.io/ {
            proxy_pass http://backend_pool;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
    }
}
```

**Capacité Actuelle** : 2 backends = ~100 requêtes/s.  
**Scaling Futur** : Ajout backend-3 = ~150 requêtes/s (simple décommentaire config).

---

## 4.4. Évaluation IT4 selon Critères de Qualité

| Critère | Note IT3 | Note IT4 | Évolution | Justification IT4 |
| :--- | :---: | :---: | :---: | :--- |
| **Fiabilité** | 🟢 5/5 | 🟢 5/5 | = | Maintenue IT3 + Alerting proactif |
| **Performance** | 🟢 5/5 | 🟢 5/5 | = | Maintenue IT3 + Load balancer |
| **Sécurité** | 🟢 5/5 | 🟢 5/5 | = | Maintenue IT2 |
| **Maintenabilité** | 🟢 4/5 | 🟢 5/5 | +1 | Logs centralisés + Doc API Swagger |
| **Évolutivité** | 🟡 4/5 | 🟢 5/5 | +1 | Load balancer + Monitoring prepare multi-sites |

**Score global IT4** : **25/25** (100%) — Score parfait ! 🎉

---

## 4.5. Livrables IT4

### Artefacts Produits
- ✅ **ELK Stack** : Configuration Elasticsearch + Logstash + Kibana.
- ✅ **Prometheus + Grafana** : Métriques temps réel + Dashboards.
- ✅ **Alertmanager** : Alerting Slack automatique.
- ✅ **Health Checks** : Endpoints `/health`, `/ready`, `/live`.
- ✅ **Load Balancer** : NGINX configuré.
- ✅ **Documentation API** : Swagger UI interactif.

### Dashboards Créés
1. **Restaurant Overview** : KPIs globaux (commandes/h, chiffre affaires, taux erreurs).
2. **Performance SLO** : Suivi SLO (99% des requêtes < 200ms).
3. **Infrastructure Health** : BDD pool, cache hit rate, WebSocket actifs.

---

## 4.6. Budget et Délai IT4

**Coût IT4** : 4 500 € HT (inclus dans enveloppe).  
**Détail** :
- Configuration monitoring (ELK + Prometheus) : 2 000 €
- Dashboards Grafana personnalisés : 1 000 €
- Documentation Swagger : 800 €
- Tests charge finale : 700 €

**Délai IT4** : 2 semaines.

---

## Conclusion IT4

L'**Itération 4** finalise le système avec **excellence opérationnelle**. Le monitoring centralisé et l'alerting automatique garantissent détection proactive des incidents. Le load balancer prépare l'expansion future vers multi-restaurants.

**Points forts IT4** :
- ✅ Observabilité complète (logs + métriques + alertes).
- ✅ Debugging simplifié (Kibana full-text search).
- ✅ Préparation scaling horizontal (load balancer).
- ✅ Documentation API professionnelle (Swagger).
- ✅ Score qualité : 25/25 (100%).

**Système Production-Ready** : ✅  
**Expansion Multi-Sites Ready** : ✅  
**Maintenance Long-Terme Ready** : ✅  

**Prochaines étapes** : C5 (Choix Technologies Détaillé), C6 (Validation Architecture), C7 (Diagrammes UML).
