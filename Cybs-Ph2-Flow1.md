As-Is Pay Button
```mermaid
    sequenceDiagram
        Actor Customer
        participant MerchantPortal
        participant WebAPI
        participant 3DS
        participant KPGW
        participant CLK
        Customer->>MerchantPortal: 1.Input card number and select save card
        MerchantPortal->>WebAPI: 2.Create customer 
        
```
