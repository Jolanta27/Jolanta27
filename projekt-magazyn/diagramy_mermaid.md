# Diagramy Architektury - Format Mermaid

## Diagram 1: Architektura C4 Level 1 (Wysokopoziomowa)

```mermaid
graph TB
    subgraph Users["Użytkownicy"]
        WKS["Warsztaty<br/>(Mechanicy)"]
        KIR["Kierowcy"]
        BIU["Biuro<br/>(Pracownicy)"]
        ADM["Administratorzy"]
    end

    subgraph BeExpress["BeExpress Platform"]
        WEB["Portal Webowy<br/>(React)"]
        MOB["Aplikacja Mobilna<br/>(React Native)"]
        ADMIN["Panel Admin<br/>(React)"]
    end

    subgraph API["API Gateway & Backend"]
        GW["API Gateway<br/>(Kong)"]
        MAG["Magazyn Service<br/>(Java)"]
        LOG["Logistyka Service<br/>(Node.js)"]
        PAY["Płatności Service<br/>(Node.js)"]
    end

    subgraph DATA["Dane"]
        PG["PostgreSQL<br/>(relacyjna)"]
        REDIS["Redis<br/>(cache)"]
        ES["Elasticsearch<br/>(wyszukiwanie)"]
    end

    subgraph EXT["Integracje"]
        PAYU["PayU<br/>(płatności)"]
        MAPS["Google Maps<br/>(trasy)"]
        SMS["Twilio<br/>(powiadomienia)"]
    end

    WKS --> WEB
    KIR --> MOB
    BIU --> ADMIN
    ADM --> ADMIN
    
    WEB --> GW
    MOB --> GW
    ADMIN --> GW
    
    GW --> MAG
    GW --> LOG
    GW --> PAY
    
    MAG --> PG
    MAG --> REDIS
    MAG --> ES
    
    LOG --> PG
    LOG --> REDIS
    
    PAY --> PG
    
    PAY --> PAYU
    LOG --> MAPS
    LOG --> SMS
```

---

## Diagram 2: Architektura Techniczna (Level 2)

```mermaid
graph TB
    subgraph Presentation["WARSTWA PREZENTACJI"]
        WEB["Portal Web<br/>React + Redux"]
        MOB["Mobile App<br/>React Native"]
        ADMIN["Admin Panel<br/>React Admin"]
    end

    subgraph Gateway["API GATEWAY"]
        KONG["Kong API Gateway<br/>Routing, Rate Limiting, Load Balancing"]
    end

    subgraph Services["MICROSERVICES (Kubernetes)"]
        MAG["Magazyn Service<br/>Java/Spring Boot<br/>- Inwentaryzacja<br/>- Rezerwacje<br/>- Skanowanie"]
        LOG["Logistyka Service<br/>Node.js/Express<br/>- Trasy<br/>- GPS Tracking<br/>- Powiadomienia"]
        PAY["Płatności Service<br/>Node.js/Express<br/>- Transakcje<br/>- Faktury<br/>- Rozliczenia"]
        USER["Użytkownicy Service<br/>Node.js/Express<br/>- Autentykacja<br/>- Autoryzacja<br/>- Profile"]
    end

    subgraph Data["WARSTWA DANYCH"]
        PG["PostgreSQL<br/>Database<br/>- Główna baza danych<br/>- Backup daily"]
        REDIS["Redis<br/>In-Memory Cache<br/>- Session cache<br/>- Rate limit tracking"]
        ES["Elasticsearch<br/>Search Engine<br/>- Indeks części<br/>- Historia"]
        MONGO["MongoDB<br/>Document Store<br/>- Logi aplikacji<br/>- Audyt"]
    end

    subgraph External["INTEGRACJE ZEWNĘTRZNE"]
        PAYU["PayU<br/>REST API + Webhooks"]
        MAPS["Google Maps<br/>Directions API"]
        TWILIO["Twilio<br/>SMS/Email"]
        FIREBASE["Firebase<br/>Push Notifications"]
    end

    subgraph Infra["INFRASTRUKTURA"]
        K8S["Kubernetes<br/>EKS on AWS<br/>- Auto-scaling<br/>- Load Balancing"]
        DOCKER["Docker<br/>Container Registry<br/>ECR on AWS"]
        MONITOR["Monitoring<br/>Prometheus + Grafana<br/>+ CloudWatch"]
        LOGGING["Logging<br/>ELK Stack<br/>Elasticsearch + Kibana"]
    end

    WEB --> KONG
    MOB --> KONG
    ADMIN --> KONG
    
    KONG --> MAG
    KONG --> LOG
    KONG --> PAY
    KONG --> USER
    
    MAG --> PG
    MAG --> REDIS
    MAG --> ES
    
    LOG --> PG
    LOG --> REDIS
    
    PAY --> PG
    
    USER --> PG
    USER --> REDIS
    
    PAY --> PAYU
    LOG --> MAPS
    LOG --> TWILIO
    LOG --> FIREBASE
    
    MAG -.-> K8S
    LOG -.-> K8S
    PAY -.-> K8S
    USER -.-> K8S
    
    K8S -.-> DOCKER
    K8S -.-> MONITOR
    K8S -.-> LOGGING
```

---

## Diagram 3: Przepływ Danych - Rezerwacja Części

```mermaid
sequenceDiagram
    participant Warsztat as Warsztat
    participant Portal as Portal
    participant Gateway as Gateway
    participant Magazyn as Magazyn Service
    participant Logistyka as Logistyka Service
    participant DB as PostgreSQL
    participant Cache as Redis
    participant SMS as SMS Service

    Warsztat->>Portal: 1. Wpisuje numer VIN
    Portal->>Gateway: 2. GET /api/search/vin?vin=ABC123
    Gateway->>Magazyn: 3. Routing do Magazyn Service
    Magazyn->>Cache: 4. Sprawdź cache
    Cache-->>Magazyn: 5. Cache miss
    Magazyn->>DB: 6. SELECT części dla VIN
    DB-->>Magazyn: 7. Lista części + dostępność
    Magazyn->>Cache: 8. Zaktualizuj cache (TTL: 5min)
    Magazyn-->>Gateway: 9. JSON response
    Gateway-->>Portal: 10. Zwróć listę części
    Portal-->>Warsztat: 11. Wyświetl części na liście

    Warsztat->>Portal: 12. Kliknij "Zarezerwuj"
    Portal->>Gateway: 13. POST /api/reservations/create
    Gateway->>Logistyka: 14. Routing do Logistyka Service
    Logistyka->>DB: 15. INSERT rezerwacja
    DB-->>Logistyka: 16. OK, ID rezerwacji
    Logistyka->>Magazyn: 17. RPC: aktualizuj stan części
    Magazyn->>DB: 18. UPDATE część status=RESERVED
    Magazyn->>Cache: 19. Invalidate cache
    Cache-->>Magazyn: 20. OK
    Logistyka->>SMS: 21. Wyślij SMS potwierdzenie
    SMS-->>Warsztat: 22. Zarezerwowano (ETA: 24h)
    Logistyka-->>Gateway: 23. JSON response
    Gateway-->>Portal: 24. Pokaż potwierdzenie
    Portal-->>Warsztat: 25. Zarezerwowano!
```

---

## Diagram 4: Przepływ Danych - Skanowanie w Magazynie

```mermaid
sequenceDiagram
    participant Magazynier as Magazynier
    participant Scanner as Scanner QR
    participant API as API Gateway
    participant Magazyn as Magazyn Service
    participant DB as Database
    participant Cache as Redis
    participant Printer as Drukarka

    Magazynier->>Scanner: 1. Skanuje barcode części
    Scanner->>API: 2. POST /api/warehouse/scan
    API->>Magazyn: 3. Routing do Magazyn Service
    Magazyn->>DB: 4. SELECT część by barcode
    DB-->>Magazyn: 5. Część znaleziona (part_id: 12345)
    Magazyn->>DB: 6. UPDATE część status=PICKED
    Magazyn->>DB: 7. INSERT do tabeli warehouse_movements
    Magazyn->>Cache: 8. Aktualizuj licznik dostępnych
    Cache-->>Magazyn: 9. OK
    Magazyn->>API: 10. JSON OK response
    API->>Printer: 11. Wydrukuj etykietę wysyłkową
    Printer-->>Magazynier: 12. Etykieta gotowa
    Magazyn-->>Scanner: 13. Beep (sukces)

    Magazynier->>Magazynier: 14. Pakuje część do skrzynki
    Magazynier->>Magazynier: 15. Naklejam etykietę
    Magazynier->>Magazynier: 16. Kładę na rampie do wysyłki
```

---

## Diagram 5: GPS Tracking Dostawy (Real-time)

```mermaid
sequenceDiagram
    participant Kierowca as Kierowca
    participant Mobile as Aplikacja Mobilna
    participant API as API Gateway
    participant Logistyka as Logistyka Service
    participant Cache as Redis Cache
    participant WebSocket as WebSocket Server
    participant Warsztat as Warsztat (Portal)

    Kierowca->>Mobile: 1. Aplikacja uruchomiona (background)
    Mobile->>Mobile: 2. GPS lokalizacja co 30 sec
    Mobile->>API: 3. POST /api/logistics/gps-update<br/>{driver_id, lat, long, timestamp}
    API->>Logistyka: 4. Routing request
    Logistyka->>Cache: 5. SET driver:1:location
    Cache-->>Logistyka: 6. OK
    Logistyka->>WebSocket: 7. Emit LOCATION_UPDATE
    WebSocket->>Warsztat: 8. WebSocket push
    Warsztat->>Warsztat: 9. Aktualizuj mapę w real-time
    Warsztat-->>Warsztat: 10. Kierowca Marek 2 km stąd, ETA: 15 min

    Note over Kierowca,Warsztat: Co 30 sekund powtarza się proces
```

---

## Diagram 6: CI/CD Pipeline

```mermaid
graph LR
    A["Developer<br/>commit"] --> B["GitHub<br/>Repository"]
    B --> C["GitHub Actions<br/>Pipeline"]
    
    C --> D["Run Tests<br/>Jest, Mocha"]
    D -->|Pass| E["Code Quality<br/>SonarQube"]
    D -->|Fail| Z["Block Merge"]
    
    E --> F["Security Scan<br/>Snyk, OWASP"]
    F --> G["Build Docker Images"]
    
    G --> H["Push to ECR<br/>AWS Container Registry"]
    
    H --> I["Deploy to Staging<br/>Kubernetes"]
    I --> J["Integration Tests"]
    J -->|Fail| Z
    
    J -->|Pass| K["Manual Approval<br/>Code Review"]
    K -->|Approved| L["Deploy to Production"]
    K -->|Rejected| Z
    
    L --> M["Health Checks<br/>+ Monitoring"]
    M --> N["Prometheus +<br/>Grafana"]
    
    style Z fill:#ff6b6b
    style N fill:#51cf66
    style M fill:#51cf66
```

---

## Diagram 7: Skalowanie Poziome (Auto-scaling)

```mermaid
graph TB
    subgraph Before["PRZED - Bottleneck"]
        LOAD1["Ruch: 10k req/s"]
        POD1["Backend Pod 1<br/>CPU: 95% / Memory: 85%"]
        LOAD1 --> POD1
    end

    subgraph After["PO - Auto-scaling"]
        LOAD2["Ruch: 10k req/s"]
        LB["Load Balancer<br/>Round Robin"]
        POD2["Pod 1<br/>CPU: 60%"]
        POD3["Pod 2<br/>CPU: 58%"]
        POD4["Pod 3<br/>CPU: 62%"]
        LOAD2 --> LB
        LB --> POD2
        LB --> POD3
        LB --> POD4
    end

    subgraph HPA["Kubernetes HPA<br/>Min: 2, Max: 10<br/>Target CPU: 70%"]
    end

    Before -.->|Trigger| HPA
    HPA -.->|Create replicas| After

    style Before fill:#ffcccc
    style After fill:#ccffcc
    style HPA fill:#cce5ff
```

---

## Diagram 8: Warstwy Bezpieczeństwa

```mermaid
graph TB
    CLIENT["Klient"]
    
    L1["Layer 1: HTTPS/TLS<br/>Encryption in Transit"]
    L2["Layer 2: WAF + Rate Limiting<br/>DDoS Protection"]
    L3["Layer 3: Auth & RBAC<br/>JWT, OAuth2, 2FA"]
    L4["Layer 4: Business Logic<br/>Input Validation"]
    L5["Layer 5: Encryption at Rest<br/>AES-256, bcrypt"]
    L6["Layer 6: Auditing<br/>Centralized Logging (ELK)"]
    
    DB[("PostgreSQL<br/>Encrypted")]
    
    CLIENT --> L1
    L1 --> L2
    L2 --> L3
    L3 --> L4
    L4 --> L5
    L5 --> L6
    L6 --> DB
    
    style L1 fill:#ff9999
    style L2 fill:#ff9999
    style L3 fill:#ffcc99
    style L4 fill:#ffff99
    style L5 fill:#99ff99
    style L6 fill:#99ccff
    style DB fill:#cc99ff
```

---

## Diagram 9: Integracje Zewnętrzne

```mermaid
graph TB
    BeExpress["BeExpress<br/>Integracja Service"]
    
    PAYU["PayU<br/>- Płatności online<br/>- Faktury<br/>- Webhooks"]
    
    MAPS["Google Maps<br/>- Planowanie tras<br/>- ETA<br/>- Geolokacja"]
    
    TWILIO["Twilio<br/>- SMS<br/>- Email<br/>- Voice"]
    
    FIREBASE["Firebase<br/>- Push notifications<br/>- Analytics"]
    
    BANK["Bank API<br/>- BLIK<br/>- OAuth2"]
    
    BeExpress --> PAYU
    BeExpress --> MAPS
    BeExpress --> TWILIO
    BeExpress --> FIREBASE
    BeExpress --> BANK
    
    PAYU -.->|Webhook| BeExpress
    TWILIO -.->|Callback| BeExpress
    FIREBASE -.->|Analytics| BeExpress
```

---

## Diagram 10: Architektura Wdrażania (Environments)

```mermaid
graph LR
    DEV["DEVELOPMENT<br/>Laptop/Docker Compose<br/>localhost:3000"]
    
    STAGING["STAGING<br/>AWS EC2<br/>https://staging.beexpress.pl<br/>Production-like"]
    
    PROD["PRODUCTION<br/>AWS EKS<br/>https://app.beexpress.pl<br/>Multi-region"]
    
    DEV -->|Test Code| STAGING
    STAGING -->|Test Integrations| PROD
    STAGING -->|Manual Approval| PROD
    
    style DEV fill:#ffcccc
    style STAGING fill:#ffff99
    style PROD fill:#99ff99
```

---
