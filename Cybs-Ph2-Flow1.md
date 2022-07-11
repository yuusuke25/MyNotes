Pay and save with Embedded (as-is)
```mermaid
    sequenceDiagram
        rect rgb(255, 220, 220)
            Customer->>Merchant:1.confirm to pay
            Merchant->>+PCIPGW:2.Request with Kasikorn Embedded JS
            PCIPGW->>-Merchant:3.return token_id

            Merchant->>+PCIPGW:4.Call create Charge API (mode=token,tokenid)
            PCIPGW->>-Merchant:5.return chargeid & redirect_url


            Merchant->>3DS:6.Redirect to redirect_url
            3DS->>3DS:7.send and verify OTP
            3DS->>PCIPGW:8.return OTP result with ECI,CAVV

            rect rgb(220, 255, 229)
                PCIPGW->>+KPGW:9.request charge,ECI,CAVV
                KPGW->>+CLK:10.request charge
                CLK->>-KPGW:11.return payment result
                KPGW->>-PCIPGW:12.return payment result
            end

            PCIPGW->>Merchant:13.return chargeid & payment result
        end
        
        rect rgb(252, 255, 220)
            Merchant->>+PCIPGW:1.Call create Customer API (mode=token,tokenid,email) to save card with KPGW
            PCIPGW->>-Merchant:2.return customerid and cardid (for payment next time)
        end

```

Pay and save with Embedded (to-be)
```mermaid
    sequenceDiagram
        rect rgb(255, 220, 220)
            Customer->>Merchant:1.confirm to pay
            Merchant->>+PCIPGW:2.Request with Kasikorn Embedded JS
            PCIPGW->>-Merchant:3.return token_id
            Merchant->>+PCIPGW:4.Call create Charge API (mode=token,tokenid)
            PCIPGW->>-Merchant:5.return chargeid & redirect_url

            Merchant->>3DS:6.Redirect to redirect_url
            3DS->>3DS:7.send and verify OTP
            3DS->>PCIPGW:8.return OTP result with ECI,CAVV

            Note left of PCIPGW: Check & flag value for isInternalCharge

            rect rgb(220, 255, 229)
                Note right of PCIPGW: isInternalCharge=true
                PCIPGW->>+KPGW:9.request charge,ECI,CAVV
                KPGW->>+CLK:10.request charge
                CLK->>-KPGW:11.return payment result
                KPGW->>-PCIPGW:12.return payment result
            end

            rect rgb(220, 251, 255)
                Note right of PCIPGW: isInternalCharge=false
                PCIPGW->>+CYBS:9.Request Create CybsTokenID
                CYBS->>-PCIPGW:10.return CybsTokenID
                PCIPGW->>+CYBS:11.Payment with CybsTokenID,ECI,CAVV
                CYBS->>-PCIPGW:12.Return payment result
            end

            PCIPGW->>Merchant:13.return chargeid & payment result
        end
        
        rect rgb(252, 255, 220)
            Merchant->>+PCIPGW:1.Call create Customer API (mode=token,tokenid,email) to save card with KPGW
            PCIPGW->>-Merchant:2.return customerid and cardid (for payment next time)
        end

```


Add card (as-is)
```mermaid
    sequenceDiagram
        rect rgb(252, 255, 220)
            Customer->>Merchant:1.confirm to add card on Kbank UI
            Merchant->>+PCIPGW:2.KbankUI call create Token API (mode=fullpan,card_number,card_exp,cvv)
            PCIPGW->>-Merchant:3.return tokenid

            Merchant->>+PCIPGW:4.Call create Charge API (mode=register3d,token_id)
            PCIPGW->>-Merchant:5.return chargeid & redirect_url

            Merchant->>3DS:6.Redirect to redirect_url
            3DS->>3DS:7.send and verify OTP
            3DS->>PCIPGW:8.return OTP result with ECI,CAVV

            rect rgb(220, 255, 229)
                PCIPGW->>+KPGW:9.request charge,ECI,CAVV (amt 0 for VS,MC)
                KPGW->>+CLK:10.request charge
                CLK->>-KPGW:11.return payment result
                KPGW->>-PCIPGW:12.return payment result
            end

            PCIPGW->>Merchant:13.return charge_id & payment result

            Merchant->>+PCIPGW:14.Call create Customer API (mode=token,token_id,email)
            PCIPGW->>-Merchant:15.return customerid and cardid
        end
        
        rect rgb(255, 220, 220)
            Merchant->>+PCIPGW:1.Call create Charge API (mode=customer,customer_id,card_id)
                PCIPGW->>+KPGW:2.request charge
                KPGW->>+CLK:3.request charge
                CLK->>-KPGW:4.return payment result
                KPGW->>-PCIPGW:5.return payment result
            PCIPGW->>-Merchant:6.return chargeid, transaction_state, transaction_status
        end
```

Add card (to-be)
```mermaid
    sequenceDiagram
        rect rgb(252, 255, 220)
            Customer->Merchant:1.confirm to add card on Kbank UI
            Merchant->>+PCIPGW:2.KbankUI call create Token API (mode=fullpan,card_number,card_exp,cvv)
            PCIPGW->>-Merchant:3.return token_id

            Merchant->>+PCIPGW:4.Call create Charge API (mode=register3d,token_id)
            PCIPGW->>-Merchant:5.return chargeid & redirect_url

            Merchant->>3DS:6.Redirect to redirect_url
            3DS->3DS:7.send and verify OTP
            3DS->PCIPGW:8.return OTP result with ECI,CAVV
            
            Note left of PCIPGW: Check & flag value for isInternalCharge
            rect rgb(220, 255, 229)
                Note right of PCIPGW: isInternalCharge=true
                PCIPGW->>+KPGW:9.request charge,ECI,CAVV (amt 0 for VS,MC)
                KPGW->>+CLK:10.request charge
                CLK->>-KPGW:11.return payment result
                KPGW->>-PCIPGW:12.return payment result
            end
            rect rgb(220, 251, 255)
                Note right of PCIPGW: isInternalCharge=false
                PCIPGW->>+CYBS:9.Request Create CybsTokenID
                CYBS->>-PCIPGW:10.return CybsTokenID
                PCIPGW->>+CYBS:11.Payment with CybsTokenID,ECI,CAVV
                CYBS->>-PCIPGW:12.Return payment result
            end
            
            PCIPGW->>Merchant:13.return chargeid & payment result

            Merchant->>+PCIPGW:14.Call create Customer API (mode=token,token_id,email)
            PCIPGW->>-Merchant:15.return customer_id and card_id <br>(which is mapped with CybsTokenID)
        end
        
        rect rgb(255, 220, 220)
            Merchant->>PCIPGW:1.Call create Charge API (mode=customer,customer_id,card_id)
            Note left of PCIPGW: Check value for isInternalCharge
            rect rgb(220, 255, 229)
                Note right of PCIPGW: isInternalCharge=true
                PCIPGW->>+KPGW:2.request charge
                KPGW->>+CLK:3.request charge
                CLK->>-KPGW:4.return payment result
                KPGW->>-PCIPGW:5.return payment result
            end
            rect rgb(220, 251, 255)
                Note right of PCIPGW: isInternalCharge=false
                PCIPGW->>+CYBS:2.Payment with CybsTokenID,ECI,CAVV
                CYBS->>-PCIPGW:3.Return payment result
            end
            PCIPGW->>Merchant:6.return chargeid, transaction_state, transaction_status
        end
```


Create Customer (as-is)
```mermaid
    sequenceDiagram
        
        rect rgb(252, 255, 220)
            Merchant->>+PCIPGW:1.Call create customer API (mode=fullpan)
            PCIPGW->>+KPGW:2.request charge (amt 0 for VS,MC)
            KPGW->>+CLK:3.request charge 0 THB
            CLK->>-KPGW:4.return payment result
            KPGW->>-PCIPGW:5.return payment result
            PCIPGW->>-Merchant:6.return customer_id and card_id
        end
        
        rect rgb(255, 220, 220)
            Merchant->>+PCIPGW:1.Call create Charge API (mode=customer,customer_id,card_id)
            PCIPGW->>+KPGW:2.request charge
            KPGW->>+CLK:3.request charge
            CLK->>-KPGW:4.return payment result
            KPGW->>-PCIPGW:5.return payment result
            PCIPGW->>-Merchant:6.return charge_id, transaction_state, transaction_status
        end
```


Create Customer (to-be)
```mermaid
    sequenceDiagram
        rect rgb(252, 255, 220)
            Merchant->>+PCIPGW:1.Call create Customer API (mode=fullpan)
            Note left of PCIPGW: Check & flag value for isInternalCharge
            rect rgb(220, 255, 229)
                Note right of PCIPGW: isInternalCharge=true
                PCIPGW->>+KPGW:2.request charge (amt 0 for VS,MC)
                KPGW->>+CLK:3.request charge 0 THB
                CLK->>-KPGW:4.return payment result
                KPGW->>-PCIPGW:5.return payment result
            end
            rect rgb(220, 251, 255)
                Note right of PCIPGW: isInternalCharge=false
                PCIPGW->>+CYBS:2.Request Create CybsTokenID
                CYBS->>-PCIPGW:3.return CybsTokenID
            end
            PCIPGW->>-Merchant:6.return customer_id and card_id<br>(which is mapped with CybsTokenID)
        end
        
        rect rgb(255, 220, 220)
            Merchant->>+PCIPGW:1.Call create Charge API (mode=customer,customer_id,card_id)
            Note left of PCIPGW: Check value for isInternalCharge
            rect rgb(220, 255, 229)
                Note right of PCIPGW: isInternalCharge=true
                PCIPGW->>+KPGW:2.request charge
                KPGW->>+CLK:3.request charge
                CLK->>-KPGW:4.return payment result
                KPGW->>-PCIPGW:5.return payment result
            end
            rect rgb(220, 251, 255)
                Note right of PCIPGW: isInternalCharge=false
                PCIPGW->>+CYBS:2.Payment with CybsTokenID
                CYBS->>-PCIPGW:3.Return payment result
            end
            PCIPGW->>-Merchant:6.return charge_id, transaction_state, transaction_status
        end
```


