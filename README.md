# pagopa-mock-ec

A mock implementation of PA pagoPA service

- [pagopa-mock-ec](#pagopa-mock-ec)
  - [Glossary](#glossary)
  - [Usage](#usage)
  - [Prerequisites](#prerequisites)
  - [Functionalities](#functionalities)
    - [Tribute description](#tribute-description)
    - [Configurations](#configurations)
  - [Certificate creation :](#certificate-creation-)
    - [generate private key and CSR](#generate-private-key-and-csr)
    - [verify the information contained in the CSR](#verify-the-information-contained-in-the-csr)
    - [create pfx file for azure](#create-pfx-file-for-azure)
  - [Developer section 💻](#developer-section-)
    - [Prerequisites](#prerequisites-1)
    - [Environment](#environment)
    - [Check mockEcService 🧪](#check-mockecservice-)

## Glossary

| Acronym  | Description            |
| -------- | ---------------------- |
| EC or PA | Public Admninistration |
| CCPost   | Postal account         |
| CCBank   | Bank account           |

<br>

## Usage
## Prerequisites

To be able to use as PSP the following `pagopa-mock-ec` remebers to configure in your requests : 

- PSP [Identification and Authentication](https://pagopa.github.io/pagopa-api/#section/Introduction/Identification-and-Authentication
)
- as EC/PA the `77777777777` fiscalCode
- as `noticeNumber` one of those described in to below section [Configurations](#Configurations)



## Functionalities

The following functionalites are available (EC Side) 
>(_see [here](https://pagopa.github.io/pagopa-api/indexPA.html) to details_)
- *paVerifyPaymentNotice*
- *paGetPayment*
- *paSendRT*

These mock functionalities allows the PSP to invoke all the payment steps 
> (_see [here](https://pagopa.github.io/pagopa-api/) to details_)
- *verifyPaymentNotice* or *verificaBollettino*
- *activatePaymentNotice* 
- *sendPaymentOutcome*

### Tribute description
The tribute is 120 EUR divided in 2 transfer: 

- Transfer 1 : TARI, 100€ due to **EC_TE**
- Transfer 2 : TEFA, 20€ due to **Comune di Milano**

With this mock are modelled both : 
1. multibeneficiary notice (comprehensive of TARI and TEFA) 
2. monobeneficiary notice (only TARI) 

### Configurations

Both EC have at their disposal a bank IBAN and a postal IBAN.
Based on notice number the mock will reply with a certain configuration of the tribute.


| Notice number        | CC EC_TE | CC Comune di Milano | Notes                          |
| -------------------- | -------- | ------------------- | ------------------------------ |
| 302**00**xxxxxxxxxxx | CCPost   | CCPost              | multibeneficiary (TARI + TEFA) |
| 302**01**xxxxxxxxxxx | CCPost   | CCBank              | multibeneficiary (TARI + TEFA) |
| 302**02**xxxxxxxxxxx | CCBank   | CCPost              | multibeneficiary (TARI + TEFA) |
| 302**03**xxxxxxxxxxx | CCBank   | CCBank              | multibeneficiary (TARI + TEFA) |
| 302**04**xxxxxxxxxxx | CCPost   | n.a.                | monobeneficiary (TARI)         |
| 302**05**xxxxxxxxxxx | CCBank   | n.a.                | monobeneficiary (TARI)         |


The following edge cases are available (stateless, based on notice number)

| Notice number        | Description     |
| -------------------- | --------------- |
| 302**99**xxxxxxxxxxx | Payment expired |
| 302**YY**xxxxxxxxxxx | Payment unknown |

**_NOTE:_**  YY: every code not mentioned before -> from 06 to 98

<br>

## Certificate creation :

### generate private key and CSR
```sh
openssl req -new -config cert_config.cfg -newkey rsa:2048 -nodes -keyout mockecservice.key -out mockecservice.csr
```
### verify the information contained in the CSR

```sh
openssl req -noout -text -in mockecservice.csr

```
### create pfx file for azure

```sh
openssl pkcs12 -export -out mockecservice.pfx -inkey mockecservice.key -in mockecservice.crt
```

Then [Import a certificate using Azure CLI](https://docs.microsoft.com/en-us/azure/key-vault/certificates/tutorial-import-certificate#import-a-certificate-using-azure-cli)

## Developer section 💻
This _optional_ section is usefull if you want run `pagopa-mock-ec` locally 🚀
<details>
  <summary>Click to expand!</summary>  

> **NOTE** : you can customize partial mock information using `.env.example`. To do that typing `cp .env.example .env` and changes the info you deem appropriate 


### Prerequisites

- [yarn](https://yarnpkg.com/) installed

```sh
yarn install
yarn build && yarn start
```
### Environment

  | name              | description                 | default            |
  | ----------------- | --------------------------- | ------------------ |
  | WINSTON_LOG_LEVEL | desired log level           | `debug`            |
  | PAGOPA_NODO_HOST  | host this server listens to | `http://localhost` |
  | PORT              | host this server listens to | 8089               |
  | BASE_PATH         | `host:port\<BASE_PATH>`     | `mockEcService`    |
  
### Check mockEcService 🧪

From `resources` folder run script `./run_all_mock_scenarios.sh` to execute all scenario.

Otherwise if you just want to see if everything is up run typing the following command on a terminal 
>always from `resources` folder
```sh
curl -H "Content-Type: text/xml; charset=utf-8" -H "SOAPAction:paVerifyPaymentNoticeReq"  -d @paVerifyPaymentNoticeReq_avv1.xml -X POST http://localhost:8089/mockEcService
```

if all rights you'd see something like that 👍

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:paf="http://pagopa-api.pagopa.gov.it/pa/paForNode.xsd">
  <soapenv:Header />
  <soapenv:Body>
    <paf:paVerifyPaymentNoticeRes>
      <outcome>OK</outcome>
      <paymentList>
        <paymentOptionDescription>
          <amount>120.00</amount>
          <options>EQ</options>
          <dueDate>2021-07-31</dueDate>
          <detailDescription>pagamentoTest</detailDescription>
          <allCCP>true</allCCP>
        </paymentOptionDescription>
      </paymentList>
      <paymentDescription>Pagamento di Test</paymentDescription>
      <fiscalCodePA>77777777777</fiscalCodePA>
      <companyName>companyName</companyName>
      <officeName>officeName</officeName>
    </paf:paVerifyPaymentNoticeRes>
  </soapenv:Body>
</soapenv:Envelope>
```

</details>
