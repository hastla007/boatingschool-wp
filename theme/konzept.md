WooCommerce B2B Shop für Bootsschulen

Gesamtkonzept für den B2B-Verkauf und die Verwaltung von Lernplattform-Zugangscodes an Bootsschulen in AT, DE, CH und ES

Fachliche und technische Grundlage für Produktplanung, Umsetzung und Go-live

Kurzempfehlung

WooCommerce soll B2B-Bestellungen, Zahlung, Rechnungen und Händlerkonten abwickeln. Die Lernplattform bleibt führend für Schulen, Kurse, Zugangscodes und Einlösungen. Ein eigens entwickeltes WooCommerce-Plugin verbindet beide Systeme über eine abgesicherte, idempotente Server-zu-Server-API. Nach bestätigter Zahlung erhält die Schule die Codes als ZIP-Datei zum Download und/oder per E-Mail. Optional werden dieselben Codes direkt in den Codebestand des verknüpften Schuladmin-Kontos auf der Bootsschulplattform übertragen.

So bleibt die bestehende Coupon- und Entitlement-Logik erhalten und der Shop muss keine Kurszugänge selbst erfinden. Das Repository dokumentiert derzeit ausdrücklich keinen WooCommerce-Adapter und keine Zahlungsabwicklung; beides ist die noch zu entwickelnde Ergänzung.

1. Ziel, Beteiligte und Rahmen

Ziel ist ein zentraler Shop, über den Bootsschulen Kurszugänge in Kontingenten kaufen. Die Schule zahlt als Geschäftskunde und erhält je gebuchtem Kurs eine definierte Anzahl digitaler Zugangscodes. Die Codes werden von Schülern in der vorhandenen Lernplattform oder mobilen App eingelöst. Der Shop verkauft keine Lerninhalte und führt keine Prüfungssimulation aus.

Beteiligter

Aufgabe

Plattformbetreiber / Superadmin

Kurse und Preise pflegen, Schulen freischalten, Bestellungen und Code-Lieferungen überwachen, Support und Erstattungen bearbeiten.

Bootsschule / Einkäufer

Geschäftskonto verwalten, Kurskontingente bestellen, Rechnung und Bestellstatus einsehen, Codes abrufen.

Schuladmin / Ausbilder

Codes intern verteilen, Lernende verwalten und deren Fortschritt in der Plattform betreuen.

Lernender

Konto anlegen oder nutzen, Code einlösen, den zugeordneten Kurs lernen und Prüfungen simulieren.

Zahlungsanbieter

Zahlung autorisieren, Statusänderungen und Rückzahlungen an WooCommerce melden.

Leitplanken

Die Lernplattform bleibt die einzige Quelle für Kurszugang und Codegültigkeit.

WordPress speichert Auftragsdaten, aber keine App-Passwörter und keine direkte Datenbankverbindung zur Lernplattform.

Jede Bestellung ist einer verifizierten Bootsschule und einem Kurs zugeordnet; Codes dürfen nicht unbeabsichtigt zwischen Mandanten wechseln.

Digitale Codes werden erst nach einem belastbaren Zahlungserfolg geliefert. Für freigegebene Rechnungskauf-Kunden kann eine bewusst konfigurierte Regel gelten.

Ausgabe und Rückabwicklung müssen nachvollziehbar, wiederholbar und auditierbar sein.

2. Repository-Analyse: vorhandener Stand

Das öffentliche Repository hastla007/boatingschool beschreibt eine mandantenfähige Laravel-12-Anwendung mit PostgreSQL. Jede Bootsschule ist ein Tenant. App-Daten sind zusätzlich durch PostgreSQL Row-Level Security getrennt. Kurse, Fragen und Lerninhalte liegen zentral; Lernfortschritt, Teilnehmer, Coupons und Zugänge sind mandantenbezogen.

Im App-Code bereits vorhanden

Bedeutung für den Shop

Zentrale Kursdefinitionen, Coupon- und Entitlement-Modelle

Ein Code kann einen Kurs, ein Produkt oder eine Verlängerung freischalten; Einlösung erzeugt Zugang bzw. Kaufnachweis.

Coupon-Zuordnung zu tenant_id

Ein Code kann beim Erstellen an eine Bootsschule gebunden werden. NULL steht für noch nicht zugeordnet bzw. global einlösbar.

Code-Status und Gültigkeit

Codes kennen eingelöst, gesperrt und abgelaufen; der Code ist einmalig einlösbar.

Batch-Label und Admin-Codefunktionen

Die Plattform kann Code-Batches verwalten, exportieren, zuweisen und auditieren.

Kurskatalog und Webshop-Link pro Kurs

Die Lernenden-App kann Kursangebote und einen externen Shop-Link anzeigen.

Mobile API v1

Lernende melden sich an, wählen ihre Bootsschule über X-Tenant-Slug und lösen Codes über POST /api/v1/coupons/redeem ein.

Keine Commerce-Integration

README nennt den echten WooCommerce-/Commerce-Adapter und Zahlungsabwicklung ausdrücklich als ausgeklammert.

Im README sind 14 Kursdefinitionen beschrieben: SBF See, SBF Binnen, SBF Binnen unter Segel, SBF See & Binnen, drei Ergänzungskurse, SRC, UBI, SRC & UBI, UBI-Ergänzung, SKN, FKN und FKN-zu-SKN-Ergänzung. Die FKN-zu-SKN-Ergänzung ist laut README noch als Entwurf markiert und kann noch keine Prüfung starten. Die Shop-Produkte dürfen daher nicht allein aus dem Katalog automatisch live geschaltet werden: Verkaufbarkeit, Inhalt, Preis und Gültigkeit brauchen je Kurs eine fachliche Freigabe.

Die App-Coupon-Tabelle enthält bereits Kurs-/Produktbezug, tenant_id, Batch-Bezeichnung, Einlösedaten, Ersteller, Ablauf- und Sperrzeitpunkt sowie Verlängerungskennzeichen. Das ist eine gute Grundlage für Kontingentcodes. Ein Shop-Auftrag, Käuferorganisation, WooCommerce-Bestell-ID und eine dedizierte Commerce-API sind im README nicht als bestehende Funktionen beschrieben. Die folgenden Teile sind Empfehlungen für die Neuentwicklung.

Verifizierte Quellen

Repository und Architektur: https://github.com/hastla007/boatingschool (README.md, Standardbranch main; geprüft am 12.11.2026).

Mobile-API-Dokumentation: docs/api/README.md und docs/api/openapi-mobile-v1.yaml im selben Repository.

Relevante Modell- und Ablaufdateien: Coupon.php, Entitlement.php, CouponRedemptionService.php, CouponController.php, CouponController (Admin) und CourseSeeder.php.

3. Zielprozess vom Shop bis zur Freischaltung

Schritt

Ablauf und Systemverantwortung

1. Kontoantrag

Bootsschule erstellt Geschäftskonto mit Firma, Kontakt, Rechnungsadresse und optional UID/VAT-ID. Shop prüft E-Mail und doppelte Organisationen; B2B-Zugang kann bis zur Freigabe eingeschränkt sein.

2. Zuordnung zur Schule

Administrativer oder automatisierter Verifikationsprozess verbindet das WooCommerce-Konto mit genau einem Plattform-Tenant. Zuordnung über UUID/Partner-ID, niemals über einen frei eingegebenen Slug.

3. Produktwahl

Käufer wählt Kurs, Code-Menge bzw. Staffel und sieht Brutto-/Nettopreis, Steuerhinweis, Codegültigkeit und Einlösebedingungen.

4. Checkout

WooCommerce erstellt Bestellung mit eindeutigen Positionen, Tenant-Referenz, Kurscode, Menge, Preis, Steuersatz, Zahlungsart und Rechnungsdaten.

5. Zahlung bestätigt

Bestellung wechselt auf bezahlten/lieferbaren Status. Der Integrationsauftrag wird in einer Outbox/Queue persistiert und mit eindeutiger Ereignis-ID an die Plattform gesendet.

6. Codes erzeugt

Plattform prüft Tenant, Kurs, Status, Menge und Idempotenzschlüssel. Sie erzeugt Codes in einer Datenbanktransaktion und bindet jeden Code an den Tenant und die bestellte Kursdefinition.

7. Lieferung

WooCommerce speichert die Bestell-/Batch-Referenz, Anzahl und Status. Codewerte werden nur für berechtigte Käufer im Kundenkonto angezeigt bzw. als CSV heruntergeladen. E-Mail informiert über Verfügbarkeit und enthält einen sicheren Konto-Link.

8. Nutzung

Schuladmin verteilt Bestand oder weist Codes Lernenden zu. Lernender löst den Code in Web-App oder mobiler App ein; Plattform erstellt das Entitlement.

9. Rückabwicklung

Bei Storno vor Codeerzeugung wird nichts ausgegeben. Bei Erstattung nach Lieferung werden ungenutzte Codes gesperrt. Bereits eingelöste Codes benötigen einen Support-/Kulanzprozess und dürfen nicht automatisch den Schülerzugang löschen.

Wiederholte Webhooks und Netzwerkabbrüche dürfen keine doppelten Codes oder zweite Lieferung erzeugen. Der Shop muss nach Timeout mit derselben Idempotenz-ID erneut anfragen und den Status der ursprünglichen Lieferung abrufen können.

4. Produkt- und Preismodell

Empfohlener MVP-Produktzuschnitt

Ein WooCommerce-Produkt je verkaufbarem Kurs und Code-Menge oder ein Kursprodukt mit Mengenstaffeln. Das App-Kurskürzel (z. B. SBF-SEE) ist ein unveränderlicher Integrationsschlüssel; WordPress-Produkt-ID und sichtbarer Name sind nicht die fachliche Identität.

Code-Kontingente als Stückzahl. Mindestmenge und Preisstaffeln können je Kurs gesetzt werden. Darstellung: Nettopreis, Steuer, Gesamtpreis sowie Preis pro Schülerzugang.

Keine Abos im MVP. Falls später laufende Lizenzen gewünscht sind, müssen Verlängerungs-, Kündigungs- und Zugangslaufzeit getrennt spezifiziert werden.

Unterscheidung zwischen Erstzugang, Verlängerung und ergänzendem Kurs. Die App kennt bereits Verlängerungscodes; im Shop müssen sie als eigene Produkte bzw. regelbasierte Produktvarianten sichtbar sein.

Optionales ergänzendes Produkt (z. B. Fahrstunden/Material) nur dann im ersten Release, wenn es tatsächlich verkauft und in der Plattform verarbeitet werden soll.

Katalog- und Freigabefelder

Feld

Regel

course_code

Stabiler Schlüssel aus der Plattform; Pflichtfeld, nicht aus Namen ableiten.

Verkaufbar

Nur wenn Kurs veröffentlicht und fachlich für Verkauf freigegeben ist.

Gültigkeit

Explizite Laufzeit in Tagen/Monaten oder unbegrenzt; mit Startregel und Verlängerungslogik.

Mindest-/Staffelmenge

Je Kurs definierbar; vor Checkout serverseitig validieren.

Preis und Steuerklasse

WooCommerce-Katalogwert; Netto-/Bruttoanzeige abhängig von Steuerkonfiguration und Kundensitz.

Storno-/Erstattungsregel

Vor Codeerzeugung automatisierbar; nach Codeausgabe abhängig von Status jedes einzelnen Codes.

Produktstatus-Synchronisierung

Deaktivierung im Shop darf historische Bestellungen und bestehende Zugänge nicht löschen.

5. Konten, Rollen und Schulbestand

Der Shop sollte Geschäftskundenkonten unterstützen und die App-Schulidentität eindeutig abbilden. Für den Start genügt ein Shop-Konto pro Schule mit mehreren berechtigten Ansprechpartnern, wenn die bestehende App-Rollenstruktur das unterstützt. Ansonsten verwaltet WooCommerce Einkäufer und der App-Schuladmin erhält seine Codes dort, ohne dass Identitäten unbemerkt zusammengeführt werden.

Rolle

Muss können

Darf nicht

Shop-Käufer

Bestellungen, Rechnungen, Lieferstatus, Code-Downloads und Ansprechpartner sehen.

Codes anderer Schulen oder Käuferkonten einsehen.

Schuladmin in der App

Codebestand der eigenen Schule sehen, zuweisen, Bestand exportieren und Einlösungen nachvollziehen.

Kursinhalte oder andere Tenant-Daten administrieren.

Plattform-Support

Bestellung und Übergabe anhand IDs prüfen; bei Bedarf Codes sperren/ersetzen.

Klartextcodes ohne Supportgrund einsehen.

Shop-Administrator

Produkte, Preise, Firmenkonten, Bestellungen und Rückerstattungen verwalten.

App-Zugänge durch direkte Datenbankänderung erzeugen.

Bestandsübersicht je Kurs sollte mindestens verfügbar, zugewiesen, eingelöst, abgelaufen, gesperrt und zur Lieferung ausstehend zeigen. Ein Schülername wird nur dann in WooCommerce gespiegelt, wenn er für Abrechnung oder Support tatsächlich benötigt wird. Die App ist die führende Stelle für persönliche Lernerdaten.

6. Integration und technische Architektur

Empfohlen wird ein kleines eigenes WooCommerce-Plugin plus eine Commerce-API in der Laravel-Plattform. Die Systeme sprechen ausschließlich über HTTPS mit getrennten Systemzugängen. WooCommerce REST API allein reicht nicht: Sie verwaltet Shopdaten, aber nicht atomar und sicher die App-Coupon- und Tenant-Regeln.

Baustein

Aufgabe

WooCommerce

B2B-Katalog, Checkout, Steuern, Rechnungen, Zahlungen, Firmenkonten, Bestell- und Rückerstattungsstatus.

Eigenes WordPress-Plugin

Kurs-Mappings, Tenant-Referenz, Lieferauftrag, Outbox/Retry, Statusanzeige, geschützter Codebestand, Admin-Überwachung.

Laravel-Commerce-API

Tenant-/Kursprüfung, Coupon-Erzeugung, Code-Batch und Code-Status, Idempotenz, Audit-Log, Abfrage und Sperrung ungenutzter Codes.

Bestehende App

Einlösung, Entitlement-Erzeugung, Kurszugang, Schuladmin-Zuweisung und mobile Codeeinlösung.

Queue und Monitoring

Asynchrone Zustellung, Wiederholungen, Dead-letter/Fehlerwarteschlange, Alarm bei ausbleibender Lieferung.

API-Vorschlag

Operation

Zweck / Schutz

POST /api/commerce/v1/code-batches

Batch aus einer bezahlten Bestellung anlegen. Erfordert tenant_id, course_code, quantity, order_reference und Idempotency-Key. Rückgabe: batch_id, Stückzahl, einmalig abrufbare Codes oder geschützter Downloadbezug.

GET /api/commerce/v1/code-batches/{id}

Lieferstatus, Fehler, Kurs, Gesamtmenge und Statussummen abfragen; keine erneute Codeerzeugung.

GET /api/commerce/v1/code-batches/{id}/codes

Berechtigter einmaliger Export oder paginierter Abruf; Zugriffe auditieren, keine Codes in Logs.

POST /api/commerce/v1/code-batches/{id}/void-unused

Nur ungenutzte Codes sperren; idempotent und mit Grund/Bestellreferenz.

GET /api/commerce/v1/catalog

Optionaler Katalogexport mit freigegebenen Kursen, Codes, Verkaufbarkeit und Gültigkeitsparametern.

Sicherheitsregeln

OAuth2 Client Credentials oder signierte HMAC-Anfragen mit Rotation, getrennten Staging-/Produktivschlüsseln und eng begrenzten Scopes.

Tenant-Zuordnung ausschließlich aus verifizierter Shop-zu-Tenant-Verknüpfung ableiten; API-Anfrage kann sie nicht beliebig festlegen.

Idempotenzschlüssel pro Bestellposition und Liefergeneration; gleicher Schlüssel und gleicher Payload liefert denselben Batch zurück. Payload-Änderung mit gleichem Schlüssel wird abgelehnt.

Rate-Limits, Eingabevalidierung, TLS, Zeitlimits und eindeutige Request-/Correlation-IDs.

Keine direkten DB-Zugriffe aus WordPress. Keine Secret- oder Codewerte in PHP-Fehlerlogs, Webhook-URLs, Analytics oder E-Mails.

Code-Downloads nur nach erneuter Berechtigungsprüfung; Ereignisprotokoll mit Benutzer, Zeitpunkt, Bestell-ID und Downloadaktion.

7. Zahlung, Rechnung, Steuern und Rückabwicklung

Zahlungsarten und Steuerlogik werden nach Geschäftssitz, Zielmarkt und Buchhaltungsprozess festgelegt. Für einen EU-weiten B2B-Verkauf sind insbesondere UID-Prüfung, Nettopreis-/Steuerdarstellung, Rechnungserstellung und die fachliche Behandlung digitaler Leistungen mit Steuerberatung zu bestätigen. Das Konzept legt keine konkrete steuerliche Rechtsfolge fest.

Onlinezahlung: Liefertrigger erst nach vertrauenswürdig bestätigtem Zahlungsstatus; Browser-Rückkehr zur Danke-Seite ist kein Zahlungsnachweis.

Rechnungskauf: nur nach manueller Freigabe bzw. konfiguriertem Kreditlimit; festlegen, ob Codes nach Auftrag, Versand der Rechnung oder Zahlung bereitgestellt werden.

Storno vor Lieferung: Bestellung stornieren und Integrationsauftrag stoppen.

Teilrückerstattung: nur für nicht gelieferte Mengen oder ungenutzte Codes gemäß Geschäftsregel.

Volle Erstattung nach Lieferung: ungenutzte Codes über die Plattform sperren; eingelöste Codes bleiben zunächst aktiv und müssen in einem dokumentierten Supportprozess behandelt werden.

Chargeback: Bestellung als Streitfall markieren, betroffene Codes ermitteln und Missbrauchs-/Kundenprüfung starten; keine automatische Sperre bereits eingelöster Lernzugänge ohne definierte Regel.

Rechnungskorrektur und Gutschrift müssen mit derselben Bestellung und Batch-ID referenziert sein.

8. Datenschutz, Sicherheit und Betrieb

Datensparsamkeit: Shop benötigt Firmen-/Rechnungs- und Ansprechpartnerdaten; App benötigt Lernendenkonten und Lernfortschritt. Keine vollständigen Lernprofile in WooCommerce kopieren.

Auftragsverarbeitung, Rollen, Löschfristen, Auskunft/Löschung, Zahlungsdaten und Analytics für die konkrete Hosting- und Plugin-Auswahl dokumentieren.

Zahlungsdaten werden beim Zahlungsanbieter verarbeitet; WooCommerce speichert Token/Referenzen statt Kartenprüfdaten.

Backups für WordPress und Laravel getrennt, verschlüsselt, regelmäßig wiederhergestellt getestet; Secrets extern verwalten und rotieren.

Staging mit Test-Zahlungsanbieter und separatem App-Tenant; keine realen Codes oder Schülerdaten aus Produktion.

Überwachung auf bezahlte, aber unversorgte Aufträge, doppelte Webhooks, API-Fehler, fehlgeschlagene Downloads und auffällige Code-Downloads.

Abstimmung der Code-Laufzeit: Es gibt bereits einen Scheduler für Coupon-Ablauf und Entitlement-Erinnerungen. Die Shop-Anzeige muss den App-Status abfragen und nicht eigene Laufzeitlogik erfinden.

Vor Go-live Abgleich von WooCommerce-/WordPress-/PHP-/Payment-Plugin-Versionen und Wartungs-/Updateprozess; konkrete Versionen erst beim Implementierungszeitpunkt festlegen.

9. MVP-Abgrenzung und spätere Ausbaustufen

MVP erforderlich

Später möglich

Geschäftskonto, Freigabe und Tenant-Verknüpfung

Self-service-Onboarding mit automatisierter Verifikation.

Kursprodukte, Mengenstaffeln, B2B-Checkout, Zahlungs-/Rechnungsablauf

Individuelle Preislisten und Rahmenverträge.

Bezahlte Bestellung löst einmalige Codebatch-Erstellung aus

Monatsabrechnung oder echtes Prepaid-Guthaben.

Codebestand, geschützter CSV-Export, Bestellhistorie und Status

Schulportal direkt in der App oder Single Sign-on.

Webhook-Retry, Idempotenz, Monitoring, Sperrung ungenutzter Codes

Partner-API für Großkunden und externe Schulverwaltung.

Manuelle Rückerstattungs-/Supportregeln

Automatisierte Teilstorno- und Kulanzworkflows.

Nicht im MVP: In-App-Käufe, automatische Mandantenerstellung ohne Verifikation, ein zweites Entitlement-System in WordPress, Einzelschüler-Direktverkauf (B2C), Abonnements oder automatisierte Content-/Kursfreigabe ohne fachliche Prüfung.

10. Umsetzungsplan

Phase

Arbeitsergebnis

Abnahmekriterium

0. Geschäftsregeln

Kurse, Preise, Staffelungen, Gültigkeit, Steuerzielmärkte, Zahlungsarten, Rückerstattungen und Rollen entschieden.

Freigegebene Produkt-/Regelmatrix; offene Steuerfragen geklärt.

1. Plattform-API

Commerce-Authentifizierung, Katalog, Batch-Erstellung, Idempotenz, Status, Code-Export und Sperrung.

Tests zeigen keinen doppelten Batch bei Wiederholung; Tenant-/Kursfehler werden abgewiesen.

2. Shop-Grundlage

WooCommerce-Konfiguration, B2B-Konten, Firmenfelder, Zahlungsanbieter, Rechnungen und Kursprodukte.

Staging-Bestellung erzeugt korrekte Positionen und Steuer-/Rechnungsdaten.

3. WooCommerce-Plugin

Bestell-zu-Tenant-Zuordnung, Liefer-Outbox, Retry, Statusseite und Code-Download.

Bezahlte Bestellung führt pro Position genau zu einem App-Batch; Fehler bleiben sichtbar und wiederholbar.

4. App-Schuladmin-Ablauf

Codes sind der Schule zugeordnet, können verteilt/zugewiesen und eingelöst werden.

Testperson löst Code ein; passender Kurszugang erscheint nur in korrektem Tenant.

5. Rückabwicklung & Sicherheit

Storno, Rückerstattung, Code-Sperrung, Rollen, Logs, Rate-Limits, Backup und Monitoring.

Ungenutzter Code kann gesperrt werden; eingelöster Code wird kontrolliert behandelt.

6. Pilot und Go-live

Pilot mit zwei bis drei Bootsschulen, Supportablauf, Produktivschlüssel, Datenschutz-/Rechnungsfreigabe.

End-to-end-Abnahme inkl. Zahlung, Codeabruf, Einlösung und Supportfall.

11. Abnahmekriterien für den Gesamtprozess

Ein verifizierter Schulshopkunde kann nur Produkte für die zugeordnete Bootsschule bestellen.

Je bezahlter Bestellposition wird exakt die bestellte Anzahl gültiger Codes dem korrekten Tenant zugeordnet.

Mehrfachzustellung desselben Zahlungsereignisses erzeugt keine doppelten Codes.

Bei fehlender oder fehlerhafter Plattformantwort bleibt die Bestellung als ausstehend/Fehler erkennbar; Wiederholung ist sicher.

Ein Code kann höchstens einmal eingelöst werden; die bestehende App-Logik und Tenant-Isolation bleiben wirksam.

Unberechtigte Benutzer können weder Codebestand noch Downloads einer fremden Schule einsehen.

Storno vor Lieferung erzeugt keinen Code; Erstattung nach Lieferung sperrt ungenutzte Codes und protokolliert den Vorgang.

Support kann einen Vorgang anhand Woo-Bestellnummer, App-Batch-ID und Request-ID ohne Klartextcode nachvollziehen.

12. Entscheidungen vor dem Start

Entscheidung

Vorschlag für den Start

Verkaufsgebiet

AT, DE, CH und ES; ausschließlich B2B. Keine privaten Käuferkonten und kein B2C-Checkout.

Zahlungsarten

Empfehlung: Online-Kartenzahlung plus Rechnung/Banküberweisung nur für manuell freigegebene Schulen. Codes erst nach verlässlich bestätigtem Zahlungseingang; konkrete Auswahl bleibt vor Start zu bestätigen.

Käuferzugang

Der Schuladmin ist zugleich der WP-Shop-Administrator. Bestehende Konten werden sicher miteinander verknüpft; keine geteilten Passwörter oder automatische Zuordnung allein über E-Mail.

Produktmengen

Alle verfügbaren Kurse kommen als Produktkandidaten in den Katalog. Preise sind noch offen; Mindestmengen und Staffelungen sind ebenfalls festzulegen.

Code-Laufzeit

Softwareprüfung: Ablaufdatum eines Codes ist optional je Coupon; Kurszugangsdauer nach Einlösung ist derzeit je Bootsschule konfiguriert, nicht je Kurs. Per-Kurs-Laufzeiten sind aktuell nicht als Kursstammdaten vorhanden.

Codeausgabe

ZIP-Download im WooCommerce-Konto und optional ZIP per E-Mail; zusätzlich kann derselbe Codebestand in das verknüpfte Bootsschul-Plattformkonto übernommen werden.

Schuladmin-Funktionen

Eine Person nutzt Shop-Admin und Plattform-Schuladmin. Kontoverknüpfung muss explizit und widerrufbar sein; App-Konto bleibt führend für Codebestand und Zuweisung.

B2C-Verkäufe

Nicht Teil dieses Vorhabens; bei Bedarf als separates Produkt- und Steuerkonzept planen.

Ergebnis

Die erste Version ist ein schlanker B2B-Checkout mit verifizierten Schulkonten, kursbezogenen Kontingenten und einer sicheren Bestellung-zu-Code-Übergabe. Nach Zahlung kann die Schule Codes als ZIP herunterladen, per E-Mail erhalten und/oder in den Codebestand ihres verknüpften Plattform-Schuladmin-Kontos übertragen. Die Codes bleiben bis zur Zuweisung an Lernende ungenutzt; die Plattform bleibt für Gültigkeit und Einlösung maßgeblich. Vor der Umsetzung müssen Produkt-/Preisregeln, Steuergebiet, Käuferzuordnung, Zustelloptionen und Rückerstattungsregeln freigegeben werden.


Gesamtkonzept | App Bootsschule B2B

Ergänzung zur Codezustellung und Startfreigabe

Diese Ergänzung legt die gewünschte Zustellung für den ersten Shop fest. Sie unterscheidet den Download beziehungsweise E-Mail-Versand einer ZIP-Datei von der Übernahme des Codebestands in das Konto einer Bootsschule auf der Lernplattform. Die beiden Zustellwege können bei einer Bestellung einzeln oder gemeinsam gewählt werden. Bei gemeinsamer Auswahl wird derselbe Code-Batch an beide Ziele geliefert; es werden keine zusätzlichen Codes erzeugt.

13. Zustelloptionen für den Käufer

Option

Verhalten nach bestätigter Zahlung

Voraussetzung

ZIP zum Download

Käufer lädt die ZIP-Datei im WooCommerce-Kundenkonto von der Bestellseite herunter. Erneuter Download ist nach Anmeldung möglich und wird protokolliert.

Bestellung ist bezahlt und Code-Batch erfolgreich angelegt.

ZIP per E-Mail

ZIP wird als Anhang an eine verifizierte Bestellkontaktadresse geschickt. Die Bestellbestätigung enthält keine Codes im Nachrichtentext.

E-Mail-Adresse verifiziert; Versand erfolgreich oder als Zustellfehler sichtbar.

In Bootsschulplattform-Konto

Der Shop veranlasst die Übernahme desselben ungenutzten Code-Batches in den Codebestand der verknüpften Bootsschule. Der Schuladmin kann Codes dort verwalten und Lernenden zuweisen.

Shopkonto ist sicher mit einem aktiven Schuladmin-Konto und genau einem Plattform-Tenant verknüpft.

Kombination

Käufer kann Download, E-Mail und Plattform-Konto gemeinsam auswählen. Die Bestellposition hat dennoch genau einen Code-Batch und eine gemeinsame Herkunftsreferenz.

Jede gewählte Zustellart ist einzeln erfolgreich oder mit eigenem Status ausgewiesen.

Im Checkout soll mindestens eine Zustellart verpflichtend gewählt werden. Empfehlung für den Start: Download standardmäßig anbieten; E-Mail und Plattformübernahme als zusätzliche Checkboxen. Wenn der Käufer eine Zustellart auswählt, die nicht erfolgreich ausgeführt werden kann, bleibt die Bestellung sichtbar auf „Zustellung ausstehend“ und der Vorgang ist sicher wiederholbar.

14. Bedeutung von Lieferung in das Plattform-Konto

Für dieses Konzept bedeutet „in mein Bootsschulplattform-Konto liefern“: die erworbenen Codes werden als ungenutzter Bestand im Konto der Bootsschule abgelegt. Das System löst sie nicht sofort ein und erstellt noch keinen Schülerzugang. Der Schuladmin kann anschließend einzelne Codes an Lernende verteilen oder zuweisen. So bleibt der Zweck eines Code-Kontingents erhalten.

Schritt

Anforderung

Verknüpfung vor Checkout

Shopkonto wird mit einem Plattform-Schuladmin verbunden. Der Nutzer bestätigt die Verbindung in der Plattform über einen einmaligen, kurzlebigen Verknüpfungscode oder einen OAuth-ähnlichen Zustimmungsablauf.

Tenant-Prüfung

Die Plattform bestimmt den Tenant aus dem angemeldeten Admin-Konto. Tenant-ID, Schulslug oder Nutzer-ID aus dem Checkout dürfen nicht ungeprüft übernommen werden.

Rollenprüfung

Nur aktiver Schuladmin oder eine ausdrücklich berechtigte Einkaufsrolle darf Codes in den Schulbestand liefern lassen. Ein Lernendenkonto darf keinen Bestand für eine Schule abrufen.

Batch-Buchung

Die Plattform legt pro WooCommerce-Bestellposition exakt einen Batch an. Codes bleiben ungenutzt und erhalten die richtige Kursdefinition, Laufzeit und tenant_id.

Anzeige im App-Konto

Neuer Bereich „Codebestand“ zeigt Kurs, verfügbare Anzahl, eingelöste Anzahl, Ablaufdatum, Bestellreferenz und Lieferstatus. Einzelne Klartextcodes sind nur für berechtigte Schuladmins sichtbar.

Zuweisung

Beim Zuweisen an einen Lernenden prüft die Plattform Tenant, Mitgliedschaft, Kursverfügbarkeit und Codezustand. Erst dann wird der Code eingelöst und der Kurszugang aktiviert.

Bestandsänderung

Rückerstattung oder Storno kann nur ungenutzte Codes sperren. Ein bereits eingelöster Code wird nicht stillschweigend zurückgesetzt.

Das Repository enthält bereits tenantgebundene Coupons und Schuladmin-Funktionen zum Verwalten beziehungsweise Zuweisen von Codes. Eine WooCommerce-Bestellverknüpfung, sichere Shopkonto-Verknüpfung und eine ausdrücklich als Shop-Codebestand gestaltete Benutzeransicht sind noch als Entwicklungsumfang einzuplanen. Die Integration darf nicht allein deshalb als erledigt gelten, weil Coupons bereits existieren.

15. ZIP-Datei und E-Mail-Zustellung

Detail

Festlegung für die Umsetzung

Dateiname

bootsschule-kurs-bestellung-<bestellnummer>.zip; keine Schülernamen im Dateinamen.

Inhalt

codes.csv mit einer Zeile pro Code sowie course_code, Kursname, Ablaufdatum und Einlösehinweis; optional eine kompakte PDF-Druckliste. Keine unnötigen Käufer- oder Lernerdaten.

Zeichensatz

UTF-8; CSV mit dokumentierter Trennung und Spaltenüberschriften, damit Excel/LibreOffice die Datei korrekt öffnet.

Erzeugungsort

ZIP dynamisch aus dem Code-Batch erzeugen, nicht als statischen WooCommerce-Downloadartikel verwalten.

Downloadschutz

Nur angemeldetes und bestellberechtigtes Shopkonto darf herunterladen. Link nicht erratbar; jeder Zugriff wird mit Benutzer, Auftrag, Zeitpunkt und Batch protokolliert.

E-Mail

Nur an verifizierte Kontaktadresse. Codes nicht zusätzlich in Betreff, HTML-Text, Logs oder Support-Benachrichtigungen kopieren.

Wiederholung

Fehlgeschlagener E-Mail-Versand lässt sich durch berechtigten Shopadmin erneut auslösen; erneuter Versand erzeugt keine neuen Codes.

ZIP-Verschlüsselung

Für den MVP entscheiden, ob Passwortschutz erforderlich ist. Falls ja, Passwort nicht in derselben E-Mail versenden. ZIP-Passwort ersetzt weder Konto- noch Berechtigungsprüfung.

Aufbewahrung

Code-Batch bleibt in der Plattform führend. Die Shopdatei wird nur so lange gespeichert, wie Download, Support und Aufbewahrungspflichten es erfordern.

Die E-Mail-Anhangvariante entspricht dem gewünschten ZIP-Versand. Sie ist bequem, verbreitet aber Zugangscodes zusätzlich in Postfächer und Mail-Backups. Deshalb sollte der Download im Shopkundenkonto immer verfügbar bleiben. Falls die Zustellung per E-Mail fehlschlägt, darf die Bestellung nicht als komplett geliefert markiert werden.

16. Bestell- und Lieferzustände

Zustand

Bedeutung und erlaubte Aktion

Zahlung ausstehend

Keine Codes erzeugen; nach bestätigter Zahlung weiterarbeiten.

Lieferung in Arbeit

App-Batch wird angelegt; Wiederholung verwendet dieselbe Idempotenz-ID.

Codes bereitgestellt

Batch existiert und zumindest eine gewählte Zustellart ist abgeschlossen.

Teilweise zugestellt

Ein Teil der ausgewählten Ziele ist erfolgreich, z. B. Plattformbestand erstellt, E-Mail-Versand fehlgeschlagen. Fehlende Zustellart sicher erneut versuchen.

Lieferfehler

Fehlergrund, Plattform-Request-ID und Wiederholungsaktion für Support sichtbar; niemals unbemerkt neue Codes anlegen.

Storniert vor Ausgabe

Kein Batch bzw. keine Codes ausgeben; ausstehende Integrationsaufträge stoppen.

Erstattet nach Ausgabe

Unbenutzte Codes sperren, eingelöste Codes auflisten und den Fall für manuelle Prüfung markieren.

Bestellstatus und Codezustand sind getrennte Dinge. Eine WooCommerce-Bestellung kann bezahlt sein, während E-Mail oder Plattformübertragung noch aussteht. Der sichtbare Status muss pro Lieferkanal anzeigen, was bereits erfolgreich war.

17. Technische Ergänzungen für Entwickler

Beim Checkout die gewählten Zustellkanäle, die verifizierte E-Mail-Adresse und die feste Plattformkonto-Verknüpfung revisionssicher an Bestellung und Bestellposition speichern.

Beim Code-Batch eine eindeutige externe Referenz aus Shop, WooCommerce-Bestell-ID und Bestellpositions-ID verwenden. Wiederholte Events müssen denselben Batch zurückgeben.

Eine eigene Batch-/Commerce-Referenz in der Plattform anlegen oder gleichwertig speichern. Das Coupon-Feld batch_label allein genügt nicht als belastbare Transaktions- und Idempotenzreferenz.

Den Codebestand für den Plattformweg im Mandanten des verknüpften Schuladmins bereitstellen; für ZIP-Weg die Codes kontrolliert einmalig exportieren. Beide Wege referenzieren denselben Batch.

API-Ausgaben minimieren: Wenn nur Plattformzustellung gewählt ist, muss WooCommerce keine Klartextcodes erhalten. Wenn ein ZIP gebraucht wird, nur ein autorisierter Export-Endpunkt darf die Werte liefern.

Lieferkanäle unabhängig und wiederholbar ausführen. Versandfehler bei E-Mail darf eine bereits erfolgreiche Code-Batch-Erstellung nicht nochmals auslösen.

Code-Dateien aus temporärem Speicher ausliefern und nach definierter Frist löschen; Downloads und E-Mail-Wiederholungen auditieren.

Administrationsseite für Bestellung, Bestellposition, App-Batch-ID, Empfängerstatus und Fehler einbauen. Klartextcodes im Standard-Supportscreen ausblenden.

Teilmenge/Storno muss pro Code prüfen, ob dieser noch ungenutzt ist. Eingelöste Codes nicht automatisch entziehen.

Im Staging Testkonten mit getrenntem Shopkonto, Schuladmin, Lernendem und Tenant einrichten; Bestellungen mit mehrfachen Webhooks, E-Mail-Fehlern, App-Timeouts und Erstattung durchspielen.

18. Noch offene Festlegungen vor dem Entwicklungsstart

Entscheidung

Noch festzulegen

Verkaufskatalog

Alle im aktuellen Plattformkatalog vorhandenen Kursdefinitionen sind Produktkandidaten. Preise stehen noch nicht fest. ADD-FKN-TO-SKN ist Entwurf und kann laut App-Dokumentation noch keine Prüfung starten; daher vor Verkauf fachlich freigeben oder bis dahin ausblenden.

Codegültigkeit

Zu unterscheiden: Codeablauf vor Einlösung (Coupon.expires_at, optional) und Zugangslaufzeit nach Einlösung (derzeit tenantweite Branding-Einstellung course_validity_days). Es gibt derzeit keine eigene Standardlaufzeit je Kurs; gewünschte per-Kurs-Regel muss fachlich bestätigt und ggf. in der App ergänzt werden.

Zustell-Standard

Empfehlung für MVP: Download ist verfügbar; Käufer kann ZIP-E-Mail und/oder Plattformkonto als Zustellziel wählen. Fehlgeschlagene optionale Zustellwege bleiben separat wiederholbar.

Plattformkonto

Festgelegt: Ziel ist der Codebestand der Bootsschule im Plattformkonto. Schuladmin und WP-Shop-Administrator sind dieselbe Person. Plattformtenant aus angemeldetem Schuladmin ableiten.

Kontoverknüpfung

Festgelegt: vorhandene WP- und Plattformkonten vor Checkout sicher miteinander verknüpfen. Verknüpfung per kurzlebigem Einmalcode/Zustimmungsablauf; Passwörter nicht systemübergreifend speichern oder abgleichen.

Käuferberechtigungen

Für den MVP eine einkaufsberechtigte Person je Schule: der Schuladmin/WP-Shop-Administrator. Zusätzliche Käuferkonten und geteilte Codebestände bleiben spätere Ausbaustufe.

Zahlung und Rechnung

Empfehlung dokumentiert: Karte als direkter Onlineweg; Rechnung/Banküberweisung nur für freigegebene Schulen. Lieferung nach bestätigtem Zahlungseingang; Kreditlimit-/Zahlungsziel noch nicht entschieden.

Steuer und Recht

Verkaufsgebiete AT/DE/CH/ES; ausschließlich B2B. Die konkrete steuerliche Einordnung der verkauften Plattformcodes, Rechnungsangaben und Schweizer Bezugsteuer/MWST muss vor Go-live mit Steuerberatung bestätigt werden.

Erstattungen

Frist und Ablauf für ungenutzte Codes; Umgang mit teilweise eingelösten Bestellungen und Chargebacks.

E-Mail und Dateizugang

Absenderdomain, Betreff, Mailtext, maximale ZIP-Größe, Zustell- und Wiederholungsregeln, Speicherfrist.

Support und Betrieb

Wer bearbeitet fehlgeschlagene Lieferungen, wie schnell, und welche IDs werden für Support angezeigt? Alarmempfänger und Betriebszeiten.

Startfreigabe

Der Shop kann zur Umsetzung freigegeben werden, wenn (1) Preise und Mengen je Verkaufskurs feststehen, (2) Codeablauf und Kurszugangsdauer fachlich getrennt entschieden sind, (3) B2B-Steuer-/Rechnungslogik für AT, DE, CH und ES von Steuerberatung freigegeben wurde, (4) Zahlungsanbieter und Zahlungsauslöser feststehen und (5) die App-Änderungen für sichere Admin-Kontoverknüpfung, Commerce-API, Codebatch und Schul-Codebestand eingeplant sind. Erstattungen, Support-SLA sowie E-Mail-/Dateiaufbewahrung bleiben aktuell offen. Für den MVP: manuelles Schul-Onboarding, sicher verknüpfter Schuladmin/WP-Shop-Administrator, Download im Shopkonto, optionale ZIP-E-Mail und optionaler Plattformbestand. Jede bestellte Position erzeugt genau einen Plattformbatch; App bleibt führend für Codes und Einlösung.


Gesamtkonzept | App Bootsschule B2B

19. Festlegungen nach Rückmeldung und Softwareprüfung

Dieser Abschnitt ersetzt frühere Annahmen, soweit sie mit den nun bestätigten Rahmenbedingungen kollidieren. Preise bleiben ausdrücklich offen. Die Recherche bezieht sich auf den öffentlichen Stand des Repositorys main am 12.11.2026.

Bestätigte Geschäftsregeln

Bereich

Festlegung für das Konzept

Käuferkreis

Nur Bootsschulen und andere verifizierte Geschäftskunden; kein B2C-Checkout.

Gebiete

Österreich, Deutschland, Schweiz und Spanien.

Shop-/Plattformrolle

Der Schuladmin ist zugleich der Administrator des WooCommerce-Shopkontos. Für den MVP wird eine verknüpfte Admin-Person je Schule angenommen.

Sortiment

Alle aktuellen Kursdefinitionen werden als Kandidaten berücksichtigt; Verkaufbarkeit bleibt eine explizite Freigabe und darf nicht allein aus einem Datenbankeintrag automatisch folgen.

Preise

Noch nicht festgelegt. Preise, Mindestmengen und Mengenstaffeln sind nicht in der Kursdefinition als Shoppreise hinterlegt.

Codes

Bestehender Verkaufsweg bleibt: ZIP-Download und/oder ZIP per E-Mail. Käufer kann zusätzlich den Bestand in das verknüpfte Schuladmin-Konto liefern lassen.

Offen gelassen

Erstattungs-/Kulanzregeln, Zahlungsziel für Rechnungskauf, Support-SLA und genaue E-Mail-/Dateiaufbewahrung werden vorerst nicht festgeschrieben.

Was die Software pro Kurs bereits festlegt

Der Seed-Katalog definiert 14 stabile Kurscodes mit Namen, Kurstyp und Modulzuordnung. Diese Angaben sind die fachliche Basis für WooCommerce-Mappings und dürfen nicht aus frei editierbaren Shopnamen abgeleitet werden. Die App definiert Kursinhalt und Prüfungslogik; das macht Preise, Codekontingente und Shop-Laufzeiten noch nicht automatisch verfügbar.

course_code

Produktname

Typ

App-Status / Shop-Hinweis

SBF-SEE

SBF See

Vollkurs

Kursdefinition vorhanden; Prüfungsregeln vorhanden.

SBF-BIN-MOTOR

SBF Binnen

Vollkurs

Kursdefinition vorhanden; Prüfungsregeln vorhanden.

SBF-BIN-SEGEL

SBF Binnen unter Segel

Vollkurs

Kursdefinition vorhanden; Prüfungsregeln vorhanden.

SBF-SEE-BIN

SBF See & Binnen

Kombikurs

Kursdefinition vorhanden; beide Teilkurse müssen korrekt als ein Kaufprodukt/Codezugang getestet werden.

ADD-SEE-TO-BIN

Ergänzung See → Binnen

Ergänzungskurs

Kursdefinition vorhanden; Prüfungsregeln vorhanden.

ADD-BIN-TO-SEE

Ergänzung Binnen → See

Ergänzungskurs

Kursdefinition vorhanden; Prüfungsregeln vorhanden.

ADD-BIN-SEGEL

Ergänzung Binnen → Segeln

Ergänzungskurs

Kursdefinition vorhanden; Prüfungsregeln vorhanden.

SRC

SRC Vollkurs

Vollkurs

Kursdefinition vorhanden; Prüfungsregeln vorhanden.

UBI

UBI Vollkurs

Vollkurs

Kursdefinition vorhanden; Prüfungsregeln vorhanden.

SRC-UBI

SRC & UBI Kombikurs

Kombikurs

Kursdefinition und Prüfungsbögen vorhanden; Bögen laufen über Teilkurs-Regeln, nicht über ein einzelnes Kombi-Regelwerk.

ADD-UBI-TO-SRC

Ergänzung SRC → UBI

Ergänzungskurs

Kursdefinition vorhanden; Prüfungsregeln vorhanden.

SKN

SKN Vollkurs

Vollkurs

Kursdefinition vorhanden; Prüfungsregeln vorhanden.

FKN

FKN Vollkurs

Vollkurs

Kursdefinition vorhanden; Prüfungsregeln vorhanden.

ADD-FKN-TO-SKN

Ergänzung FKN → SKN

Ergänzungskurs

Im Repository als Entwurf dokumentiert; laut README kann damit noch keine Prüfung gestartet werden. Bis fachlicher Freigabe nicht automatisch verkaufen.

Wichtige Abweichung: Laufzeit ist aktuell nicht pro Kurs

Die App enthält zwei verschiedene Zeitregeln. Der einzelne Coupon besitzt ein optionales expires_at-Datum; damit kann ein Code vor der Einlösung ablaufen, aber der Code-Datensatz hat keinen aus dem Kurs abgeleiteten Standardwert. Nach Einlösung berechnet die App die Kurszugangsdauer aus tenant.branding.course_validity_days. Diese Einstellung gilt auf Bootsschulebene und wird für den Kurszugang verwendet; sie ist kein separater Wert je Kurs. Wenn jede Kursart eine eigene Zugangsdauer erhalten soll, braucht es eine neue per-course Konfiguration und eine Commerce-Katalog/API-Ausgabe dafür. Für den MVP muss entschieden werden, ob die bestehende tenantweite Laufzeit für alle Kurse genügt oder ob die App vor dem Shopbau erweitert wird.

Außerdem sind Kurszugangsdauer nach Einlösung und Codeablauf vor Einlösung fachlich getrennte Dinge. Die erste gehört zum Entitlement; die zweite zum Coupon. WooCommerce darf diese Werte nicht selbst erfinden oder als eigene führende Laufzeitlogik speichern.

20. Best-Case-Empfehlung für B2B-Verkauf in AT, DE, CH und ES

Empfohlen wird ein zugangsbeschränkter B2B-Shop mit manueller Erstfreigabe der Bootsschule, bestätigter Kontoverknüpfung vor dem ersten Kauf und klarer Trennung zwischen Bestellzahlung und Code-Lieferstatus. Die Empfehlung zu Zahlungs- und Steuerablauf ist eine technische Produktentscheidung; die konkrete steuerliche Behandlung muss vor Go-live fachlich geprüft werden, weil sie vom Sitz des Verkäufers und der rechtlichen Einordnung der Code-/Plattformleistung abhängt.

Thema

Empfohlener MVP-Ablauf

B2B-Prüfung

Bei Registrierung Firmenname, Rechnungsanschrift, Land, Kontaktperson und erforderliche Unternehmens-/Umsatzsteuer-ID erfassen. Shopkonto bis zur manuellen Freigabe nicht bestellberechtigt machen.

EU-UID-Prüfung

Für grenzüberschreitende EU-B2B-Bestellungen UID/VAT-ID mit VIES prüfen. Prüfergebnis, Zeitpunkt und Käuferdaten zur Bestellung speichern. Bei VIES-Ausfall oder ungültigem Ergebnis Bestellung nicht stillschweigend als steuerfrei behandeln; in manuelle Prüfung geben.

EU-Umsatzsteuer

EU-Regel für B2B-Dienstleistungen sieht grundsätzlich den Ort des Geschäftskunden vor; die Steuerschuld kann bei erfüllten Voraussetzungen auf den Kunden übergehen (Reverse Charge). Vor Umsetzung ist zu bestätigen, ob Codeverkauf/Plattformzugang in diesem Geschäftsmodell tatsächlich unter diese Dienstleistungsregel fällt und welche Rechnungsvermerke gelten.

Schweiz

CH separat behandeln: Schweizer MWST/Bezugsteuer und Registrierungspflicht hängen vom Leistungsmodell und Umsatz ab. Die Schweizer ESTV nennt bei ausländischen Unternehmen eine weltweite Umsatzgrenze von CHF 100’000 für steuerbare/zero-rated Leistungen und beschreibt Ausnahmen u. a. für Leistungen unter Bezugsteuer. Vor Verkauf nach CH Steuerberatung/ESTV-Klassifizierung einholen.

Preis- und Währungsanzeige

Nettopreise als B2B-Grundlage anzeigen; Steuer, Gesamtbetrag und Steuerhinweis im Checkout und auf Rechnung getrennt ausweisen. Preise sind noch offen. Für CHF entweder feste CHF-Preisliste oder bewusst EUR-only festlegen; keine unbemerkte tägliche FX-Umrechnung als verbindliche B2B-Preisliste verwenden.

Zahlung

Online-Kartenzahlung als einfacher Direktweg; zusätzlich Banküberweisung/Rechnung nur für freigegebene Schulen. Rechnungskauf benötigt eine manuelle Kredit-/Zahlungsziel-Freigabe. Konkreter Gateway und Gebührenvergleich bleiben nach Festlegung des Verkäuferlands offen.

Liefertrigger

Codebatch erst nach serverseitig verifiziertem Zahlungsstatus erzeugen. Rückkehr auf die Danke-Seite reicht nicht. Banküberweisung bleibt „Zahlung ausstehend“, bis Zahlung im Shop bestätigt oder autorisiert freigegeben wurde.

Lieferkanäle

Download im geschützten Shopkonto als verlässlicher Standard. ZIP-E-Mail und Plattform-Codebestand als wählbare Ergänzungen. Jeder Kanal hat eigenen Status und Retry; erneuter Versand oder Download erzeugt niemals neue Codes.

Kontoverknüpfung

Eine Person darf beide Adminrollen haben, aber die Systeme behalten getrennte Konten und Secrets. Plattformadministrator bestätigt die Shopverknüpfung per kurzlebigem Einmalcode/Zustimmungsflow; Shop erhält keine App-Passwörter. Tenant kommt serverseitig aus dieser Verknüpfung.

Währung/Markt

AT/DE/ES verwenden Euro; CH-Währung und fixe Preislisten sind noch zu entscheiden. Die Länderfreigabe darf erst nach landesspezifischem Steuer-/Rechnungscheck aktiviert werden.

Steuerliche Einordnung: notwendige fachliche Freigabe

Die App gibt online Lerninhalte und Prüfungsübung frei; im Shop wird ein Zugangscode verkauft. Ob der Verkauf steuerlich als digitale Dienstleistung, Bildungsleistung oder Voucher/Einzweckgutschein einzuordnen ist, lässt sich aus dem Repository allein nicht zuverlässig bestimmen. Die EU-Kommission weist ausdrücklich darauf hin, dass Leistungsort sowohl von der genauen Art der Leistung als auch vom Status des Kunden abhängt. Deshalb darf das Dokument keine endgültige Steuerrate oder pauschale Befreiung voraussetzen. Eine Steuerberatung soll das konkrete Modell für den Sitz des Verkäufers und jedes Verkaufsland freigeben, insbesondere die CH-Regel.

Quellen der Recherche und der Softwareprüfung

App-Katalog: database/seeders/CourseSeeder.php; Prüfungsstatus: database/seeders/ExamRuleSetSeeder.php und README.md im Repository https://github.com/hastla007/boatingschool (geprüft 12.11.2026).

App-Laufzeit: app/Services/CouponRedemptionService.php (Kurszugang verwendet tenant branding course_validity_days), app/Models/Coupon.php (optional expires_at), app/Models/Entitlement.php.

EU-Kommission, Place of taxation: https://taxation-customs.ec.europa.eu/taxation/vat/vat-directive/place-taxation_en

EU-Kommission, Persons liable for VAT: https://taxation-customs.ec.europa.eu/taxation/vat/vat-directive/persons-liable-vat_en

Your Europe, VAT number check / VIES: https://europa.eu/youreurope/business/finance-and-tax/vat/check-vat-number-vies/index_en.htm

Schweizer ESTV, VAT liability – foreign companies: https://www.estv.admin.ch/en/vat-liability-foreign-companies

Schweizer ESTV, Questions & answers VAT (electronic services / training): https://www.estv.admin.ch/en/questions-answers-vat

21. Noch offen und nicht als entschieden dokumentiert

Preise, Mindestmengen und Rabattstaffeln; per-Kurs-Zugangszeit bzw. Entscheidung zur bestehenden tenantweiten Laufzeit; Einlösefrist der Codes; konkrete steuerliche Qualifikation und Rechnungslogik; CHF- oder EUR-Abrechnung in der Schweiz; ausgewählter Zahlungsanbieter und dessen Gebühren; Rechnungskauf-Limit/Zahlungsziel; Erstattungs-/Chargeback-Prozess; E-Mail-Absender, maximale ZIP-Größe und Speicherfrist; Support-SLA und Alarmempfänger. Diese Punkte bleiben offen, bis sie gemeinsam mit Buchhaltung/Steuerberatung bzw. dem Betreiber entschieden sind.

Die Preisfelder werden im Konzept bewusst als „offen“ geführt. Kurscode, Name, Typ, Module und aktuelle Prüfungsfreigaben sind aus der App zu übernehmen; für ein belastbares Verkaufsprodukt muss die spätere Commerce-API zusätzlich die freigegebenen Verkaufsparameter pro Kurs liefern (Verkaufbarkeit, Codeablaufregel, Mindestmenge, Preise/Steuerklasse und gegebenenfalls Laufzeitregel).


Gesamtkonzept | App Bootsschule B2B

22. Ergebnis der moderierten Expertendebatte

Die folgende Debatte ist eine strukturierte Simulation der bestätigten Fachrollen. Sie trennt technische Empfehlungen von steuerlichen Fragen, die nur mit Kenntnis der Verkäufergesellschaft und der konkreten Leistungsbeschreibung verbindlich beantwortet werden können.

Die drei Rollen prüfen und schärfen den Vorschlag

WooCommerce-B2B-Architektur

„WooCommerce soll Besteller, Preise, Zahlung, Rechnungsdaten und Shopbestellungen führen. Die App soll nicht durch doppelte Kurs-Stammdaten auseinanderlaufen. Ich würde daher ein WooCommerce-Produkt mit dem unveränderlichen course_code der Plattform verbinden. Ein Preis darf im Shop gepflegt werden, aber Verkaufbarkeit und Einlösung müssen von der Plattform bestätigt werden. Wichtig ist eine klare B2B-Freigabe vor dem ersten Checkout.“

API- und Integrationsarchitektur

„Das reicht nur, wenn der Käufer nicht selbst eine tenant_id im Checkout setzen kann. Der Schuladmin muss die Shopidentität ausdrücklich mit seinem bestehenden Plattformkonto verbinden; die App leitet daraus den Tenant ab. Pro Bestellposition braucht es eine unveränderliche Auftragsreferenz und einen Idempotenzschlüssel. Wiederholte Webhooks liefern denselben Batch zurück. Wenn Download, ZIP-Mail und App-Bestand gemeinsam gewählt sind, müssen alle drei denselben Batch verwenden; der reine Plattformweg soll keine Klartextcodes an WordPress schicken.“

Steuer- und Rechtsprüfung

„Die technische Auswahl B2B-only beantwortet noch nicht, ob der Code rechtlich ein Gutschein, ein Zugang zu einer elektronischen Leistung oder eine Bildungsleistung ist. Ohne Verkäuferland, Vertragsgestaltung und genaue Leistungsbeschreibung sollten wir weder eine Steuerbefreiung noch Reverse Charge pauschal konfigurieren. EU-UID-Prüfung und Belegaufbewahrung gehören in den Prozess; die Schweizer MWST ist separat zu prüfen.“

Die Integrationsrolle widerspricht einer automatischen Steuerlogik auf Basis allein einer UID: Sie kann Geschäftskundenstatus belegen helfen, ersetzt aber nicht die rechtliche Einordnung der gelieferten Leistung. Die WooCommerce-Rolle stimmt zu, trennt daher Kontofreigabe und Steuerentscheidung und schlägt einen sichtbaren manuellen Prüfstatus vor. Gemeinsame Leitlinie: Die App bleibt führend für Kurs und Codes; WooCommerce führt Preis, Zahlung und Bestellung; die Verknüpfung bestimmt den Tenant; Steuereinstellungen werden erst nach fachlicher Freigabe aktiviert.

Beschlossene Produkt- und UX-Leitlinien

Schritt im Kauf

Konzeptanforderung

Schule freischalten

Vor dem Checkout Firma und Rechnungsland erfassen, Schulkonto freigeben und den WP-Shop-Administrator mit dem Plattform-Schuladmin verbinden.

Kurs wählen

Produktseite zeigt stabilen Kurscode, Kursname und -typ, Codeanzahl, Nettoangebot, Steuerhinweis, Codeablauf bzw. Zugangsdauer und den Hinweis: Der Kauf liefert ungenutzte Codes in den Schulbestand; er aktiviert noch keine Lernenden.

Zustellung wählen

Mindestens Shopkonto-Download anbieten; ZIP per E-Mail und Übernahme in den verknüpften Plattformbestand als auswählbare Ziele. Bei Mehrfachauswahl werden keine zusätzlichen Codes erzeugt.

Zahlung bestätigen

Nach Bestellung verständlicher Status „Zahlung ausstehend“. Codes entstehen erst nach bestätigtem Zahlungseingang bzw. einer ausdrücklich genehmigten Rechnungsfreigabe.

Codes verwalten

Bestellseite zeigt Codebatch, Menge, Kanalstatus, Zeitstempel und Fehlerhinweis. Plattformkonto zeigt den ungenutzten Schulbestand mit verfügbarem, zugewiesenem, eingelöstem, abgelaufenem und gesperrtem Anteil.

Fehler beheben

Jeden Zustellkanal separat erneut versuchen können. Ein Fehler beim Mailversand darf einen erfolgreichen App-Batch nicht neu erzeugen; Standard-Supportansichten blenden Klartextcodes aus.

Die Benutzerführung soll ausdrücklich „Codekontingent für die Schule“ von „Kurszugang für einen Lernenden“ unterscheiden. Die Plattformübernahme ist ein Lagerort für ungenutzte Codes, kein sofortiger Schülerzugang. Für Käufer werden Status und nächster sinnvoller Schritt direkt bei Bestellung und Codebestand erklärt.

Betrieb, Pilot und Marketing

Bereich

Ergänzung des Konzepts

Pilotbetrieb

Vor breitem Go-live mit zwei bis drei Bootsschulen einen Ende-zu-Ende-Pilot durchführen. Enthalten: Konto-Verknüpfung, EU- und CH-Testfälle, Zahlung, ZIP-Download, ZIP-Mail, Plattformbestand, Webhook-Wiederholung, Mailfehler und Erstattungssimulation.

Tägliche Kontrolle

Administrationsübersicht für bezahlte, aber noch nicht gelieferte Bestellungen, Teillieferungen, fehlgeschlagene API-Aufrufe, Mailfehler und wiederholte Downloads. Supportfall anhand Woo-Bestellnummer, Bestellpositions-ID, App-Batch-ID und Request-ID nachvollziehen.

Produktmarketing

Nutzenversprechen auf den Schulbetrieb zuschneiden: Kurskontingente zentral kaufen, Codes im Schulbestand verwalten und Lernenden zuweisen. Mit klarer B2B-Registrierung und Demo-/Pilotangebot starten; keine unbelegten Erfolgs- oder Prüfungsversprechen.

Länderauftritt

Deutsch für AT/DE und deutschsprachige CH-Schulen; spanische Produkt- und Checkout-Texte für ES einplanen. Preise, Steuerhinweise, Währung und Rechnungstexte pro Markt erst nach fachlicher und kaufmännischer Freigabe veröffentlichen.

Erfolgsmessung

Im Pilot messen: registrierte und freigegebene Schulen, Checkout-Abbruch, Zeit von Zahlung bis Codebereitstellung, Fehlerrate je Zustellkanal, erste Codezuweisung und Wiederbestellung.

Konkrete Anforderungen an den Entwicklungsauftrag

WooCommerce-Produkt enthält eine unveränderliche Zuordnung zu course_code; Kursname, Kurstyp, Verfügbarkeit und Einlöseregeln werden aus einem freigegebenen Plattformkatalog bezogen. Preise und Mengenstaffeln bleiben WooCommerce-/Shopregeln, bis sie explizit in der App verwaltet werden.

App-Commerce-API prüft die aktive Kontoverknüpfung und leitet Tenant und Plattform-Schuladmin selbst ab. Kein frei gesetzter Tenant aus einem Browser- oder Checkout-Feld.

Pro Bestellposition genau ein idempotenter Codebatch. Lieferkanäle sind getrennte Empfängeraktionen mit eigenem Status; erneutes Senden, Download oder Webhook erzeugt keine neuen Codes.

Bei App-Kontozustellung Codebestand im Schuladmin-Tenant buchen, ohne Codes einzulösen. Bei ZIP-Ausgabe einen berechtigten Export erzeugen; Zugriff kurzlebig, kontrolliert und protokolliert bereitstellen.

Steuer- und Rechnungsergebnis, geprüfte UID, Verkäufer- und Käuferland, Währung und verwendete Regelversion revisionssicher mit der Bestellung speichern. Steuerlogik nicht aus dem Kurscode ableiten.

Produktfreigabe getrennt vom bloßen Vorhandensein einer Kursdefinition behandeln. ADD-FKN-TO-SKN bleibt bis fachlicher Freigabe gesperrt; Kombikurse erhalten eigene End-to-End-Abnahmetests.

Offene Fragen für die nächste Arbeitsrunde

In welchem Land sitzt die verkaufende Gesellschaft und wer stellt die Rechnung aus?

Soll der Kurszugang nach Codeeinlösung künftig je Kurs unterschiedlich lange gelten, oder genügt vorerst die bestehende Laufzeit je Bootsschule?

Soll die Schweiz in CHF abgerechnet werden oder zunächst in EUR? Welche feste Preisliste gilt je Markt, sobald Preise festgelegt werden?

Wer gibt Steuerklassifizierung und Rechnungslogik für EU/CH frei, und wer ist intern für Lieferfehler/Support verantwortlich?

Die Preisbeträge müssen diese Runde nicht blockieren. Der Shop kann mit Preisfeldern, Steuerregeln, Mengenstaffeln und Marktlisten vorbereitet werden; vor Liveverkauf müssen die tatsächlichen Werte und Freigaben eingetragen sein.


Gesamtkonzept | App Bootsschule B2B

23. Kontensynchronisierung und Zustell-Fallback

Dieser Abschnitt konkretisiert die zuletzt abgestimmten Zustell- und Kontenregeln. Pro Bestellung wird genau ein Haupt-Zustellweg gewählt. Ein automatischer Ersatzweg darf erst nach einem bestätigten Fehler eingesetzt werden und muss dem Käufer beim Kauf transparent angekündigt sein. Sämtliche Zustellversuche beziehen sich auf denselben Code-Batch.

Ein Hauptweg je Bestellung

Haupt-Zustellweg

Ergebnis bei Erfolg

Vorgeschlagener Ersatz

WooCommerce-Kundenkonto

Geschützter Downloadlink zur bestellungsspezifischen ZIP-Datei.

E-Mail-Link an verifizierte Adresse, falls beim Checkout als Fallback erlaubt.

E-Mail-Link

E-Mail enthält einen geschützten, befristeten Link zur ZIP-Datei.

Download im WooCommerce-Konto, sofern Konto vorhanden und Käufer darüber informiert werden kann.

Bootsschulplattform-Konto

Ungenutzter Code-Bestand wird dem verknüpften Schuladmin-Tenant zugeordnet.

Downloadlink oder E-Mail-Link nur nach ausdrücklicher Zustimmung; sonst Supportfall.

Fehlerbehandlung und Wiederholung

Vor einem Wiederholungsversuch wird zuerst der Status der ursprünglichen App-Anfrage abgefragt. Ein Timeout gilt nicht als Beweis, dass die Code-Erzeugung fehlgeschlagen ist.

Vorläufiger Vorschlag: drei Wiederholungsversuche mit wachsendem Abstand. Danach erhält die Bestellung den Status „Zustellung offen“ oder „Zustellfehler“ und der Shopadmin wird informiert.

Ein Fallback ist nur nach bestätigtem Fehlschlag und entsprechend der beim Kauf angezeigten Ersatzregel erlaubt. Es entstehen keine neuen Codes.

Jeder Kanal hat eigenen Status und Zeitstempel. Ein erfolgreicher Kanal darf nicht durch einen Fehler des zweiten Versands rückgängig gemacht werden.

Downloadzugriff als Startvorschlag: 90 Tage und bis zu fünf Downloads; danach kann ein berechtigter Admin denselben Batch nach Prüfung erneut freigeben.

Konten und führende Systeme

WooCommerce darf Shopkonten bei Registrierung anlegen. Der Abgleich mit der Bootsschulplattform erfordert jedoch einen eigenen Connector und App-API-Funktionen. Die App bleibt führend für Schule/Tenant, App-Identität und Codebestand. WooCommerce ist führend für Shopkonto, Bestellungen, Zahlung und Rechnungsdaten. Ein Shopkonto allein erzeugt noch keine freigegebene Schule.

Registrierung in WooCommerce: Shopkonto wird erstellt; die App erhält über den Server-Connector eine Anlage- oder Verknüpfungsanfrage. Ein bestehender Tenant wird über einen kurzlebigen Bestätigungscode verknüpft. Neue Schulen bleiben bis zur B2B-Prüfung im ausstehenden Status.

Schulanlage durch den Superadmin in der App: Die App stößt idempotent die Anlage eines WooCommerce-Kundenkontos an. Der Shopadmin erhält einen Passwort-Einrichtungslink; Passwörter werden nicht zwischen den Systemen synchronisiert.

Verknüpfung über stabile App-Tenant-ID und WooCommerce-Kunden-ID; nicht allein über eine passende E-Mail-Adresse. Konflikte und doppelte E-Mails werden zur Prüfung markiert.

Bootsschul-Kunden erhalten eine begrenzte Shop-Kundenrolle. WordPress-Administratorrechte bleiben beim Shopbetreiber.

Vor Entwicklung noch zu bestätigen

Ersatzweg je Hauptweg und nötige Zustimmung im Checkout.

Ob neu registrierte Shopkunden sofort App-Nutzer werden oder erst nach verifizierter E-Mail und Schulprüfung.

Vorgeschlagene Downloadfrist von 90 Tagen und fünf Downloads.

Wer Sync- und Zustellfehler bearbeitet und welche Benachrichtigungen Käufer sowie Shopadmin erhalten.

Gesamtkonzept | App Bootsschule B2B
