# 1. Allgemeine Anforderungen der Schnittstellen

Die folgende allgemeine Beschreibung dient zur Ergänzung der Spezifikation und grundlegenden Regelungen in der gemSpec_Perf "Ereignisdaten" ([Link](https://gemspec.gematik.de/docs/gemSpec/gemSpec_Perf/latest/#2.5.7)).

Nachfolgend sind die konkreten Endpunkte, die Authentifizierung sowie Ein- und Ausgaben der Schnittstellen zunächst allgemein und jeweils in einer spezifischen technischen Beschreibung beschrieben.


# 2. Datentypen und Endpunkte

Ereignisdaten "erfassen den Zustand von Anwendungsfällen" entsprechend und müssen im vorgegebenen Format "regelmäßig selbstständig und automatisiert" an die Ereignisdatenschnittstellen geliefert werden.

Format, Lieferintervalle sowie Verhalten bei fehlgeschlagener Lieferung werden ebenfalls in dem genannten Kapitel definiert.

Eine detaillierte technische Beschreibung der Ereignisdatenschnittstellen kann den nachfolgenden Links entnommen werden.
Die weiteren Beschreibungen auf dieser Seite gelten für alle Endpunkte der Ereignisdaten.

| Ereignisdatenschnittstelle | technische Beschreibung | Referenz gemSpec_Perf  (ab Version 2.41)|
|----------|----------|----------|
| eHealth-CardLink | [Link: eHealth-CardLink ](./eHealth-CardLink.md) | 3.23 eHealth-CardLink (PDT77) |
| TI-Messenger Fachdienst | [Link: TI-Messenger](./TI-Messenger.md) | 3.3 TI-Messenger Fachdienst (TI-M) (PDT80, PDT83)  |


# 3. Authentifizierung und Request

Um Daten an die REST-API senden zu können ist eine Authentifizierung mittels OAuth – konkret OAuth 2.0 "client credentials grant flow - notwendig. 

Dies läuft in zwei Schritten ab: 

1. Abruf des Id-Token 
2. Nutzung des Token für REST-API Request

Eine X.509 Identität wie in gemSpec_Perf mit A_25259sowie gemSpec_Krypt mit GS-A_4384-03 beschrieben ist hier nicht erforderlich.

## 3.1 Abruf eines Id-Tokens

Für die Verwaltung von Applikationen und deren Berechtigung wird ein von der gematik betriebenes Entra-Id (früher Azure AD) Verzeichnis verwendet. Für die Generierung eines Id-Token sind folgende Parameter notwendig:
- Tenant-Id
- Client-Id / App-Id
- Scope
- Client-Secret

Diese werden von der gematik zur Verfügung gestellt. 


Daraus ergibt sich ein Abruf mit folgendem Schema aus URL, http -Methode, -Version und -Header, sowie den entsprechenden Parametern:
- URL:

  | Übersicht:| |
  |:----------|:----------|
  | *Basis* | https://login.microsoftonline.com/[:tenant-id]/oauth2/v2.0/token |
  | *URL-Parameter*   | tenant-id => wird von der gematik bereitgestellt  | 
  | *Beispiel:*  | https://login.microsoftonline.com/12345678-abcd-4321-dcba-123456789abc/oauth2/v2.0/token|

- http-Methode: 
  - POST
- http-Version: 
  - entspricht 1.1
- http-Header:
  - Content-Type: application/x-www-form-urlencoded
- Request Body:
  - client_id
  - scope
  - client_secret
  - grant_type => ist immer „client_credentials“

Bei Nutzung der Parameter ist auf ein korrektes URL Encoding zu achten. Die entsprechende Entra-Id Dokumentation zu diesem Schritt befindet sich unter:

https://learn.microsoft.com/en-us/entra/identity-platformv2-oauth2-client-creds-grant-flow#get-a-token

Ein erfolgreicher Abruf liefert eine Rückgabe in folgendem Format:

```json
{"token_type": "Bearer","expires_in": 3599,"access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNBVGZNNXBP..."
```

# 3.2 Nutzung des Tokens für REST-API Request

Der von Entra-Id ausgegebene Token kann nun für Anfragen gegen die REST-API verwendet werden. Der Inhalt des Attributes „*access_token*“ wird in den „*authorization*“ Header übernommen. 

Der Token ist ab Generierung durch die Entra-Id für 60 Minuten gültig. 

Der Zeitraum der Gültigkeit muss selbstständig verwaltet werden. Ein zeitlich abgelaufener Token wird von der REST-API mit http-Statuscode 401 beantwortet. Ein Token sollte innerhalb seiner zeitlichen Gültigkeit jedoch für alle REST-API Anfragen wiederverwendet werden.

# 4. Allgemeiner Aufbau der Response 

Zu den beschriebenen Request ergibt sich eine Response, welche aus den folgenden Komponenten besteht:

- **http-Statuscode:** 
  - Erläuterung in nachstehender Tabelle 
- **http-Header:** 
  - x-ratelimit-limit–max. Anzahl an Anfragen im gleitenden Zeitintervall
  - x-ratelimit-remaining–verbleibende Anzahl von Anfragen im gleitenden Zeitintervall
  - x-ratelimit-reset–Dauer des Zeitintervalls in Millisekunden
- **Response Body:**
  - enthält folgende Attribute:
    - errorMessage –> Erläuterung zu Fehlerfällen, für erfolgreiche Requests ist der Text leer
  - statusCode –> gleicht dem http-Statuscode

Der "Response Body" bildet entsprechend der Attribute das Ergebnis ab und kann zur Deutung herangezogen werden. Im Folgenden werden einzelne Beispiele von zu erwartenden Antworten zu Datenlieferungen und eine Tabelle zu http-Statuscodes zur Unterstützung der Ergebnisdeutung und Handlungsempfehlung dargestellt.

Beispiele von Datenlieferung und entsprechend zu erwartender Response:

- Lieferung eines zulässigen JSON-Formates:
  ```bash
  {"errorMessage":"","statusCode":200}
  ```
- Lieferungen eines unzulässigen JSON-Formates: 
  ```bash
  {"statusCode":400,"message":"body not parsable"}
  ```
- Beispiel für Lieferungen mit mehr als einer unzulässigen Datenangabe:
  ```bash
  {"statusCode":400,"message":"missing fields: msList"}
  ```
- Lieferungen mit unzulässigen Dateninhalten:
  ```bash
  {"errorMessage": "content errors: timestemp is not an integer","statusCode": 400}
  ```
  ```bash
  {"statusCode":400,"message":"content errors: ci is not a valid CI-ID format (CI-0000000)"}
  ```
  ```bash
  {"statusCode":400,"message":"content errors: abfragezeitpunkt has not the specified timestamp format (YYYY-MM-DDTHH:mm:ss[.fff]Z) or is an invalid value,ci value from body and url are not matching"}
  ```

**Allgemeine Übersicht zu http-Statuscodes**

| **Status** | **Beschreibung** | **autom. Retry** |
|:----------|:----------|:----------|
| *200* | Request erfolgreich |nein|
| *400* | Unterschiedliche Ursachen: <br> -> Request Body konnte nicht verarbeitet <br> -> Wert des „content-length“ Header stimmt nicht mit übermittelter Datengröße im Body überein werden <br> -> Wert des „content-type“ Header stimmt nicht mit erwartetem Wert überein <br> -> fehlende Attribute innerhalb des Body (Details können dem Response Body entnommen werden) <br> -> im Body enthaltene Attribute sind nicht valide, z.B. falscher Datentyp (Details können dem Response Body entnommen werden) | nein |
| *401* | Authentifizierung anhand Header-Attribut nicht erfolgreich | nein |
| *403* | Autorisierung anhand Header-Attribut nicht erfolgreich | nein |
| *404* | unbekannte API Route oder falsche http-Methode verwendet | nein |
| *406* | "content-type" Header nicht vorhanden | nein |
| *411* | "content-length" Header nicht vorhanden | nein |
| *413* | max. Größe des Body überschritten | nein |
| *429* | Rate-Limit überschritten | ja (nach entsprechender Wartezeit) |
| *500* | Request aus anderen Gründen nicht erfolgreich | nein |
| *502* | für Proxy & Loadbalancer ist temporär das dahinterliegende Ziel nicht erreichbar | ja |


# 5. Cipher und TLS Versionen

| **TLS Version** | **Cipher** | **gemSpec_Krypt** | **BSI TR-02102-2** |
|:----------|:----------|:----------|:----------|
| TLS 1.2 | TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384 | | &#9989; |
| TLS 1.2 | TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256 | | &#9989; |
| TLS 1.2 | TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 | &#9989;| &#9989;|
| TLS 1.2 | TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 | &#9989;| &#9989;|
| TLS 1.3 | TLS_AES_128_GCM_SHA256 | &#9989; | &#9989;|
| TLS 1.3 | TLS_AES_256_GCM_SHA384 | | &#9989; |

