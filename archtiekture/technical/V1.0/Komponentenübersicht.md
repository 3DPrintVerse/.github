# Komponentenübersicht V1.0 (MVP)

## 1. Frontends

### Customer Web App (Shop + Upload)

* Katalog & Produktkonfiguration (Material, Farbe, Infill)
* Login/Registrierung
* STL-Upload (nur `.stl`)
* Angebote einsehen/annehmen, Bestellstatus

### Admin Dashboard

* Aufträge / Angebote / Bestellungen
* Manuelle Prüfungen & Preisvergabe
* Statuspflege (Prüfung → Angebot → Produktion → Versand)
* Produkte / Materialien / SLAs pflegen
* Mail-Templates verwalten

---

## 2. Edge / Access

### API Gateway / BFF

* Einheitliche API für beide Frontends
* Auth-Zwischenschicht, Ratenbegrenzung, Request Logging
* Weiterleitung an die Domain-Services

---

## 3. Core Domain Services

### Identity & Auth Service

* Nutzerkonten, Rollen (Admin / Prüfer / Produktion / Support / Kunde)
* Tokens / Sessions

### Catalog & Config Service

* Produkte, Varianten, Material- und Farboptionen, SLA-Zeitfenster

### Upload Service

* Annahme von Upload-Metadaten, Validierungsstatus „light“ (Dateityp)
* Übergabe der Datei an den File-Service, Referenzen zurück an Order / Quote

### File Service (MinIO)

* Speichern / Abrufen der Dateien im Object Storage
* Signierte / temporäre URLs für andere Services

### Quote Service (Angebote)

* Anlage / Verwaltung von Angeboten (Preis & Lieferfenster werden manuell im Admin gesetzt)
* Gültigkeit / Fristen, Annahme / Ablehnung

### Order Service

* Bestellung aus angenommenem Angebot erzeugen
* 1..n Dateien pro Order (OrderItems)
* Statusverwaltung (manuell im MVP)

### Payment Service (MVP-light)

* Zahlung erst nach Angebotsannahme
* Übergabe an PSP (z. B. Checkout / Charge), Rückmeldung an Order

### Notification / Mail Service

* E-Mail-Versand (Bestätigung, Angebot bereit, Statuswechsel)
* RabbitMQ Listener (z. B. QuoteReady, OrderStatusChanged)
* Template-Engine
