# 4. Zaprojektowanie Architektury Rozwiązania/Systemu

## Spis Treści

1. [Koncepcja Ogólna](#koncepcja-ogólna)
2. [Diagram Architektury Wysokopoziomowej (C4 Level 1)](#diagram-c4-level-1)
3. [Architektura Techniczna (Level 2)](#architektura-techniczna)
4. [Opis Komponentów Systemu](#opis-komponentów-systemu)
5. [Przepływy Danych](#przepływy-danych)
6. [Integracje Zewnętrzne](#integracje-zewnętrzne)
7. [Architektura Wdrażania](#architektura-wdrażania)
8. [Bezpieczeństwo](#bezpieczeństwo)
9. [Skalowanie i Monitoring](#skalowanie-i-monitoring)

---

## Koncepcja Ogólna

### 🎯 Cel Architektury

System BeExpress zaprojektowany został według **nowoczesnych standardów cloud-native**:
- **Microservices Architecture** - niezależne, skalowalne usługi
- **Event-Driven** - asynchroniczna komunikacja
- **API-First** - wszystko dostępne przez API
- **Cloud-Ready** - wdrażany na AWS
- **Highly Available** - 99.95% uptime SLA

### 📊 Założenia Projektowe

| Parametr | Wartość |
|----------|---------|
| **Liczba warsztatów** | 50+ |
| **Liczba kierowców** | 5-10 |
| **Liczba pracowników biura** | 5 |
| **Indeksów części** | 15,000+ |
| **Średnie zamówienia/dzień** | 200-300 |
| **Peak traffic (req/s)** | 500-1000 |
| **Uptime SLA** | 99.95% |
| **Czas odpowiedzi API** | <200ms (p95) |

---

## Diagram C4 Level 1

### Architektura Wysokopoziomowa

```
┌────────────────────────────────────────────────────────────────┐
│                         UŻYTKOWNICY                             │
├─────────────┬──────────────┬──────────────┬────────────────────┤
│  🏢         │  🚗          │  👔         │  ⚙️                │
│  Warsztaty  │  Kierowcy    │  Biuro      │  Administratorzy   │
│  (50+)      │  (5-10)      │  (5 osób)   │                    │
└─────────────┴──────────────┴──────────────┴────────────────────┘
                              ▼
┌────────────────────────────────────────────────────────────────┐
│                    BeExpress Platform                           │
├──────────────────┬───────────────────┬────────────────────────┤
│ 🌐 Portal Web    │ 📱 Mobile App     │ 📊 Admin Panel       │
│ React + Redux    │ React Native      │ React Admin          │
│ localhost:3000   │ iOS/Android       │ localhost:3001       │
└──────────────────┴───────────────────┴────────────────────────┘
                              ▼
┌────────────────────────────────────────────────────────────────┐
│                    🔌 API GATEWAY (Kong)                        │
│      Authentication | Rate Limiting | Load Balancing           │
└────────────────────────────────────────────────────────────────┘
                              ▼
┌──────────────┬──────────────┬──────────────┬──────────────────┐
│  📦 Magazyn  │  🚚 Logist.  │  💳 Płatno- │  👤 Użytkownicy  │
│  Service     │  Service     │  ści Service │  Service         │
│  (Java)      │  (Node.js)   │  (Node.js)   │  (Node.js)       │
└──────────────┴──────────────┴──────────────┴──────────────────┘
                              ▼
┌──────────────┬──────────────┬──────────────┬──────────────────┐
│  🐘 Postgre  │  ⚡ Redis    │  🔍 Elastic │  📝 MongoDB      │
│  SQL Baza    │  Cache       │  Search      │  Logi            │
└──────────────┴──────────────┴──────────────┴──────────────────┘
```

---

## Architektura Techniczna

### Level 2 - Szczegółowy Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                   CLIENT LAYER (Presentation)                    │
├─────────────────┬────────────────────┬────────────────────────┤
│  🌐 Portal Web  │  📱 Mobile App     │  📊 Admin Panel       │
│  SPA - React    │  React Native      │  React Admin          │
│  Redux Store    │  Redux/MobX        │  Material-UI          │
│  TypeScript     │  TypeScript        │  Charts (Recharts)    │
└─────────────────┴────────────────────┴────────────────────────┘
         │                     │                      │
         └─────────────────────┼──────────────────────┘
                               ▼
    ┌───────────────────────────────────────────────────────┐
    │  🌐 CDN (CloudFront)                                  │
    │  - Cache static assets (CSS, JS, Images)              │
    │  - Compression (Gzip/Brotli)                          │
    │  - HTTPS/TLS 1.3                                      │
    └───────────────────────────────────────────────────────┘
                               ▼
    ┌───────────────────────────────────────────────────────┐
    │  🔐 WAF (Web Application Firewall)                    │
    │  - DDoS Protection                                    │
    │  - SQL Injection Prevention                           │
    │  - Rate Limiting (1000 req/min per IP)                │
    │  - Geolocation Blocking                               │
    └───────────────────────────────────────────────────────┘
                               ▼
    ┌───────────────────────────────────────────────────────┐
    │  🔌 API GATEWAY (Kong) - Kubernetes Service           │
    │  - Load Balancing (Round Robin)                       │
    │  - Service Discovery                                  │
    │  - Request/Response Logging                           │
    │  - JWT Token Validation                               │
    │  - OAuth2 Integration                                 │
    │  - CORS Configuration                                 │
    │  - Request/Response Transformation                    │
    └───────────────────────────────────────────────────────┘
         │                     │                      │
    ┌────┴────┬────────────────┼────────────────┬───┴────┐
    ▼         ▼                ▼                ▼        ▼
  ┌───────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
  │ Maga- │ │ Logis-   │ │ Płatno-  │ │ Użytko-  │ │ Notifi-  │
  │ zyn   │ │ tyka     │ │ ści      │ │ wnicy    │ │ kacje    │
  │Service│ │Service   │ │ Service  │ │ Service  │ │ Service  │
  └───────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘
    (Java)   (Node.js)   (Node.js)    (Node.js)    (Node.js)
    Spring   Express     Express      Express      Express
    Boot     Mongoose    Mongoose     Mongoose     Mongoose
```

### Każdy Microservice (Kontener Docker)

```
┌─────────────────────────────────────────────────┐
│           MICROSERVICE CONTAINER                │
├─────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────┐  │
│  │  REST API Endpoints                      │  │
│  │  /api/v1/resource/{id}                   │  │
│  │  GET/POST/PUT/DELETE                     │  │
│  └──────────────────────────────────────────┘  │
│         │              │              │        │
│         ▼              ▼              ▼        │
│  ┌──────────────────────────────────────────┐  │
│  │  Business Logic Layer                    │  │
│  │  - Validation                            │  │
│  │  - Processing                            │  │
│  │  - Calculations                          │  │
│  │  - Event Publishing (Kafka)              │  │
│  └──────────────────────────────────────────┘  │
│         │              │              │        │
│         ▼              ▼              ▼        │
│  ┌──────────────────────────────────────────┐  │
│  │  Data Access Layer (Repository Pattern)  │  │
│  │  - ORM/Query Builder                     │  │
│  │  - Connection Pooling                    │  │
│  │  - Caching Logic                         │  │
│  └──────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
```

---

## Opis Komponentów Systemu

### 1. Frontend - Portal Web (React)

**Technologia**: React 18, Redux, Material-UI, TypeScript  
**Hosting**: AWS CloudFront + S3  
**Port**: 3000

**Główne Moduły**:
```
├── Authentication (Login/Register/2FA)
├── Dashboard (Overview zamówień)
├── Parts Search (Wyszukiwanie po VIN)
├── Orders (Rezerwacje, historia)
├── Delivery Tracking (Mapa real-time)
├── Payments (BLIK, Karta, Przelew)
├── Profile Management (Dane warsztatu)
└── Notifications (SMS, Email, Push)
```

### 2. Frontend - Aplikacja Mobilna (React Native)

**Technologia**: React Native, Expo, Redux  
**Platform**: iOS + Android  
**Dla**: Kierowców

**Główne Funkcje**:
```
├── GPS Tracking (co 30 sekund)
├── Route Navigation (Google Maps integration)
├── QR Code Scanning (Warehouse items)
├── Delivery Confirmation (Photo + Signature)
├── Real-time Chat (Kierowca ↔ Warsztat)
├── Offline Mode (Sync gdy online)
└── Driver Statistics (Przebiegane km, czas)
```

### 3. API Gateway (Kong)

**Technologia**: Kong Community Edition  
**Funkcje**:
- Load Balancing (across services)
- Authentication (JWT, OAuth2)
- Rate Limiting (1000 req/min global)
- Request/Response Logging
- Service Discovery
- CORS, HTTPS termination

**Konfiguracja**:
```yaml
Kong:
  database: PostgreSQL
  cache: Redis
  plugins:
    - request-transformer
    - response-transformer
    - jwt
    - rate-limiting
    - cors
    - request-size-limiting
```

### 4. Magazyn Service (Java/Spring Boot)

**Technologia**: Java 17, Spring Boot 3, Maven  
**Port**: 8001  
**Database**: PostgreSQL (tabela: parts, warehouse_stock, warehouse_movements)

**REST Endpoints**:
```
GET    /api/v1/parts/search?vin=ABC123        # Search parts by VIN
GET    /api/v1/parts/{partId}/availability    # Check availability
POST   /api/v1/warehouse/scan                 # Scan part (QR)
PUT    /api/v1/warehouse/{partId}/reserve     # Reserve part
PUT    /api/v1/warehouse/{partId}/release     # Release reservation
GET    /api/v1/inventory/report               # Inventory report
POST   /api/v1/warehouse/movements            # Log movement
```

**Baza Danych - Schema**:
```sql
-- Główna tabela części
CREATE TABLE parts (
    id SERIAL PRIMARY KEY,
    sku VARCHAR(50) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    vin_compatibility JSON,
    price DECIMAL(10, 2),
    supplier_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP
);

-- Stan magazynowy
CREATE TABLE warehouse_stock (
    id SERIAL PRIMARY KEY,
    part_id INT REFERENCES parts(id),
    location VARCHAR(50),          -- np. "A-12-3" (klatka-półka-pozycja)
    quantity_available INT,
    quantity_reserved INT,
    quantity_damaged INT,
    last_counted_at TIMESTAMP,
    created_at TIMESTAMP
);

-- Ruchy magazynowe (audit trail)
CREATE TABLE warehouse_movements (
    id SERIAL PRIMARY KEY,
    part_id INT REFERENCES parts(id),
    movement_type VARCHAR(50),     -- PICK, RESERVE, RELEASE, DAMAGE
    quantity INT,
    user_id INT,
    location_from VARCHAR(50),
    location_to VARCHAR(50),
    reference_id INT,              -- reservation/order ID
    created_at TIMESTAMP
);
```

### 5. Logistyka Service (Node.js/Express)

**Technologia**: Node.js 18, Express, Mongoose  
**Port**: 8002  
**Database**: MongoDB (collections: deliveries, routes, driver_locations)

**REST Endpoints**:
```
POST   /api/v1/deliveries/create                    # Create delivery
GET    /api/v1/deliveries/{deliveryId}             # Get delivery details
PUT    /api/v1/deliveries/{deliveryId}/route       # Optimize route
POST   /api/v1/drivers/gps-update                  # GPS update
WS     /ws/deliveries/{deliveryId}/tracking        # WebSocket stream
GET    /api/v1/routes/optimize?stops=10            # TSP algorithm
POST   /api/v1/deliveries/{deliveryId}/confirm     # Confirm delivery
```

**WebSocket Events**:
```javascript
// Client subscribes to tracking
client.subscribe(`delivery:${deliveryId}`);

// Server publishes updates every 30 seconds
{
  event: "LOCATION_UPDATE",
  driver_id: "DRV001",
  lat: 52.2297,
  lng: 21.0122,
  eta_minutes: 15,
  distance_km: 2.3,
  timestamp: 1652348520000
}
```

### 6. Płatności Service (Node.js/Express)

**Technologia**: Node.js 18, Express  
**Port**: 8003  
**Database**: PostgreSQL (tabela: transactions, invoices, payment_methods)

**REST Endpoints**:
```
POST   /api/v1/payments/create                  # Initiate payment
POST   /api/v1/payments/blik                    # BLIK payment
POST   /api/v1/payments/card                    # Card payment
GET    /api/v1/payments/{transactionId}         # Get transaction
POST   /api/v1/invoices/generate                # Generate invoice
GET    /api/v1/invoices/{invoiceId}             # Get invoice
```

**Integracja PayU**:
```javascript
// POST https://secure.snd.payu.com/api/v2_1/orders
{
  "notifyUrl": "https://api.beexpress.pl/webhooks/payu",
  "customerIp": "127.0.0.1",
  "merchantPosId": "YOUR_POS_ID",
  "description": "Zamówienie #ORD001",
  "currencyCode": "PLN",
  "totalAmount": 15000,    // w groszach
  "buyer": {
    "email": "info@warsztat.pl",
    "phone": "+48123456789"
  },
  "products": [{
    "name": "Klocki hamulcowe",
    "unitPrice": 15000,
    "quantity": 1
  }]
}
```

### 7. Bazy Danych

#### PostgreSQL (Relacyjna)
```
Tabele główne:
├── users (autoryzacja)
├── workshops (warsztaty)
├── parts (części)
├── warehouse_stock (stan magazynu)
├── warehouse_movements (ruchy)
├── orders (zamówienia)
├── reservations (rezerwacje)
├── transactions (płatności)
└── invoices (faktury)
```

#### Redis (Cache)
```
Keys:
├── user:{userId}:session         # Session storage
├── part:{partId}:availability    # Cache dostępności (TTL: 5min)
├── driver:{driverId}:location    # Bieżąca lokalizacja
├── rate_limit:{ipAddress}        # Rate limiting counter
└── order:{orderId}:status        # Cache statusu
```

#### Elasticsearch (Search)
```
Indices:
├── parts-*                       # Indeks części (searched by name, SKU)
├── warehouse-movements-*         # Historia ruchów
└── orders-*                      # Historia zamówień
```

#### MongoDB (Document Store - opcjonalne)
```
Collections:
├── deliveries                    # GPS trackingu
├── driver_locations              # Historyczne lokalizacje
├── audit_logs                    # Audyt systemowy
└── notifications                 # Powiadomienia
```

---

## Przepływy Danych

### Scenariusz 1: Rezerwacja Części przez Warsztat

```
1. Warsztat wchodzi w portal → React aplikacja
2. Wpisuje numer VIN → React wysyła GET /api/search/vin
3. API Gateway routuje do Magazyn Service
4. Magazyn Service:
   - Sprawdza cache Redis (część dostępna?)
   - Jeśli miss: SELECT z PostgreSQL
   - Zwraca listę części kompatybilnych
5. Portal wyświetla listę części z dostępnością
6. Warsztat klika "Zarezerwuj"
7. React wysyła POST /api/reservations/create
8. API Gateway routuje do Logistyka Service
9. Logistyka Service:
   - INSERT do orders table
   - PUBLISH event na Kafka topic: "order.created"
   - Informuje Magazyn Service (RPC): aktualizuj stan
10. Magazyn Service:
    - UPDATE warehouse_stock: reserved +1
    - INVALIDATE cache Redis
11. Notifikacja Service (listens to Kafka):
    - Wysyła SMS do Warsztatu: "Zarezerwowano części, ETA: 24h"
12. Portal wyświetla potwierdzenie ✓

Timeline: ~500ms (P95)
```

### Scenariusz 2: Skanowanie Części w Magazynie

```
1. Magazynier skanuje kod QR → Mobile App
2. POST /api/warehouse/scan {part_id, location}
3. API Gateway routuje do Magazyn Service
4. Magazyn Service:
   - SELECT część z PostgreSQL
   - UPDATE warehouse_movements (audit trail)
   - UPDATE warehouse_stock (quantity_available -1)
   - INVALIDATE cache Redis
5. Response zwraca etykietę do druku
6. Drukarka wysyła etykietę (label)
7. Magazynier pakuje część i kładzie na rampie
8. Kierowca odbiera skrzynkę

Feedback do magazyniera: ~300ms (beep ✓)
Dokładność stanu magazynu: 99.8% (zamiast 70%)
```

### Scenariusz 3: GPS Tracking Dostawy

```
1. Kierowca uruchamia aplikację mobilną
2. Co 30 sekund: POST /api/logistics/gps-update
   {driver_id, lat, lng, timestamp}
3. API Gateway routuje do Logistyka Service
4. Logistyka Service:
   - SET driver:{driverId}:location w Redis
   - PUBLISH event na WebSocket server
5. WebSocket server broadcast do wszystkich Warsztatów:
   {event: "LOCATION_UPDATE", driver, lat, lng, eta}
6. Portal Warsztatu: aktualizuje mapę w real-time
7. Warsztat widzi:
   - 📍 Kierowca Marek 2 km stąd
   - ⏱️ ETA: 15 minut

Latency: ~50-100ms (real-time experience)
```

### Scenariusz 4: Płatność BLIK

```
1. Kierowca dostarcza części do Warsztatu
2. Warsztat klika "Zapłać" w portalu
3. Modal wyświetla opcje płatności (BLIK, Karta, Przelew)
4. Warsztat wybiera "BLIK"
5. Portal wysyła POST /api/payments/blik
6. Płatności Service:
   - Inicjuje transakcję w PayU
   - Zwraca URL do aplikacji bankowej
7. Warsztat otwiera aplikację banku, skanuje kod BLIK
8. PayU przetwarza płatność (2-3 sekund)
9. PayU wysyła webhook na:
   POST https://api.beexpress.pl/webhooks/payu
   {order_id, status: "COMPLETED", amount: 150.00}
10. Płatności Service:
    - UPDATE transactions: status = PAID
    - PUBLISH event: "payment.completed"
11. Logistyka Service (listens):
    - UPDATE orders: status = COMPLETED
    - PUBLISH event do Warsztatu
12. Portal wyświetla potwierdzenie ✓
    Invoice automatycznie wysłana na email

Timeline: ~3-5 sekund
```

---

## Integracje Zewnętrzne

### 1. PayU (Płatności)

```
Metoda: REST API + Webhooks
Endpoint: https://secure.snd.payu.com/api/v2_1/orders

Obsługiwane metody:
- BLIK
- Karta kredytowa/debetowa
- Przelew bankowy
- e-Portfele

Authentication: OAuth2 (POS ID + Signature)
Response: JSON z redirectUrl

Error Handling:
- Retry logic: exponential backoff
- Fallback: przechowanie transakcji w status PENDING
- Reconciliation job: daily cron
```

### 2. Google Maps (Trasy)

```
Metoda: REST API
Endpoints:
- Directions API  (planning tras)
- Distance Matrix API (macierz odległości)
- Geocoding API (adresy → koordinaty)

Użycie:
1. Logistyka Service pobiera zamówienia (10+ miast)
2. Geocoduje adresy warsztatów
3. Wywołuje Distance Matrix API
4. Solver TSP (Travelling Salesman Problem)
5. Optymalizuje trasę dla kierowcy

Rezultat: -30% czasów przejazdu
```

### 3. Twilio (Powiadomienia)

```
Metoda: REST API + Webhooks
Endpoints:
- /Messages  (SMS)
- /Emails    (Email via SendGrid)

Use cases:
- SMS: "Zarezerwowano części, ETA: 24h"
- SMS: "Kierowca Marek przyjedzie za 15 minut"
- Email: "Faktura nr INV-2024-001"
- SMS: "Zwrot zaakceptowany, kwota zwrócona"

Rate: ~500 SMS/dzień
Cost: ~0.5 PLN/SMS = ~250 PLN/dzień
```

### 4. Firebase (Push Notifications)

```
Metoda: Cloud Messaging
Endpoints:
- /fcm/send (send notifications)

Use cases:
- Push: "Nowe zamówienie w systemie"
- Push: "Kierowca przyjedzie za 15 min"
- Push: "Płatność zaakceptowana"

Clients:
- Web: Firebase JS SDK
- Mobile: Firebase React Native

Delivery: ~95% w ciągu 5 sekund
```

---

## Architektura Wdrażania

### Środowiska

```
┌─────────────────────────────────────────────────────────────┐
│                    DEVELOPMENT                              │
├─────────────────────────────────────────────────────────────┤
│ Docker Compose na laptopie deweloperskim                    │
│ - Frontend: http://localhost:3000                           │
│ - Backend: http://localhost:8000                            │
│ - PostgreSQL: localhost:5432                                │
│ - Redis: localhost:6379                                     │
│ - Elasticsearch: localhost:9200                             │
│ Dane: fake data (50 części, 5 warsztatów)                  │
└─────────────────────────────────────────────────────────────┘
                          ▼ (commit → push)
┌─────────────────────────────────────────────────────────────┐
│                    STAGING                                  │
├─────────────────────────────────────────────────────────────┤
│ AWS EC2 instance (t3.xlarge)                                │
│ - Frontend: https://staging.beexpress.pl                   │
│ - API: https://api-staging.beexpress.pl                    │
│ - Kubernetes: 2 replicas per service                        │
│ Dane: Kopia danych PROD z 7 dni temu (anonimizowana)      │
│ Users: Internal users only                                  │
│ SLA: Best effort                                            │
└─────────────────────────────────────────────────────────────┘
              ▼ (manual approval → code review)
┌─────────────────────────────────────────────────────────────┐
│                    PRODUCTION                               │
├─────────────────────────────────────────────────────────────┤
│ AWS EKS (Elastic Kubernetes Service)                        │
│ - Frontend: https://app.beexpress.pl                        │
│ - API: https://api.beexpress.pl                             │
│ - Kubernetes: 3-10 replicas per service (auto-scaling)     │
│ - Multi-AZ (3 availability zones)                           │
│ - Database: RDS PostgreSQL (Multi-AZ)                       │
│ - Cache: ElastiCache Redis (cluster mode)                   │
│ Dane: Real data                                             │
│ Users: All customers                                        │
│ SLA: 99.95% uptime                                          │
│ Backup: Every 6 hours → S3                                  │
└─────────────────────────────────────────────────────────────┘
```

### CI/CD Pipeline (GitHub Actions)

```yaml
name: Deploy to Production

on:
  push:
    branches: [ main ]
  
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Unit Tests
        run: npm test
      - name: Run Integration Tests
        run: npm run test:integration
      - name: Code Quality
        run: npx sonarqube-scanner
      - name: Security Scan
        run: npm audit
  
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Build Docker Image
        run: docker build -t $ECR_REGISTRY/beexpress:${{ github.sha }} .
      - name: Push to ECR
        run: docker push $ECR_REGISTRY/beexpress:${{ github.sha }}
  
  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Staging
        run: kubectl set image deployment/beexpress-staging
  
  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
    steps:
      - name: Approval Required
        run: echo "Waiting for approval..."
      - name: Deploy to Production
        run: kubectl set image deployment/beexpress-prod
      - name: Health Check
        run: curl https://api.beexpress.pl/health
```

---

## Bezpieczeństwo

### 7 Warstw Bezpieczeństwa

```
┌────────────────────────────────────┐
│ 🔐 Layer 1: HTTPS/TLS 1.3          │
│    Encryption in Transit           │
└────────────────────────────────────┘
             ▼
┌────────────────────────────────────┐
│ 🛡️ Layer 2: WAF + DDoS Protection │
│    Cloudflare / AWS Shield         │
└────────────────────────────────────┘
             ▼
┌────────────────────────────────────┐
│ 🔑 Layer 3: Authentication & RBAC  │
│    JWT + OAuth2 + 2FA              │
│    - Super Admin (CAN_ALL)         │
│    - Workshop Manager (READ_ORDERS)│
│    - Driver (UPDATE_DELIVERY)      │
│    - Viewer (READ_ONLY)            │
└────────────────────────────────────┘
             ▼
┌────────────────────────────────────┐
│ ⚙️ Layer 4: Business Logic         │
│    - Input Validation              │
│    - Rate Limiting                 │
│    - Fraud Detection               │
└────────────────────────────────────┘
             ▼
┌────────────────────────────────────┐
│ 🔒 Layer 5: Encryption at Rest     │
│    - AES-256 (database)            │
│    - bcrypt (passwords)            │
│    - HSM (PayU credentials)        │
└────────────────────────────────────┘
             ▼
┌────────────────────────────────────┐
│ 📝 Layer 6: Auditing & Logging     │
│    - ELK Stack (Elasticsearch)     │
│    - CloudTrail (AWS events)       │
│    - All changes logged             │
└────────────────────────────────────┘
             ▼
┌────────────────────────────────────┐
│ 🚨 Layer 7: Monitoring & Alerts    │
│    - Prometheus + Grafana          │
│    - Real-time anomaly detection   │
│    - Email/Slack alerts            │
└────────────────────────────────────┘
```

### Specyfikacja Bezpieczeństwa

| Obszar | Implementacja |
|--------|---|
| **HTTPS** | TLS 1.3, HSTS headers |
| **Hasła** | bcrypt, min 12 znaków |
| **Sesje** | JWT, TTL 24h, refresh token |
| **2FA** | TOTP (Time-based OTP) |
| **RBAC** | 4 role: Admin, Manager, Driver, Viewer |
| **API Auth** | OAuth2 + JWT |
| **Podatności** | OWASP Top 10 mitigation |
| **Skanowanie** | Weekly vulnerability scans |
| **RODO** | GDPR compliant (data export, right to be forgotten) |
| **PCI DSS** | Level 1 compliance (through PayU) |
| **Backup** | Daily encrypted backups to S3 |
| **DLP** | Data Loss Prevention rules |

---

## Skalowanie i Monitoring

### Kubernetes Auto-scaling

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: beexpress-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: beexpress-api
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 100
        periodSeconds: 30
```

### Monitoring (Prometheus + Grafana)

```
Metryki monitorowane:
├── API Response Time (ms)          # Target: <200ms (P95)
├── Error Rate (%)                   # Target: <0.1%
├── Request Rate (req/s)             # Actual: 200-500
├── Database Connections (%)         # Target: <80%
├── Cache Hit Rate (%)               # Target: >85%
├── Pod CPU Usage (%)                # Target: <70%
├── Pod Memory Usage (%)             # Target: <80%
├── Disk I/O (ops/sec)              # Monitor for saturation
├── Network Latency (ms)            # Target: <50ms
└── Uptime (%)                      # Target: 99.95%

Alerts (Slack/Email):
├── API error rate > 1% 🚨
├── Response time > 500ms ⚠️
├── Pod restart count > 3
├── Database connection pool exhausted
├── Disk usage > 90%
└── Service down (health check failed)
```

### Exemplo Dashboardu Grafana

```
Top: CPU Usage (gauge)
     Memory Usage (gauge)
     Error Rate (gauge)

Middle: 
  - Request Rate (line chart) - last 24h
  - Response Time P95 (line chart) - last 24h
  - Error Rate Trend (area chart) - last 7d

Bottom:
  - Service Health Status (table)
  - Top 5 Slowest Endpoints (table)
  - Recent Alerts (log panel)
```

---

## Podsumowanie Architektury

### ✅ Zalety Tego Podejścia

| Feature | Benefit |
|---------|---------|
| **Microservices** | Niezależne skalowanie, łatwe deploymenty |
| **API Gateway** | Centralna autentykacja, rate limiting |
| **Kubernetes** | Auto-healing, auto-scaling, rolling updates |
| **Event-Driven** | Loose coupling, asynchroniczna komunikacja |
| **Cloud-Native** | Elastyczność, disaster recovery, backup |
| **Monitoring** | Real-time insights, proactive alerts |
| **Security Layers** | Defense-in-depth podejście |

### 📊 Spodziewane Parametry

| Metryka | Wartość |
|---------|---------|
| **Latency P95** | ~150-200ms |
| **Latency P99** | ~300-400ms |
| **Throughput** | 1000+ req/s |
| **Error Rate** | <0.1% |
| **Availability** | 99.95% |
| **Cache Hit Rate** | >85% |
| **Deploy Time** | <5 minut |
| **MTTR** | <15 minut |

### 🎯 Następne Kroki

1. **Infrastructure Setup** - Terraform dla AWS
2. **Microservices Development** - kod backend
3. **Frontend Development** - React portal
4. **Integration Testing** - end-to-end testy
5. **Load Testing** - 1000 req/s test
6. **Security Testing** - penetration testing
7. **UAT** - acceptance testy
8. **Go-Live** - wdrożenie na produkcję

---

**Wersja:** 1.0  
**Data:** 2026-05-12  
**Status:** ✅ Approved
