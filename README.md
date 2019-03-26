<p align="center">
<img src="https://www.bill.it/documents/100506/373235/BillBanner.png" alt="Bill Banner">
</p>

<p align="center">
[![Version](https://img.shields.io/badge/version-1.0.4-blue.svg)](https://github.com/SisalSpA/bill-consumer-ios-sdk/releases)
[![Platform](https://img.shields.io/badge/platform-iOS-cc9c00.svg)](https://developer.apple.com/ios)
[![License](https://img.shields.io/badge/license-Apache%202-lightgrey.svg)](https://github.com/SisalSpA/bill-consumer-ios-sdk/blob/master/LICENSE)
</p>

---

<p align="center">
    <a href="#features">Features</a> &bull;
    <a href="#requisiti">Requisiti</a> &bull;
    <a href="#installazione-ed-utilizzo">Installazione ed Utilizzo</a> &bull;
    <a href="#ambienti">Ambienti</a> &bull;
    <a href="#documentazione">Documentazione</a> &bull;
    <a href="#changelog">Changelog</a> &bull;
    <a href="#licenza">Licenza</a>
</p>

---

Features
========

Lo strumento di pagamento Bill offre due modalità alternative per completare un pagamento:

-	`App2App`, che tramite un’applicazione esterna consumer permette di inviare all’applicazione Bill una richiesta di pagamento sullo stesso dispositivo
-	`Push2App`, che tramite una notifica push permette di inviare una richiesta di pagamento su dispositivo selezionato dall’utente

Requisiti
=========

| **Versione**  | **iOS** |
|---------------|---------|
| 1.0.0...1.0.4 | 9.0+    |

Installazione ed Utilizzo
=========================

## Manuale
- Scarica il file zip dell'ultima versione da https://github.com/SisalSpA/bill-consumer-ios-sdk/releases
- Importa il framework all'interno del tuo progetto

## CocoaPods
- Aggiungi all'interno del `Podfile` la seguente riga:
  ```ruby
  pod 'BillConsumerSDK'
  ```
- Aprire il terminale ed eseguire il comando `pod install`

Entrambe le soluzioni prevedono l'utilizzo del framework tramite l'importazione all'interno dei file sorgente del progetto.

**Swift**

`import BillConsumerSDK`

**Objective-C**

`#import <BillConsumerSDK/BillConsumerSDK.h>`

Ambienti
========

L'SDK prevede il supporto a più ambienti semplicemente aggiungendo alla file della `partnerKey` o del `transactionTokenOnly` il nome dell'ambiente.

Per l'ambiente di produzione non è necessario aggiungere alcun suffisso.

| **Ambiente** | **Suffisso** |
|--------------|--------------|
| Produzione   |              |
| Test         | `-EXTERNAL`  |
> Se per esempio la chiave è `abcdefgh0123456789` questa diventerà `abcdefgh0123456789-EXTERNAL`.

Documentazione
==============

> Tutte le API ("set" esclusa) prevedono l'utilizzo di block (Objective-C) o closure (Swift) per la gestione di eventuali risposte.
Queste non verranno chiamate sul main thread, rimane quindi responsabilità dello sviluppatore gestire l'esecuzione del codice all'interno dei blocks o closures sul main thread.

## Inizializzazione

All'avvio dell'app o del processo di pagamento è necessario inizializzare l'SDK utilizzando i parametri `Parnter Key`, `Partner ID` e `Partner Secret` che saranno forniti per ogni integrazione.

| **Parametro**   | **Tipo** |
|-----------------|----------|
| `partnerKey`    | String   |
| `partnerID`     | String   |
| `partnerSecret` | String   |

**Swift**

```swift
BillConsumerSDK.shared().set(partnerKey: String, partnerID: String, partnerSecret: String)
```

**Objective-C**

```objective-c
[[BillConsumerSDK sharedInstance] setPartnerKey:partnerKey partnerID:partnerID partnerSecret:partnerSecret];
```

## Richiesta di Pagamento

Il flusso di pagamento prevede che l'applicazione esterna, tramite l'SDK, possa invocare l'applicazione Bill Consumer per effettuare una transazione.

I possibili risultati sono:
-	Impossibilità di proseguire nel pagamento per incompatibilità con l’applicazione Bill Consumer
-	Transazione andata a buon fine
-	Transazione non andata a buon fine con relativo caso di errore da gestire

Il risultato della transazione deve comunque essere verificato tramite la funzionalità di controllo dell'esito della transazione.

### Codice Fiscale e Token di Transazione

Nel caso in cui si sia in possesso di un token di transazione bisogna aggiungere le seguenti chiave-valore nel `Dictionary` per l'API `pay`:

#### Richiesta

| **Parametro**      | **Tipo**        | **Obbligatorio** | **Note** |
|--------------------|-----------------|------------------|----------|
| `vatCode`          | String          | Si               |          |
| `transactionToken` | String          | Si               |          |
| `timeout`          | Int             | No               | Timeout nel caso in cui l’app Bill non sia installata e venga aperta la WebView per completare il pagamento |

#### Risposta

Verrà restituito un `enum` di tipo `BillConsumerSDKResponse` all'interno di una block (Objective-C)/ closure (Swift):

| **Parametro**                 | **Note**                                                            |
|-------------------------------|---------------------------------------------------------------------|
| SDKConsumerGenericError       | Errore generico                                                     |
| SDKConsumerBillOpened         | L'app Bill è installata ed è stata aperta                           |
| SDKConsumerTransactionOK      | La WebView è stata aperta e la transazione è andata a buon fine     |
| SDKConsumerTransactionKO      | La WebView è stata aperta e la transazione non è andata a buon fine |
| SDKConsumerTransactionTimeout | La WebView è stata aperta ma la transazione è andata in timeout     |

#### Esempio di Codice

**Swift**

```swift
BillConsumerSDK.shared().pay([vatCode: vatCode, transactionToken: transactionToken, timeout: timeout])
```

**Objective-C**

```objective-c
[[BillConsumerSDK sharedInstance] payWithParameters:@{kVatCode: vatCode, kTransactionToken: transactionToken, kTimeout: timeout}];
```

### Codice Fiscale e Posizione del Merchant

Nel caso in cui si sia in possesso di una posizione di un merchant o quella attuale dell'utente (longitudine e latitudine) bisogna aggiungere le seguenti chiave-valore nel `Dictionary` per l'API `pay`:

#### Richiesta

| **Parametro** | **Tipo**        | **Obbligatorio** | **Note** |
|---------------|-----------------|------------------|----------|
| `vatCode`     | String          | Si               |          |
| `amount`      | Double          | Si               |          |
| `latitude`    | Double          | Si               |          |
| `longitude`   | Double          | Si               | &nbsp;   |

#### Risposta

Verrà restituito un `enum` di tipo `BillConsumerSDKResponse` all'interno di una block (Objective-C)/ closure (Swift):

| **Parametro**                       | **Note**                                                                                |
|-------------------------------------|-----------------------------------------------------------------------------------------|
| SDKConsumerGenericError             | Errore generico                                                                         |
| SDKConsumerBillInstallationNotValid | L’app non è presente sul dispositivo oppure il device non rispetta gli standard di Bill |
| SDKConsumerNoShop                   | Non è stato trovato alcun shop vicino alle coordinate fornite                           |
| SDKConsumerGenericShopError         | Errore generico con lo shop                                                             |
| SDKConsumerBillOpened               | L'app Bill è installata ed è stata aperta                                               |

#### Esempio di Codice

**Swift**

```swift
BillConsumerSDK.shared().pay([vatCode: vatCode, amount: amount, latitude: latitude, longitude: longitude])
```

**Objective-C**

```objective-c
[[BillConsumerSDK sharedInstance] payWithParameters:@{kVatCode: vatCode, kAmount: amount, kLatitude: latitude, kLongitude: longitude}];
```

### Token di Transazione

Nel caso in cui si sia in possesso del solo token di transazione bisogna aggiungere le seguenti chiave-valore nel `Dictionary` per l'API `pay`:

#### Richiesta

| **Parametro**          | **Tipo**        | **Obbligatorio** | **Note** |
|------------------------|-----------------|------------------|----------|
| `transactionTokenOnly` | String          | Si               | &nbsp;   |

#### Risposta

Verrà restituito un `enum` di tipo `BillConsumerSDKResponse` all'interno di una block (Objective-C)/ closure (Swift):

| **Parametro**                       | **Note**                                                            |
|-------------------------------------|---------------------------------------------------------------------|
| SDKConsumerGenericError             | Errore generico                                                     |
| SDKConsumerBillOpened               | L'app Bill è installata ed è stata aperta                           |
| SDKConsumerTransactionOK            | La WebView è stata aperta e la transazione è andata a buon fine     |
| SDKConsumerTransactionKO            | La WebView è stata aperta e la transazione non è andata a buon fine |
| SDKConsumerTransactionTimeout       | La WebView è stata aperta ma la transazione è andata in timeout     |

#### Esempio di Codice

**Swift**

```swift
BillConsumerSDK.shared().pay([transactionTokenOnly: transactionToken])
```

**Objective-C**

```objective-c
[[BillConsumerSDK sharedInstance] payWithParameters:@{kTransactionTokenOnly: transactionToken}];
```

## Esito della Transazione

Alla conclusione di un pagamento è necessario controllare l'esito della transazione tramite la funzionalità esposta dall'SDK Bill.

I possibili risultati sono:
-	Transazione andata a buon fine
-	Transazione non andata a buon fine
-	Transazione ancora in corso
-	Transazione in timeout

#### Richiesta

| **Parametro**          | **Tipo**        |
|------------------------|-----------------|
| `partnerID`            | String          |
| `transactionToken`     | String          |
| `handler`              | Block / Closure |

#### Risposta

Verrà restituito un `enum` di tipo `BillConsumerSDKResponse` all'interno di una block (Objective-C)/ closure (Swift):

| **Parametro**                   | **Note**                                                            |
|---------------------------------|---------------------------------------------------------------------|
| SDKConsumerGenericError         | Errore generico                                                     |
| SDKConsumerGenericStatusError   | Errore generico con lo stato della transazione                      |
| SDKConsumerTransactionOK        | La WebView è stata aperta e la transazione è andata a buon fine     |
| SDKConsumerTransactionKO        | La WebView è stata aperta e la transazione non è andata a buon fine |
| SDKConsumerTransactionPending   | L'app Bill è installata ed è stata aperta                           |
| SDKConsumerTransactionTimeout   | La WebView è stata aperta ma la transazione è andata in timeout     |

#### Esempio di Codice

**Swift**

```swift
BillConsumerSDK.shared().transactionStatus(partnerID: partnerID, transactionToken: transactionToken, completionHandler: completionHandler)
```

**Objective-C**

```objective-c
[[BillConsumerSDK sharedInstance] transactionStatusWithPartnerID:partnerID withTransactionToken:transactionToken withCompletionHandler:completionHandler];
```

Changelog
=========

To see what has changed in recent versions of BillConsumerSDK, see the **[CHANGELOG.md](https://github.com/SisalSpA/bill-consumer-ios-sdk/blob/master/CHANGELOG.md)** file.

Licenza
=======

BillConsumerSDK is available under the Apache 2.0 license. See the **[LICENSE](https://github.com/SisalSpA/bill-consumer-ios-sdk/blob/master/LICENSE)** file for more info.
