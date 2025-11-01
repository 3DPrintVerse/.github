# 🔁 End-to-End Flow – V1.0 (MVP)

## ⚙️ Überblick

Der folgende Ablauf beschreibt den vollständigen Prozess im MVP von **3D-PrintVerse**, inklusive beteiligter Services, Kommunikationsrichtungen und Rückmeldungen.

**Legende:**

* **HTTP (→)** = synchrone Kommunikation
* **RabbitMQ (⇢)** = asynchrone Kommunikation / Events

---

## 🧱 Basis-Komponenten

**Frontends:** Customer Web App, Admin Dashboard
**Edge:** API Gateway / BFF
**Core Services:** Identity & Auth, Catalog & Config, Upload, File, Quote, Order, Payment, Notification/Mail
**Infra:** RabbitMQ (Events), MinIO (Storage), PostgreSQL (DB)

---

## 🧩 A) Upload-Pfad – „Eigenes Modell bestellen“

### A1. Login & Session

1. Customer App → **API Gateway** → **Identity & Auth Service**
   → `POST /auth/login` → 200 (JWT/Session)

### A2. Upload & Dateiablage

2. Customer App → **API Gateway** → **Upload Service**
   → `POST /uploads` (Metadaten) → 201 (Upload-ID + presigned URL)
3. Customer App → **File Service (MinIO)**
   → `PUT` Upload-Datei → 200
4. **Upload Service** ⇢ RabbitMQ → Event `FileUploaded`

### A3. Light-Validierung & Verknüpfung

5. **Upload Service** prüft Dateityp & Größe → Status `UPLOADED` oder `REJECTED_LIGHT`
6. **Upload Service** → **Quote Service** → `POST /quotes` (upload_id, config) → 201 (`PENDING_REVIEW`)
7. **Quote Service** ⇢ RabbitMQ → `QuoteCreated`

### A4. Manuelle Prüfung & Angebot

8. Admin Dashboard → **Quote Service** → `GET /quotes?status=PENDING_REVIEW`
9. Admin Dashboard → **File Service** → `GET /files/{file_key}` → temporäre Download-URL
10. Admin vergibt Preis & Lieferzeit → `PATCH /quotes/{id}` (price, eta, status=READY) → 200
11. **Quote Service** ⇢ RabbitMQ → `QuoteReady`
12. **Notification Service** lauscht → sendet Angebots-Mail über SMTP → `EmailSent`

### A5. Angebotsannahme & Zahlung

13. Customer App → **Quote Service** → `POST /quotes/{id}/accept` → 200
14. **Quote Service** ⇢ RabbitMQ → `QuoteAccepted`
15. **Order Service** empfängt → erzeugt Order → ruft **Payment Service**: `POST /payments` → 200 (`CAPTURED`)
16. **Order Service** ⇢ RabbitMQ → `OrderCreated`
17. **Notification Service** sendet Bestellbestätigung → SMTP OK

### A6. Produktion (manuell im MVP)

18. Admin Dashboard → **Order Service** → `PATCH /orders/{id}/status` (`IN_REVIEW` → `IN_PRODUCTION`) → 200 ⇢ `OrderStatusChanged`
19. Mitarbeiter erstellt G-Code manuell, startet Druck (außerhalb System)
20. Nach QS → `PATCH /orders/{id}/status` (`IN_QA` → `READY_TO_SHIP` → `SHIPPED`) → Events → Mails über Notification

---

## 🛒 B) Katalog-Pfad – „Shop-Produkt bestellen“

### B1. Browsing & Checkout

1. Customer App → **Catalog & Config Service** → `GET /catalog` → 200
2. Customer App → **Order Service** über API Gateway → `POST /checkout` → 201 (Order erstellt)
3. **Order Service** → **Payment Service** → `POST /payments/charge` → 200 (`CAPTURED`)
4. **Order Service** ⇢ RabbitMQ → `OrderCreated`
5. **Notification Service** → Bestellbestätigung
6. Admin pflegt Status manuell → Events & automatische Mails

---

## ✉️ C) Status-Events & Benachrichtigungen

* `OrderStatusChanged`: `RECEIVED → IN_REVIEW → IN_PRODUCTION → IN_QA → READY_TO_SHIP → SHIPPED`
* `QuoteReady`, `QuoteAccepted`, `EmailSent`: je nach Workflow-Schritt
* **Notification Service** reagiert asynchron auf Events → Mails
* Frontends ziehen Status synchron via REST

---

## 🔄 D) Kommunikationsmuster

* **Synchron (HTTP):** Frontends ↔ API Gateway ↔ Domain Services
* **Asynchron (RabbitMQ):** Domain Events, Statusänderungen, Benachrichtigungen

---

## ⚠️ E) Fehlerfälle & Rückfälle

* **File-Upload fehlgeschlagen:** kein `FileUploaded`-Event → UI-Fehleranzeige
* **Mailversand fehlgeschlagen:** Notification retry in Queue
* **Zahlung fehlschlägt:** Payment → 4xx → Quote bleibt aktiv
* **Manuelle Prüfung negativ:** Quote → `REJECTED` → Ablehnungsmail
* **Status vergessen:** keine Events → keine Mails → Dashboard-Warnung

---

## 🧾 F) Beispiel-Sequenz (Upload-Pfad)

```text
Customer → API GW → Upload: POST /uploads           → 201 (upload_id, presigned_url)
Customer → FileSvc(MinIO): PUT file                 → 200
Upload ⇢ MQ: FileUploaded(upload_id, file_key)

API GW → Quote: POST /quotes (upload_id, config)    → 201 (quote_id, PENDING_REVIEW)
Quote ⇢ MQ: QuoteCreated

Admin → Quote: PATCH /{id} (price, ETA, READY)      → 200
Quote ⇢ MQ: QuoteReady
Notification ⇠ MQ → SMTP: send(QuoteReady)          → ok

Customer → Quote: POST /{id}/accept                 → 200
Quote ⇢ MQ: QuoteAccepted
Order ⇠ MQ: create(order_id) → Payment: charge      → 200
Order ⇢ MQ: OrderCreated
Notification ⇠ MQ → SMTP: send(OrderConfirmation)   → ok

Admin → Order: PATCH status(IN_PRODUCTION)          → 200 → MQ: OrderStatusChanged
Admin → Order: PATCH status(IN_QA)                  → 200 → MQ: OrderStatusChanged
Admin → Order: PATCH status(SHIPPED)                → 200 → MQ: OrderStatusChanged
Notification ⇠ MQ → SMTP: send(StatusUpdates)       → ok
```
