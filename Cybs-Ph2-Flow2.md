Settlement (cybs)
```mermaid
    sequenceDiagram
        PCIPGW->>CYBS:1.Payment with CybsTokenID,ECI,CAVV
        CYBS->>PCIPGW:2.Return payment result
        
        alt manual settlement
          Merchant->>PCIPGW:3a.Request settlement
          loop txn by txn
            PCIPGW->>CYBS:4a.Capture a payment
          end
          PCIPGW->>Merchant:5a.Return settlement result
        else auto settlement
          Batch->>PCIPGW:3bstart settlement at xx:xx pm
          loop txn by txn
            PCIPGW->>CYBS:4b.Capture a payment
          end
        end
```


void (cybs)
```mermaid
    sequenceDiagram
        Merchant_Portal->>PCIPGW_BE:0.Request by Void Button
        PCIPGW_BE->>CYBS:1.Void a payment
        CYBS->>PCIPGW:2.Return void result
        
        Bank_Portal->>PCIPGW_BE:0.Request by Void Button
        PCIPGW_BE->>CYBS:1.Void a payment
        CYBS->>PCIPGW:2.Return void result
```

Refund (cybs)
```mermaid
    sequenceDiagram
        Merchant_Portal->>PCIPGW_BE:0.Request by Refund Button
        PCIPGW_BE->>KPGW:1.Refund a settled transaction
        KPGW->>PCIPGW:2.Return Refund result
        
        Bank_Portal->>PCIPGW_BE:0.Request by Refund Button
        PCIPGW_BE->>KPGW:1.Refund a settled transaction
        KPGW->>PCIPGW:2.Return Refund result
        
        Note over Merchant_Portal,KPGW: KPGW batch job to process refund at xx:xx pm
        KPGW->>CLK:1.Request Refund
        CLK->>2.Return Refund result
```
