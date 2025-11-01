# 3D-PrintVerse – MVP Architekturübersicht

## 🔍 Oberfläche (eine zentrale UI)

* **Public Shop (Gast möglich):** Katalog → Konfiguration (Material, Farbe, Infill) → Warenkorb → Checkout.
* **Upload-Plattform (Login Pflicht):** Upload `.stl` → Konfiguration → "Zur Prüfung einreichen" → Angebot im Konto.
* **Konto-Bereich:** Angebote annehmen/ablehnen, Bestellstatus, Rechnungen, Kommunikation.
* **Admin-Dashboard:** Aufträge/Angebote, Dateien, Katalog, Materialien, SLAs/Zeitfenster, Mail-Templates.

## 📦 Backstage-Domänen (Services)

1. **Identity & Accounts** – Registrierung, Login, Rollen.
2. **Catalog & Config** – Produkte, Varianten, Material-/Farboptionen.
3. **Upload & Files** – Dateiupload, Format-Check (MVP: nur `.stl`), Ablage.
4. **Quotation (Angebote)** – Manuelle Prüfung, Preisfindung, Gültigkeit, Annahme/Ablehnung.
5. **Orders** – Bestellung (mehrere Dateien pro Order), Zahlung nach Annahme, Statuslauf.
6. **Production (später)** – Slicing, Drucker-Queue, Auslastung.
7. **Notifications** – E-Mails, Templates mit Platzhaltern.
8. **Shipping (später)** – Versandlabel, Tracking.
9. **Inventory (später)** – Materialien, Verfügbarkeiten.
10. **Analytics/Reporting (später)** – Durchlaufzeiten, Kostenübersicht.

## 🔹 Zentrale Objektlogik

* **User** → hat **Orders**.
* **Order** → umfasst 1..n **OrderItems** (Datei + Konfiguration).
* **Quote** → entsteht aus Upload+Konfig, wird **zur Order**, wenn Kunde annimmt.
* **Statuslauf:** Eingegangen → In Prüfung → Angebot bereit → In Produktion → QS → Versandt.

## 🔄 End-to-End-Flows

### A) Katalogkauf (MVP)

1. Produkt wählen → konfigurieren → Gast/Login → Checkout.
2. Admin setzt Status manuell (Produktion/QS/Versand).
3. Kunde erhält automatische Mails bei Statuswechseln.

### B) Upload-Kauf (MVP)

1. Login → `.stl` hochladen → konfigurieren → "Zur Prüfung".
2. Admin/Prüfer checkt Datei & Machbarkeit, trägt **Preis + Lieferfenster** ein → System sendet **Angebots-Mail**.
3. Kunde nimmt im Konto an → **Order entsteht**, Zahlung jetzt.
4. Produktion manuell (G-Code, Druckstart, QS, Versand), Status manuell gepflegt → Auto-Mails.

## 🔗 MVP vs. Wachstum

### MVP (manuell, klar, kontrolliert)

* Upload nur `.stl`.
* Angebot & Preis **manuell**.
* Statuswechsel **manuell**, Mails **automatisch**.
* Produktion & G-Code **manuell**.
* Zeitfenster **im Admin einstellbar**.

### V2–V3 (gezielte Automatisierung)

* **Auto-Validierung:** Geometrie-Checks.
* **Auto-Pricing:** Regeln (Volumen/Druckzeit/Material).
* **Auto-Status & Ereignisse:** Ereignisbasierte Workflows.
* **Slicing-Service:** G-Code-Erzeugung.
* **Printer-Queue:** Auftragszuweisung, Re-Print bei Fehler.
* **Shipping-Service:** Label, Tracking.
* **Inventory:** Materialverfügbarkeit.

### Zukunft (skalierbar)

* **Multi-Site / Farm-Routing.**
* **APIs/Partner-Integrationen.**
* **Community-/Marktplatz-Erweiterungen.**
* **Technik-Ansicht optional.**

## 🕵️ Prinzipien

* **Ein UI, viele Services.**
* **Schrittweise Automation.**
* **Mensch bleibt eingreifbar.**
* **Transparenz ohne Overload.**
* **Skalierbarkeit mitgedacht.**
