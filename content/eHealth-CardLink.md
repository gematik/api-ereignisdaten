# Ereignisdatenschnittstelle eHealth-CardLink

Die Schnittstelle dient zum Empfang von Ereignisdaten von mit der Telematik Infrastruktur verbundenen Systemen. Hier im Spezifischen Instanzen des eHealth-CardLink Dienstes zum Empfang von Ereignisdaten zu PDT77.

Formate und Lieferintervalle sind in gemSpec_Perf unter [A_25266] erfasst. 

Für eine konkrete Implementierung eines Sende-Clients sind den nachfolgenden Abschnitten die notwendigen Informationen zu entnehmen:

## 1. Aufbau des Request

### URL-Informationen

   | Übersicht:|  |
|:----------|:----------|
| *Basis* | https://bd.prod.ccs.gematik.solutions/api/v1/sensor/cardlink/pn/:CI |
| *URL-Parameter*   | CI => die von der gematik zur Verfügung gestellte CI-Id für die jeweilige Betriebsumgebung | 
| *Beispiel:*  | https://bd.prod.ccs.gematik.solutions/api/v1/sensor/cardlink/pn/CI-9999999|



### Header-Informationen
- **http-Methode:** 
  - POST
- **http-Version:** 
  - enstpricht 1.1
- **http-Header:**
  - *authorization*: auszuführen mit dem entsprechend erzeugten Token, wie beispielsweise "Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6..." -> Weitere Erläuterung befinden sich im allgemeinen Abschnitt zur Authentifizierung
  - *content-type*: „application/json“
  - *content-length*: Entsprechend der Größe des Bodys zu setzen
  - *content-encoding*: Auf den Header ist zu verzichten, eine Komprimierung des Inhaltes soll nicht vorgenommen werden 

### Request Body: 
Der Post-Body ist gemäß der Beschreibung der gemSpec_Perf aus Abschnitt 3.23.2 aufzubauen.
  
Beispielschema der Ereignisdatenlieferung eHealth-CardLink:
```json
  {
  "timestamp": 1711442517123,
  "eventId": 3,
  "ikn": 108018121,
  "duration": 28,
  "err": null
  }
  ```
 
 Für die Abfrage selbst weiterhin ein beispielhafter curl Request:

<div style="border:1px solid #ccc; padding:10px; border-radius:5px; background:#f9f9f9;">
curl --location --request POST 'https://bd.prod.ccs.gematik.solutions/api/v1/sensor/cardlink/pn/CI-9999999' \

--header 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbG...' \
--header 'Content-Type: application/json' \
--data-raw '{"timestamp": 1711442517123,"eventId": 3,"ikn": 108018121,"duration": 28,"err": null}'

</div>

## 2. Erläuterung der Übertragungsattribute 

Ergänzend zur A_25266der gemSpec_Perf hier nachstehend die Erläuterung derzu übermittelnden Attribute:

Die **validierten** Lieferattribute ergeben sich wie folgt:

| **Art der Liefersystematik** | **Attribut** | **Beschreibung** |
|:----------|:----------|:----------|
| URL-Pfad | CI| je Betriebsumgebung (RU, TU, PU) wird eine CI-Id von der gematikbereitgestellt, wird je Umgebung als statischer Wert geliefert |
| Body  | timestamp | Zeitpunkt als Unix Timestamp der UTC Zeitzone, IntegerDer Zeitpunkt wird in Millisekunden angegeben. <br> Beispiel: <br>  -> 1711442517123=> 26.03.2024 08:41:57.123UTC <br> -> 1718432784567=> 15.06.2024 06:26:24.567 UTC <br> Bildet den Zeitpunkt des Beginns der Messung der Bearbeitungszeit nach A_24810der gemSpec_Perf ab.  |
| Body | eventID | Ergebnis des Prüfungsnachweises, Feld EventId, Integer |
| Body | ikn  | IK Nummer der genutzten eGK, Integer |
| Body | duration  | Dauer des Anwendungsfalles vom Start der Ausführung desAnwendungsfalles bis zu dessen Ende in Millisekunden, Integer. <br> Der Start und Ende der Messung des Anwendungsfalls findetwie in A_24810der gemSpec_Perf dargelegt statt. |
| Body | err  | ErrorCodedes Anwendungsfalls ReadVSD, Integer |


## 3. Aufbau der Response 

Der Aufbau der Response ist in der allgemeinen Dokumentation der Schnittstellen unter 4. "Allgemeiner Aufbau der Response" definiert und erläutert und für alle beschriebenen Fälle gemeinhin gültig. 

Eine allgemeine Auflistung zu den jeweiligen http-Statuscode befindet sich ebenfalls in der Dokumentation unter Abschnitt 4.
