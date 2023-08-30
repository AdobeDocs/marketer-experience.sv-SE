---
title: Unitary-händelse
description: Det här är en instruktionssida för att simulera[!UICONTROL Unitary event]Typ av resans validering.
source-git-commit: 0a52611530513fc036ef56999fde36a32ca0f482
workflow-type: tm+mt
source-wordcount: '728'
ht-degree: 0%

---


# Unitary-händelse

## Steg som ska följas {#steps-to-follow}

>[!CONTEXTUALHELP]
>id="marketerexp_sampledata_unitaryevent"
>title="Hur du använder?"
>abstract="Följ länken för mer information"

>[!IMPORTANT]
>
>Instruktionerna kan ändras **[!UICONTROL Playbook]** se till att alltid referera till datautsnittet Sample i respektive **[!UICONTROL Playbook]**.

## Förutsättning

* Du måste ha Postman installerat
* Använd Playbook för att skapa instanser som **[!UICONTROL Journey]**, **[!UICONTROL Schemas]**, **[!UICONTROL Segments]**, **[!UICONTROL Messages]** osv.

Skapade resurser visas på `Bill Of Material` Sida

![Produktstruktursida](../assets/bom-page.png)

## Förbered Postman med den samling som krävs

1. Besök **[!UICONTROL Use Case Playbook]** program.
1. Klicka på respektive **[!UICONTROL Playbook]** besök **[!UICONTROL Playbook]** informationssida.
1. Besök **[!UICONTROL Bill Of Material]** sidan och hitta **[!UICONTROL Sample data]** -avsnitt.
1. Ladda ned `postman.json` genom att klicka på respektive knappar i användargränssnittet.
1. Importera `postman.json` i **[!DNL Postman Software]**.
1. Skapa en dedikerad Postman-miljö för den här valideringen (t.ex. `Adobe <PLAYBOOK_NAME>`).

## Hämta IMS-token

>[!NOTE]
>
>Alla miljövariabler är skiftlägeskänsliga, så använd alltid det exakta variabelnamnet.

1. Följ [Autentisera och få åtkomst till Experience Platform API:er](https://experienceleague.adobe.com/docs/experience-platform/landing/platform-apis/api-authentication.html) dokumentation för att generera åtkomsttoken.
1. Lagra åtkomsttokenvärdet i miljövariabler med namnet `ACCESS_TOKEN`.
1. Lagra andra autentiseringsrelaterade värden som `API_KEY`, `IMS_ORG` och `SANDBOX_NAME` i miljövariabler.

>[!IMPORTANT]
>
>Innan du kör ett API från Postman måste alla miljövariabler som krävs läggas till.

## Publicera den resa som skapats av Playbook

Det finns två sätt att publicera resan. Du kan välja något av dem:

1. **Använda AJO-gränssnittet** - klicka på länken Resa på `Bill Of Material Page`; den här dirigerar om dig till sidan Journey där du kan klicka på **[!UICONTROL Publish]** och Journey skulle publiceras.

   ![Reseobjekt](../assets/journey-object.png)

1. **Använda Postman API**

   1. Utlösare **[!DNL Publish Journey]** begäran från **[!DNL Journey Publish]** > **[!DNL Queue journey publish job]**.
   1. Det kan ta en stund att publicera på resa, så för att kontrollera statusen kör du API:t för Kontrollera resestatus fram till `response.status` är `SUCCESS`ska du vänta i 10-15 sekunder om det tar tid att publicera resan.

   >[!NOTE]
   >
   >Alla miljövariabler är skiftlägeskänsliga, så använd alltid det exakta variabelnamnet.

## Infoga kundprofil

>[!TIP]
>
>Du kan återanvända samma e-postadress genom att lägga till `+<variable>` i e-postmeddelandet, t.ex. `usertest@email.com` kan återupptas som `usertest+v1@email.com` eller `usertest+24jul@email.com`. Det är praktiskt att ha en ny profil varje gång, men ändå använda samma e-post-id.

1. Första gången användaren måste skapa **[!DNL customer dataset]** och **[!DNL HTTP Streaming Inlet Connection]**.
1. Om du redan har skapat **[!DNL customer dataset]** och **[!DNL HTTP Streaming Inlet Connection]** går du vidare till steget `5`.
1. Utlösare **[!DNL Customer Profile Ingestion]** > **[!DNL Create Customer Profile InletId]** > **[!DNL Create Dataset]** att skapa **[!DNL customer dataset]**; den här lagrar en `CustomerProfile_dataset_id` i postmansmiljövariabler.
1. Skapa **[!DNL HTTP Streaming Inlet Connection]** använder du Postman API:er under **[!DNL Customer Profile Ingestion > Create Customer Profile InletId]**.

   1. `CustomerProfile_dataset_id` måste vara tillgängligt i postmansmiljövariabler, om inte, se steg `3`.
   1. Utlösare **[!DNL `CREATE Base Connection`]** till [!DNL create base connection].
   1. Utlösare **[!DNL `CREATE Source Connection`]** till [!DNL create source connection].
   1. Utlösare **[!DNL `CREATE Target Connection`]** till [!DNL create target connection].
   1. Utlösare **[!DNL `CREATE Dataflow`]** till [!DNL create dataflow].
   1. Utlösare **[!DNL `GET Base Connection`]**- detta lagrar automatiskt `CustomerProfile_inlet_id` i postmanernas miljövariabler.

1. I det här steget måste du ha `CustomerProfile_dataset_id` och `CustomerProfile_inlet_id` i postmansmiljövariabler; om inte, se steg `3` eller `4` respektive.
1. Användaren måste lagra för att kunna importera kunder `customer_country_code`, `customer_mobile_no`, `customer_first_name`, `customer_last_name` och `email` i postmansmiljövariabler.

   1. `customer_country_code` skulle vara landskoden för mobilnummer, t.ex. `91` eller `1`
   1. `customer_mobile_no` skulle vara mobilnummer, t.ex. `9987654321`
   1. `customer_first_name` skulle vara användarens förnamn
   1. `customer_last_name` skulle vara användarens efternamn
   1. `email` är användarens e-postadress, vilket är avgörande för att kunna använda ett distinkt e-post-ID så att en ny profil kan hämtas.

1. Uppdatera Postman-begäran **[!DNL Customer Ingestion]** > **[!DNL Customer Streaming Ingestion]** för att ändra den kanal kunden föredrar; som standard [!DNL `email`] har konfigurerats i begäran.

   ```js
   "consents": {
       "marketing": {
           "preferred": "email",
           "email": {
               "val": "y"
           },
           "push": {
               "val": "n"
           },
           "sms": {
               "val": "n"
           }
       }
   }
   ```

1. Ändra föredragen kanal till `sms` eller `push` och skapa respektive kanalvärde för `y` och `n` till andra värden, t.ex.

   ```js
   "consents": {
       "marketing": {
           "preferred": "sms",
           "email": {
               "val": "n"
           },
           "push": {
               "val": "n"
           },
           "sms": {
               "val": "y"
           }
       }
   }
   ```

1. Slutligen utlösare **[!DNL `Customer Profile Ingestion > Customer Profile Streaming Ingestion`]** för att importera kundprofilen.

## Ingest-händelse

1. Första gången användaren måste skapa **[!DNL event dataset]** och **[!DNL HTTP Streaming Inlet Connection for events]**
1. Om du redan har skapat **[!DNL event dataset]** och **[!DNL HTTP Streaming Inlet Connection for events]** går du vidare till steget `5`.
1. Utlösare **[!DNL `Schemas Data Ingestion > AEP Demo Schema Ingestion > Create AEP Demo Schema InletId > Create Dataset`]** att skapa **[!DNL event dataset]** lagrar den en `AEPDemoSchema_dataset_id` in postman environment variables
1. Skapa **[!DNL HTTP Streaming Inlet Connection for events]** använder du Postman API:er under **[!DNL Schemas Data Ingestion]** > **[!DNL AEP Demo Schema Ingestion]** > **[!DNL Create AEP Demo Schema InletId]**.

   1. `AEPDemoSchema_dataset_id` måste vara tillgängligt i postmansmiljövariabler, om inte, se steg `3`
   1. Utlösare **[!DNL `CREATE Base Connection`]** till [!DNL create base connection]
   1. Utlösare **[!DNL `CREATE Source Connection`]** till [!DNL create source connection]
   1. Utlösare **[!DNL `CREATE Target Connection`]** till [!DNL create target connection]
   1. Utlösare **[!DNL `CREATE Dataflow`]** till [!DNL create dataflow]
   1. Utlösare **[!DNL `GET Base Connection`]**- detta lagrar automatiskt `AEPDemoSchema_inlet_id` i postmans miljövariabler

1. I det här steget måste du ha `AEPDemoSchema_dataset_id` och `AEPDemoSchema_inlet_id` i postmansmiljövariabler, om de inte är det, se steg `3` eller `4` respektive
1. Användaren måste ändra tidsvariabeln för att kunna importera händelsen `timestamp` i begärande av **[!DNL Schemas Data Ingestion]** > **[!DNL AEP Demo Schema Ingestion]** > **[!DNL AEP Demo Schema Streaming Ingestion]** på postman.

   1. `timestamp` använder den aktuella tidsstämpeln, t.ex. `2023-07-21T16:37:52+05:30` justera tidszonen efter behov.

1. Utlösare **[!DNL Schemas Data Ingestion > AEP Demo Schema Ingestion > AEP Demo Schema Streaming Ingestion]** för att importera händelsen, så att resan kan utlösas

## Slutlig validering

Du måste få ett meddelande om den valda föredragna kanal som används i **[!DNL Ingest the Customer Profile]** steg `8`

* `SMS` om önskad kanal är `sms` på `customer_country_code` och `customer_mobile_no`
* `Email` om önskad kanal är `email` på `email`

Du kan även kontrollera `Journey Report`, för att kontrollera det klickar du på `Journey Object` på `Bill of Materials page` kommer du att omdirigeras till `Journey Details page`.

För varje publicerad reseanvändare måste en **[!UICONTROL View report]** knapp
![Journey Report Page](../assets/journey-report-page.png)


## Rensa

Använd inte flera instanser `Journey` köra samtidigt, vänligen stoppa resan om den endast är till för validering när valideringen är klar.
