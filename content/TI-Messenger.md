# Ereignisdatenschnittstelle TI-Messenger 

Die Ereignisdatenschnittstelle TI-Messenger dient zum Empfang von Ereignisdaten zu PDT80 sowie PDT83.
Formate und Lieferintervalle sind in gemSpec_Perf unter [A_27487] erfasst. 

Für eine konkrete Implementierung eines Sende-Clients sind den nachfolgenden Abschnitten die notwendigen Informationen zu entnehmen:

## 1. Aufbau des Request

Entsprechend der Spezifikation des TI-Messenger Fachdienstes ergeben sich für den Aufbau der URL-Parameter die Unterscheidung von PDT80 sowie PDT83. Zusätzlich entsteht eine Aufgliederung durch Unterscheidung der täglichen und stündlichen Datenlieferung, die sich in Lieferintervall sowie im Post-Body unterscheidet. 

Über die URL Parameter ist die Lieferung für den korrekten Produkttyp und dem Zeitintervall zuzuordnen. 

https://bd.prod.ccs.gematik.solutions/api/v1/sensor/bestand/[:pdt]/[:interval]/[:ci]

| **PDT** | **Intervallvariable** | **URL** |
|:----------|:----------|:----------|
| *PDT80* | stündliche Lieferung - 24| https://bd.prod.ccs.gematik.solutions/api/v1/sensor/bestand/pdt80/24/[:ci] |
| *PDT80* | tägliche Lieferung - 60  | https://bd.prod.ccs.gematik.solutions/api/v1/sensor/bestand/pdt80/60/[:ci] |
| *PDT83* | stündliche Lieferung - 24| https://bd.prod.ccs.gematik.solutions/api/v1/sensor/bestand/pdt83/24/[:ci] |
| *PDT83* | tägliche Lieferung - 60  | https://bd.prod.ccs.gematik.solutions/api/v1/sensor/bestand/pdt83/60/[:ci] |


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
Der Post-Body ist gemäß der Beschreibung der gemSpec_Perf aus Abschnitt 3.3.2 aufzubauen, wobei nach stündlicher und täglicher Lieferung unterschieden werden muss.
  
Beispielschema der täglichen Ereignisdatenlieferung im Post-Body:
```json
  {
  "abfragezeitpunkt": "2026-02-18T12:00:00Z",
  "ci": "CI-0999999",
  "anzMs": 2,
  "msList":  [{
        "md": "UUIDV",
        "hsv":"version",
        "isIns": 1,
        "dbSize": "",
        "uptime": 3,
        "hasScan": 1,
        "anzScan": 4,
        "anzNu": 5,
        "anzAktNu": 6,
        "anzRa": 7,
        "anzRaOe": 8,
        "profOID": 9999,
        "plz": 99,
         "clientOverview": ["uaPtv": "ProdIdentZ","uaPv":"version","uaKue": """anz":     9}]
         }]} 
  ```

Beispielschema der stündlichen Ereignisdatenlieferung im Post-Body:
   
```json
     {
      "abfragezeitpunkt":"2026-02-18T12:00:00Z",
      "ci":"CI-9999999",
      "md":"550e8400-e29b-41d4-a716-446655440000",
      "anzFed":1,
      "anzVzd":2,
      "anzEmpf":3,
      "anzSend":4,
      "anzRaEe":5,
      "anzInv":6,
      "anzInvF":7,
      "anzEvEe":8,
      "anzEvEeL":9,
      "syncTime":{
         "min":11,
         "max":99,
         "avg":55,
         "Q95":95
      },
      "uploads":{
         "anzUp":9,
         "anzDown":8,
         "min":11,
         "max":99,
         "avg":55
      }
   }
```
 
 Für die Abfrage selbst weiterhin ein beispielhafter curl Request:

<div style="border:1px solid #ccc; padding:10px; border-radius:5px; background:#f9f9f9;">
curl --location "https://bd.dev.ccs.gematik.solutions/api/v1/sensor/bestand/pdt80/60/CI-9999999"^

--header "Content-Type: application/json" ^    
--header "Authorization: eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6..." ^   
--data "@Bestand_pdt80_60.json"
</div>

## 2. Erläuterung der Übertragungsattribute 

Ergänzend zur [A_27699*] sowie [A_27785*] der gemSpec_Perf zur Validierung der Dateninhalte. Die Beschreibung aller weiteren Attribute ist der gemSpec_Perf zu entnehmen.

Als Validierungsregeln gelten für die folgenden Attribute die aufgelisteten Formatbeschreibungen und für diese Fälle ein zwingend notwendig gelieferter Inhalt.

Die **validierten** Lieferattribute ergeben sich wie folgt:

| **Art der Liefersystematik** | **Attribut** | **Beschreibung** |
|:----------|:----------|:----------|
| *täglich* - 24 | Abfragezeitpunkt |  Zeitangabe gemäß ISO 8601 unter expliziter Angabe einer Zeitzone Format YYYY-MM-DDTHH:mm:ss[.fff]Z als String |
| *täglich* - 24   | CI | CI-ID des abgefragten Fachdienstes gemäß [A_17764] als String |
| *täglich* - 24  | anzMs |Anzahl der zum Abfragezeitpunkt instanziierten Messenger-Service als Integer |
| *stündlich* - 60  | Abfragezeitpunkt  | Zeitangabe gemäß ISO 8601 unter expliziter Angabe einer Zeitzone im Format YYYY-MM-DDTHH:mm:ss[.fff]Z als String|
| *stündlich* - 60 | CI   | CI-ID des abgefragten Fachdienstes gemäß [A_17764] als String |
| *stündlich* - 60 | md  | UUIDV4 als Ersatz für die Messenger URL als String|

Alle weiteren Attribute und deren festgelegten Formate sowie Beschreibungen befinden sich unter [A_27699*] und [A_27785*] der gemSpec_Perf und sind einzuhalten.

Attribute welche nicht geliefert werden, werden als leerer Text bzw. Zahl 0 übernommen.

## 3. Aufbau der Response 

Der Aufbau der Response ist in der allgemeinen Dokumentation der Schnittstellen unter 4. "Allgemeiner Aufbau der Response" definiert und erläutert und für alle beschriebenen Fälle gemeinhin gültig. 

Eine allgemeine Auflistung zu den jeweiligen http-Statuscode befindet sich ebenfalls in der Dokumentation unter Abschnitt 4.