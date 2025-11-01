# 🧩 3D-PrintVerse

---

## 📘 Projektbeschreibung

**3D-PrintVerse** ist ein modulares System zur Verwaltung, Automatisierung und Durchführung von 3D-Druck-Aufträgen.
Das Ziel ist die Entwicklung einer skalierbaren Softwareplattform mit Microservice-Architektur, bestehend aus:

* Frontends für Kunden und Administratoren
* API Gateway / BFF als zentrales Entry-Point
* Domain-spezifischen Backend-Services
* Asynchroner Kommunikation über RabbitMQ
* Persistenz über PostgreSQL und MinIO Object Storage

---

## 🏗️ Architekturüberblick (MVP – V1.0)

### Hauptkomponenten

| Typ          | Komponente          | Beschreibung                                |
|--------------|---------------------|---------------------------------------------|
| **Frontend** | Customer Web App    | Shop + Upload-Funktion                      |
| **Frontend** | Admin Dashboard     | Verwaltung, manuelle Prüfung & Statuspflege |
| **Gateway**  | API Gateway / BFF   | Einheitlicher Zugang zu allen Services      |
| **Service**  | Identity & Auth     | Authentifizierung, Rollen, Sessions         |
| **Service**  | Catalog & Config    | Produktkatalog & Materialkonfiguration      |
| **Service**  | Upload              | Upload-Management & Validierung             |
| **Service**  | File (MinIO)        | Dateiablage & Zugriff auf 3D-Modelle        |
| **Service**  | Quote               | Angebotserstellung & Verwaltung             |
| **Service**  | Order               | Bestellungen & Statusverlauf                |
| **Service**  | Payment             | Zahlungsabwicklung (Basis)                  |
| **Service**  | Notification / Mail | Mailversand über RabbitMQ                   |
| **Infra**    | RabbitMQ            | Event-Bus zur asynchronen Kommunikation     |
| **Infra**    | MariaDB             | Zentrale Datenbank                          |
| **Infra**    | MinIO               | Object Storage für 3D-Dateien               |

---

## ⚙️ Entwicklungsumgebung

### Voraussetzungen

* Docker / Docker Compose
* Node.js & npm / pnpm
* Java mit SpringBoot (für Backend-Services)
* RabbitMQ
* MariaDB
* MinIO

### Lokales Setup

```bash
# 1. Repository klonen
git clone <INTERNAL_REPO_URL> 

# 2. Services starten (Docker)
docker compose up -d

# 3. Frontends starten (Dev-Modus)
cd frontend-customer && npm run dev
cd frontend-admin && npm run dev
```

### Umgebungsvariablen

Jeder Service hat eine `.env`-Datei, z. B.:

```env
POSTGRES_URL=postgresql://user:pass@db:5432/printverse
MINIO_URL=http://minio:9000
RABBITMQ_URL=amqp://guest:guest@rabbitmq:5672/
JWT_SECRET=<PLACEHOLDER>
SMTP_SERVER=<PLACEHOLDER>
```

---

## 🔁 Kommunikationsprinzipien

* **HTTP →** synchron über das API Gateway
* **RabbitMQ ⇢** asynchron für Statusänderungen & Events
* **Mail-Service ⇢** Event-Driven (z. B. `QuoteReady`, `OrderStatusChanged`)

---

## 🧪 Tests & Qualitätssicherung

```bash
# Unit Tests
npm run test

# Integration Tests
docker compose -f docker-compose.test.yml up --build
```

---

## 🚀 Deployment

### Staging

```bash
docker compose -f docker-compose.staging.yml up -d
```

### Production (Beispiel)

```bash
kubectl apply -f k8s/
```

> Platzhalter: `TODO: CI/CD Workflow beschreiben (GitHub Actions, ArgoCD, Jenkins etc.)`

---

## 📅 Versionierung

| Version        | Inhalt                                                             |
|----------------|--------------------------------------------------------------------|
| **V1.0 (MVP)** | Manuelle Prozesse, Kernlogik & Upload-Funktion                     |
| **V2.0+**      | Erste Automatisierungsschritte (Validierung, Preisberechnung etc.) |

---

> 🛠️ *Internal Developer Documentation – last updated: 01.11.2025*
