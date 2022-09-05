To Get CybsTokenID (New DB Table token_cybs.payment_instrument_id)
```mermaid
    sequenceDiagram
    autonumber
        rect rgb(252, 255, 220)
            PCIPGW->>+CYBS:Call CreateInstrumentIdentifierCard API
            Note right of CYBS: Req: POST /tms/v1/instrumentidentifiers<br>vcMerchantId,cardNo
            CYBS->>-PCIPGW:Return instrumentid
            Note right of CYBS: Res:instrumentId ("id": "7030000000150931111")
        end
        rect rgb(220, 251, 255)
            PCIPGW->>+CYBS:Call CreatePaymentInstrumentCard API
            Note right of CYBS: Req: POST /tms/v1/paymentinstruments<br>vcMerchantId,instrumentId,cardType,expirationMonth,expirationYear
            CYBS->>-PCIPGW:Return paymentInstrumentId/tokenId
            Note right of CYBS: Res:paymentInstrumentId ("id": "E383B9C1D2EF35ECE053AF598E0ADE66")
        end
```

To Get CybsTokenID (timeout)
```mermaid
    sequenceDiagram
    autonumber
        rect rgb(252, 255, 220)
            PCIPGW->>+CYBS: Call CreateInstrumentIdentifierCard API
            Note right of CYBS: Req: POST /tms/v1/instrumentidentifiers<br>vcMerchantId,cardNo
            rect rgb(255, 220, 220)
              CYBS--XPCIPGW: xxx timeout xxx
            end
            PCIPGW->>+CYBS: Call CreateInstrumentIdentifierCard API
            Note right of CYBS: Req: POST /tms/v1/instrumentidentifiers<br>USE THE SAME REQUEST PAYLOAD
            CYBS->>-PCIPGW:Return instrumentid
            Note right of CYBS: Res:instrumentId ("id": "7030000000150931111")
        end
        
        rect rgb(220, 251, 255)
            PCIPGW->>+CYBS: Call CreatePaymentInstrumentCard API
            Note right of CYBS: Req: POST /tms/v1/paymentinstruments<br>vcMerchantId,instrumentId,cardType,expirationMonth,expirationYear
            rect rgb(255, 220, 220)
              CYBS--XPCIPGW: xxx timeout xxx
            end
            PCIPGW->>+CYBS: Call CreatePaymentInstrumentCard API
            Note right of CYBS: Req: POST /tms/v1/paymentinstruments<br>USE THE SAME REQUEST PAYLOAD
            CYBS->>-PCIPGW: Return paymentInstrumentId/tokenId
            Note right of CYBS: Res:paymentInstrumentId ("id": "E383B9C1D2EF35ECE053AF598E0ADE66") the value will be changed
        end
```


Process a Payment
```mermaid
    sequenceDiagram
    autonumber
        rect rgb(252, 255, 220)
            PCIPGW->>+CYBS:Call AuthorizationWithCustomerTokenId API
            Note right of CYBS: Req: POST /pts/v2/payments<br>vcMerchantId,paymentInstrumentId,merchantRef,transactionId,totalAmount,currency
            CYBS->>-PCIPGW:Return payment result
            Note right of CYBS: Res:paymentId ("responseCode": "00","approvalCode": "831000",<BR>"status": "AUTHORIZED","id": "6575302598316189303954")
        end
```

Process a Payment (timeout)
```mermaid
    sequenceDiagram
    autonumber
        rect rgb(252, 255, 220)
            PCIPGW->>+CYBS:Call AuthorizationWithCustomerTokenId API
            Note right of CYBS: Req: POST /pts/v2/payments<br>vcMerchantId,paymentInstrumentId,merchantRef,transactionId,totalAmount,currency
            rect rgb(255, 220, 220)
              CYBS--XPCIPGW: xxx timeout xxx
            end
            
            %% PCIPGW->>+CYBS:Call CreateSearchRequest API
            %% Note right of CYBS: Req: POST /tss/v2/searches<br>vcMerchantId,save,name,merchantRef,offset,limit
            %% CYBS->>-PCIPGW:Return requestId
            %% Note right of CYBS: Res:requestId ("_embedded": { "transactionSummaries": [ {"id": "6575339131026684403954",)
            %% PCIPGW->>+CYBS:Call RetrieveTransaction API
            %% Note right of CYBS: Req: GET /tss/v2/transactions/{requestId}<br>vcMerchantId,requestId
            
            PCIPGW->>+CYBS:Call TimeoutReversal API
            Note right of CYBS: Req: POST /pts/v2/reversals<br>vcMerchantId,transactionId,totalAmount,reason
            CYBS->>-PCIPGW:Return reverse result
            Note right of CYBS: Res:paymentId ("responseCode": "00","approvalCode": "831000",<BR>"status": "REVERSED","id": "6576218032476857203955")
            
            PCIPGW->>+CYBS:Call AuthorizationWithCustomerTokenId API
            Note right of CYBS: Req: POST /pts/v2/payments<br>vcMerchantId,paymentInstrumentId,merchantRef,transactionId,totalAmount,currency
            CYBS->>-PCIPGW:Return payment result
            Note right of CYBS: Res:paymentId ("responseCode": "00","approvalCode": "831000",<BR>"status": "AUTHORIZED","id": "6576217299316245703954")
        end
```

Process a Payment 3DS - Old Solution, Don't use
```mermaid
    sequenceDiagram
    autonumber
        rect rgb(252, 255, 220)
            PCIPGW->>+CYBS:Call SetupCompletionWithTMSToken API
            Note right of CYBS: Req: POST /risk/v1/authentication-setups<br>vcMerchantId,merchantRef,expirationMonth,expirationYear,customerId
            CYBS->>-PCIPGW:Return PA Setup result
            Note right of CYBS: Res: accessToken,referenceId,status,id
            
            Note left of PCIPGW: Return merchant for redirectUrl if the status from above API is Completed
            
            Note right of PCIPGW: Once the merchant/customer go to the redirectUrl, the system will start html code on step below
            PCIPGW->>PCIPGW: Front-End to Submit the Device Data Collection Iframe
            
            PCIPGW->>+CYBS:Call EnrollWithPendingAuthentication API
            Note right of CYBS: Req: POST /risk/v1/authentications<br>vcMerchantId,merchantRef,expirationMonth,expirationYear,customerId,<br>totalAmount,currency,referenceId,returnUrl
            CYBS->>-PCIPGW:Return Enroll result
            Note right of CYBS: Res: challengeRequired,authenticationTransactionId,accessToken,pareq,directoryServerTransactionId,<br>threeDSServerTransactionId,specificationVersion,acsTransactionId,status,id
            
            Note right of CYBS: If it's frictionless,<br> the status will show as success or fail authetication immediately
            
            Note right of CYBS: If it's NOT frictionless,<br> the status will show as PENDING_AUTHENTICATION<br> and continue next step for Step-Up Iframe
            
            PCIPGW->>PCIPGW: Front-End to submit the Step-Up Iframe with width & high as enroll response and display OTP page on html page
            
            Customer->>PCIPGW: Customer to confirm OTP & there will be notify from CYBS to call back returnUrl for transactionId
            
            PCIPGW->>+CYBS:Call ValidateAuthenticationResults API
            Note right of CYBS: Req: POST /risk/v1/authentication-results<br>vcMerchantId,merchantRef,expirationMonth,expirationYear,customerId,<br>totalAmount,currency,authenticationTransactionId
            CYBS->>-PCIPGW:Return VA result
            Note right of CYBS: Res: indicator,authenticationResult,authenticationStatusMsg,cavv,xid,<br>directoryServerTransactionId,threeDSServerTransactionId,specificationVersion,acsTransactionId,status,id
            
            Note right of PCIPGW: if step number 5 or 9 get AUTHENTICATION_SUCCESSFULLY, then continue payment step10-11
            PCIPGW->>+CYBS:Call AuthorizationWithPayerAuthSeparated API
            Note right of CYBS: Req: POST /pts/v2/payments<br>vcMerchantId,paymentInstrumentId,merchantRef,transactionId,totalAmount,<br>currency,authenticationTransactionId
            CYBS->>-PCIPGW:Return payment result
            Note right of CYBS: Res:paymentId ("responseCode": "00","approvalCode": "831000",<BR>"status": "AUTHORIZED","id": "6575302598316189303954", & 3DS details)
        end
```

Process a Payment 3DS (timeout) - Old Solution, Don't use
```mermaid
    sequenceDiagram
    autonumber
        rect rgb(252, 255, 220)
            PCIPGW->>+CYBS:Call SetupCompletionWithTMSToken API
            Note right of CYBS: Req: POST /risk/v1/authentication-setups<br>vcMerchantId,merchantRef,expirationMonth,expirationYear,customerId
            rect rgb(255, 220, 220)
              CYBS--XPCIPGW: xxx timeout xxx
            end
            Note right of CYBS: Return error to merchant
            
            PCIPGW->>PCIPGW: Front-End to Submit the Device Data Collection Iframe
            
            PCIPGW->>+CYBS:Call EnrollWithPendingAuthentication API
            Note right of CYBS: Req: POST /risk/v1/authentications<br>vcMerchantId,merchantRef,expirationMonth,expirationYear,customerId,<br>totalAmount,currency,referenceId,returnUrl
            rect rgb(255, 220, 220)
              CYBS--XPCIPGW: xxx timeout xxx
            end
            Note right of CYBS: Return error to merchant
            
            Customer->>PCIPGW: Customer to confirm OTP
            
            PCIPGW->>+CYBS:Call ValidateAuthenticationResults API
            Note right of CYBS: Req: POST /risk/v1/authentication-results<br>vcMerchantId,merchantRef,expirationMonth,expirationYear,customerId,<br>totalAmount,currency,authenticationTransactionId
            rect rgb(255, 220, 220)
              CYBS--XPCIPGW: xxx timeout xxx
            end
            Note right of CYBS: Return error to merchant
            
            PCIPGW->>+CYBS:Call AuthorizationWithPayerAuthSeparated API
            Note right of CYBS: Req: POST /pts/v2/payments<br>vcMerchantId,paymentInstrumentId,merchantRef,transactionId,totalAmount,<br>currency,authenticationTransactionId
            rect rgb(255, 220, 220)
              CYBS--XPCIPGW: xxx timeout xxx
            end
            
            Note right of PCIPGW: SyncStateProcess
            PCIPGW->>+CYBS:Call TimeoutReversal API
            Note right of CYBS: Req: POST /pts/v2/reversals<br>vcMerchantId,transactionId,totalAmount,reason
            CYBS->>-PCIPGW:Return reverse result
            Note right of CYBS: Res:paymentId ("responseCode": "00","approvalCode": "831000",<BR>"status": "REVERSED","id": "6576218032476857203955")
            
            PCIPGW->>+CYBS:Call AuthorizationWithPayerAuthSeparated API
            Note right of CYBS: Req: POST /pts/v2/payments<br>vcMerchantId,paymentInstrumentId,merchantRef,transactionId,totalAmount,<br>currency,authenticationTransactionId
            CYBS->>-PCIPGW:Return payment result
            Note right of CYBS: Res:paymentId ("responseCode": "00","approvalCode": "831000",<BR>"status": "AUTHORIZED","id": "6575302598316189303954", & 3DS details)
        end
```

Process a Payment 3DS - New Solution 20220905
```mermaid
    sequenceDiagram
    autonumber
        rect rgb(252, 255, 220)            
            Note right of PCIPGW: Follow the as-is 3DS flow, if the authentication successful<br>then authorize the transaction with CYBS payment API
            PCIPGW->>+CYBS:Call AuthorizationWithPayerAuthSeparated API
            Note right of CYBS: Req: POST /pts/v2/payments<br>vcMerchantId,paymentInstrumentId,merchantRef,transactionId,<br>totalAmount,currency,authenticationTransactionId,commerceIndicator,cavv,xid,<br>ucafCollectionIndicator,ucafAuthenticationData,paSpecificationVersion,directoryServerTransactionId
            CYBS->>-PCIPGW:Return payment result
            Note right of CYBS: Res:paymentId ("responseCode": "00","approvalCode": "831000",<BR>"status": "AUTHORIZED","id": "6575302598316189303954", & 3DS details)
        end
```

Process a Payment 3DS (timeout) - New Solution 20220905
```mermaid
    sequenceDiagram
    autonumber
        rect rgb(252, 255, 220)
            Note right of PCIPGW: Follow the as-is 3DS flow, if the authentication successful<br>then authorize the transaction with CYBS payment API
            
            PCIPGW->>+CYBS:Call AuthorizationWithPayerAuthSeparated API
            Note right of CYBS: Req: POST /pts/v2/payments<br>vcMerchantId,paymentInstrumentId,merchantRef,transactionId,<br>totalAmount,currency,authenticationTransactionId,commerceIndicator,cavv,xid,<br>ucafCollectionIndicator,ucafAuthenticationData,paSpecificationVersion,directoryServerTransactionId
            CYBS->>-PCIPGW:Return payment result
            Note right of CYBS: Res:paymentId ("responseCode": "00","approvalCode": "831000",<BR>"status": "AUTHORIZED","id": "6575302598316189303954", & 3DS details)
        end
```

Void a Payment
```mermaid
    sequenceDiagram
    autonumber
        rect rgb(252, 255, 220)
            PCIPGW->>+CYBS:Call VoidPayment API
            Note right of CYBS: Req: POST /pts/v2/payments/{requestId}/voids<br>vcMerchantId,requestId,merchantRef
            CYBS->>-PCIPGW:Return void result
            Note right of CYBS: Res:paymentId ("responseCode": "00","status": "REVERSED")
        end
```

Void a Payment (timeout)
```mermaid
    sequenceDiagram
    autonumber
        rect rgb(252, 255, 220)
            PCIPGW->>+CYBS:Call VoidPayment API
            Note right of CYBS: Req: POST /pts/v2/payments/{requestId}/voids<br>vcMerchantId,requestId,merchantRef
            rect rgb(255, 220, 220)
              CYBS--XPCIPGW: xxx timeout xxx
            end
            PCIPGW->>+CYBS:Call CreateSearchRequest API
            Note right of CYBS: Req: POST /tss/v2/searches<br>vcMerchantId,save,name,merchantRef,offset,limit
            CYBS->>-PCIPGW:Return requestId
            Note right of CYBS: Res:requestId ("_embedded": { "transactionSummaries": [ {"id": "6576224137426631103954",)
            PCIPGW->>+CYBS:Call RetrieveTransaction API
            Note right of CYBS: Req: GET /tss/v2/transactions/{requestId}<br>vcMerchantId,requestId
            CYBS->>-PCIPGW:Return transaction detail result
            Note right of CYBS: Res:detail ("applicationInformation": { "applications": [ {"name": "ics_payment_cancel")<BR>manual update DB for "responseCode": "00","status": "REVERSED"
        end
```

Capture a Payment
```mermaid
    sequenceDiagram
    autonumber
        rect rgb(252, 255, 220)
            PCIPGW->>+CYBS:Call CapturePayment API
            Note right of CYBS: Req: POST /pts/v2/payments/{requestId}/captures<br>vcMerchantId,requestId,merchantRef,totalAmount,currency
            CYBS->>-PCIPGW:Return settle result
            Note right of CYBS: Res:captureId and status ("id": "6575375643626212203954","status": "PENDING")
        end
```

Capture a Payment (timeout)
```mermaid
    sequenceDiagram
    autonumber
        rect rgb(252, 255, 220)
            PCIPGW->>+CYBS:Call CapturePayment API
            Note right of CYBS: Req: POST /pts/v2/payments/{requestId}/captures<br>vcMerchantId,requestId,merchantRef,totalAmount,currency
            rect rgb(255, 220, 220)
              CYBS--XPCIPGW: xxx timeout xxx
            end
            PCIPGW->>+CYBS:Call CreateSearchRequest API
            Note right of CYBS: Req: POST /tss/v2/searches<br>vcMerchantId,save,name,merchantRef,offset,limit
            CYBS->>-PCIPGW:Return requestId
            Note right of CYBS: Res:requestId ("_embedded": { "transactionSummaries": [ {"id": "6575339131026684403954",)
            PCIPGW->>+CYBS:Call RetrieveTransaction API
            Note right of CYBS: Req: GET /tss/v2/transactions/{requestId}<br>vcMerchantId,requestId
            CYBS->>-PCIPGW:Return settle result
            Note right of CYBS: Res:captureId and status ("applicationInformation": { "applications": [ {"name": "ics_bill","status": "PENDING",)
        end
```

STATUS:
AUTHORIZED => payment approval,
PENDING => captured/settled,
REVERSED => voided/reversed

RequestID:
payment_request_id,
capture_request_id,
reverse_request_id
