Pay and save with Embedded (as-is)
```mermaid
    sequenceDiagram
    autonumber
        rect rgb(255, 220, 220)
            %% Customer->>Merchant:confirm to pay
            Merchant->>+PCIPGW:Request with Kasikorn Embedded JS
            PCIPGW->>-Merchant:return token_id

            Merchant->>+PCIPGW:Call create Charge API (mode=token,tokenid)
            PCIPGW->>-Merchant:return chargeid & redirect_url


            Merchant->>+PCIPGW:Redirect to redirect_url
            PCIPGW->>+3DS:Redirect to redirect_url
            3DS->>3DS:send and verify OTP
            3DS->>-PCIPGW:return OTP result with ECI,CAVV

            rect rgb(220, 255, 229)
                PCIPGW->>+KPGW:request charge,ECI,CAVV
                KPGW->>+CLK:request charge
                CLK->>-KPGW:return payment result
                KPGW->>-PCIPGW:return payment result
            end

            PCIPGW->>-Merchant:return chargeid & payment result
        end
        autonumber 1
        rect rgb(252, 255, 220)
            Merchant->>+PCIPGW:Call create Customer API (mode=token,tokenid,email) to save card with KPGW
            PCIPGW->>-Merchant:return customerid and cardid (for payment next time)
        end

```

Pay and save with Embedded (to-be)
```mermaid
    sequenceDiagram
        autonumber 1
        rect rgb(255, 220, 220)
            %% Customer->>Merchant:confirm to pay
            Merchant->>+PCIPGW:Request with Kasikorn Embedded JS
            PCIPGW->>-Merchant:return token_id
            Merchant->>+PCIPGW:Call create Charge API (mode=token,tokenid)
            PCIPGW->>-Merchant:return chargeid & redirect_url

            Merchant->>+PCIPGW:Redirect to redirect_url
            PCIPGW->>+3DS:Redirect to redirect_url
            3DS->>3DS:send and verify OTP
            3DS->>-PCIPGW:return OTP result with ECI,CAVV

            rect rgb(220, 255, 229)
                PCIPGW->>+KPGW:request charge,ECI,CAVV
                KPGW->>+CLK:request charge
                CLK->>-KPGW:return payment result
                KPGW->>-PCIPGW:return payment result
            end
            
            PCIPGW->>-Merchant:return chargeid & payment result
        end
        autonumber 1
        rect rgb(252, 255, 220)
            Merchant->>+PCIPGW:Call create Customer API (mode=token,tokenid,email) to save card with KPGW
              Note left of PCIPGW: Check MID have comp config to active cybs (for e-com)
              rect rgb(220, 251, 255)
                  PCIPGW->>+CYBS:Request Create CybsTokenID
                  CYBS->>-PCIPGW:return CybsTokenID
                  Note right of PCIPGW: save CybsTokenID into new table and FK from table charge trx_id(pm)
              end
            PCIPGW->>-Merchant:return customerid and cardid (for payment next time)
        end

```


Add card (as-is)
```mermaid
    sequenceDiagram
        autonumber 1
        rect rgb(252, 255, 220)
            && Customer->>Merchant:confirm to add card on Kbank UI
            Merchant->>+PCIPGW:KbankUI call create Token API (mode=fullpan,card_number,card_exp,cvv)
            PCIPGW->>-Merchant:return tokenid

            Merchant->>+PCIPGW:Call create Charge API (mode=register3d,token_id)
            PCIPGW->>-Merchant:return chargeid & redirect_url

            Merchant->>+PCIPGW:Redirect to redirect_url
            PCIPGW->>+3DS:Redirect to redirect_url
            3DS->>3DS:send and verify OTP
            3DS->>-PCIPGW:return OTP result with ECI,CAVV

            rect rgb(220, 255, 229)
                PCIPGW->>+KPGW:request charge,ECI,CAVV (amt 0 for VS,MC)
                KPGW->>+CLK:request charge
                CLK->>-KPGW:return payment result
                KPGW->>-PCIPGW:return payment result
            end

            PCIPGW->>-Merchant:return charge_id & payment result

            Merchant->>+PCIPGW:Call create Customer API (mode=token,token_id,email)
            PCIPGW->>-Merchant:return customerid and cardid
        end
        autonumber 1
        rect rgb(255, 220, 220)
            Merchant->>+PCIPGW:Call create Charge API (mode=customer,customer_id,card_id)
                PCIPGW->>+KPGW:request charge
                KPGW->>+CLK:request charge
                CLK->>-KPGW:return payment result
                KPGW->>-PCIPGW:return payment result
            PCIPGW->>-Merchant:return chargeid, transaction_state, transaction_status
        end
```

Add card (to-be)
```mermaid
    sequenceDiagram
        autonumber 1
        rect rgb(252, 255, 220)
            && Customer->>Merchant:confirm to add card on Kbank UI
            Merchant->>+PCIPGW:KbankUI call create Token API (mode=fullpan,card_number,card_exp,cvv)
            PCIPGW->>-Merchant:return token_id

            Merchant->>+PCIPGW:Call create Charge API (mode=register3d,token_id)
            PCIPGW->>-Merchant:return chargeid & redirect_url

            Merchant->>+PCIPGW:Redirect to redirect_url
            PCIPGW->>+3DS:Redirect to redirect_url
            3DS->>3DS:send and verify OTP
            3DS->>-PCIPGW:return OTP result with ECI,CAVV
            
            rect rgb(220, 255, 229)
                PCIPGW->>+KPGW:request charge,ECI,CAVV (amt 0 for VS,MC)
                KPGW->>+CLK:request charge
                CLK->>-KPGW:return payment result
                KPGW->>-PCIPGW:return payment result
            end
            
            PCIPGW->>-Merchant:return chargeid & payment result

            Merchant->>+PCIPGW:Call create Customer API (mode=token,token_id,email)
              Note left of PCIPGW: Check MID have comp config to active cybs (for e-com)
              rect rgb(220, 251, 255)
                  PCIPGW->>+CYBS:Request Create CybsTokenID
                  CYBS->>-PCIPGW:return CybsTokenID
                  Note right of PCIPGW: save CybsTokenID into new table and FK from table charge trx_id(pm)
              end
            PCIPGW->>-Merchant:return customer_id and card_id <br>(which is mapped with CybsTokenID)
        end
        autonumber 1
        rect rgb(255, 220, 220)
            Merchant->>+PCIPGW:Call create Charge API (mode=customer,customer_id,card_id)
              alt
                rect rgb(220, 251, 255)
                  Note right of PCIPGW: Check MID have comp config to active cybs (for e-com)
                  Note right of PCIPGW: AND matched with BP Config for card_brand(Visa,MC,JCB), card_type(credit,debit), onus/offus
                  Note right of PCIPGW: AND also found CybsTokenID in the database (token_cybs.payment_instrument_id)
                    PCIPGW->>+CYBS:Payment with CybsTokenID
                    CYBS->>-PCIPGW:Return payment result
                end
              else
                rect rgb(220, 255, 229)
                  Note right of PCIPGW: does not meet the conditions
                    PCIPGW->>+KPGW:request charge
                    KPGW->>+CLK:request charge
                    CLK->>-KPGW:return payment result
                    KPGW->>-PCIPGW:return payment result
                end
              end

            PCIPGW->>-Merchant:return chargeid, transaction_state, transaction_status
        end
```


Create Customer (as-is)
```mermaid
    sequenceDiagram
        autonumber 1
        rect rgb(252, 255, 220)
            Merchant->>+PCIPGW:Call create customer API (mode=fullpan)
            PCIPGW->>+KPGW:request charge (amt 0 for VS,MC)
            KPGW->>+CLK:request charge 0 THB
            CLK->>-KPGW:return payment result
            KPGW->>-PCIPGW:return payment result
            PCIPGW->>-Merchant:return customer_id and card_id
        end
        autonumber 1
        rect rgb(255, 220, 220)
            Merchant->>+PCIPGW:Call create Charge API (mode=customer,customer_id,card_id)
            PCIPGW->>+KPGW:request charge
            KPGW->>+CLK:request charge
            CLK->>-KPGW:return payment result
            KPGW->>-PCIPGW:return payment result
            PCIPGW->>-Merchant:return chargeid, transaction_state, transaction_status
        end
```


Create Customer (to-be)
```mermaid
    sequenceDiagram
        autonumber 1
        rect rgb(252, 255, 220)
            Merchant->>+PCIPGW:Call create Customer API (mode=fullpan)
            PCIPGW->>+KPGW:request charge (amt 0 for VS,MC)
            KPGW->>+CLK:request charge 0 THB
            CLK->>-KPGW:return payment result
            KPGW->>-PCIPGW:return payment result
            Note left of PCIPGW: Check MID have comp config to active cybs (for e-com)
            rect rgb(220, 251, 255)
              PCIPGW->>+CYBS:Request Create CybsTokenID
              CYBS->>-PCIPGW:return CybsTokenID
              Note right of PCIPGW: save CybsTokenID into new table and FK from table charge trx_id(pm)
            end
            PCIPGW->>-Merchant:return customer_id and card_id<br>(which is mapped with CybsTokenID)
        end
        autonumber 1
        rect rgb(255, 220, 220)
            Merchant->>+PCIPGW:Call create Charge API (mode=customer,customer_id,card_id)
              alt
                rect rgb(220, 251, 255)
                  Note right of PCIPGW: Check MID have comp config to active cybs (for e-com)
                  Note right of PCIPGW: AND matched with BP Config for card_brand(Visa,MC,JCB), card_type(credit,debit), onus/offus
                  Note right of PCIPGW: AND also found CybsTokenID in the database (token_cybs.payment_instrument_id)
                    PCIPGW->>+CYBS:Payment with CybsTokenID
                    CYBS->>-PCIPGW:Return payment result
                end
              else
                rect rgb(220, 255, 229)
                  Note right of PCIPGW: does not meet the conditions
                    PCIPGW->>+KPGW:request charge
                    KPGW->>+CLK:request charge
                    CLK->>-KPGW:return payment result
                    KPGW->>-PCIPGW:return payment result
                end
              end
            PCIPGW->>-Merchant:return chargeid, transaction_state, transaction_status
        end
```

API - Create Token (Token Process)
```mermaid
    sequenceDiagram
        autonumber 1
        rect rgb(252, 255, 220)
            Note over Merchant,PCIPGW: Token API mode=fullpan, API will be called by Kbank UI (PTT Blue Connect)
            Merchant->>+PCIPGW:KbankUI call create Token API (mode=fullpan,card_number,card_exp,cvv)
            PCIPGW->>-Merchant:return token_id
        end
        
```


API - Create Customer (Register Card)
```mermaid
    sequenceDiagram
        autonumber 1
        rect rgb(252, 255, 220)
            Note over Merchant,PCIPGW: Customer API mode=fullpan, usually request from lazada
            Merchant->>+PCIPGW:Call create Customer API (mode=fullpan)
            PCIPGW->>+KPGW:request charge (amt 0 for VS,MC)
            KPGW->>+CLK:request charge 0 THB
            CLK->>-KPGW:return payment result
            KPGW->>-PCIPGW:return payment result
            Note left of PCIPGW: Check MID have comp config to active cybs (for e-com)
            rect rgb(220, 251, 255)
              PCIPGW->>+CYBS:Request Create CybsTokenID
              CYBS->>-PCIPGW:return CybsTokenID
              Note right of PCIPGW: save CybsTokenID into new table and FK from table charge trx_id(pm)
            end
            PCIPGW->>-Merchant:return customer_id and card_id<br>(which is mapped with CybsTokenID)
        end
        
        autonumber 1
        rect rgb(255, 220, 220)
        Note over Merchant,PCIPGW: Customer API mode=token, check the tokenid is already got charge approval
            Merchant->>+PCIPGW:Call create Customer API (mode=token,token_id,email)
            Note left of PCIPGW: Check MID have comp config to active cybs (for e-com)
              rect rgb(220, 251, 255)
                  PCIPGW->>+CYBS:Request Create CybsTokenID
                  CYBS->>-PCIPGW:return CybsTokenID
                  Note right of PCIPGW: save CybsTokenID into new table and FK from table charge trx_id(pm)
              end
            PCIPGW->>-Merchant:return customer_id and card_id <br>(which is mapped with CybsTokenID)
        end
```

API - Create Charge (Charge Process)
```mermaid
    sequenceDiagram
        autonumber 1
        rect rgb(252, 255, 220)
            Note over Merchant,PCIPGW: Charge API mode=customer, already called customer API to get approval and customerid,cardid
            Merchant->>+PCIPGW:Call create Charge API (mode=customer,customer_id,card_id)
            alt
              rect rgb(220, 251, 255)
                Note right of PCIPGW: Check MID have comp config to active cybs (for e-com)
                Note right of PCIPGW: AND matched with BP Config for card_brand(Visa,MC,JCB), card_type(credit,debit), onus/offus
                Note right of PCIPGW: AND also found CybsTokenID in the database (token_cybs.payment_instrument_id)
                  PCIPGW->>+CYBS:Payment with CybsTokenID
                  CYBS->>-PCIPGW:Return payment result
              end
            else
              rect rgb(220, 255, 229)
                Note right of PCIPGW: does not meet the conditions
                  PCIPGW->>+KPGW:request charge
                  KPGW->>+CLK:request charge
                  CLK->>-KPGW:return payment result
                  KPGW->>-PCIPGW:return payment result
              end
            end
            PCIPGW->>-Merchant:return chargeid, transaction_state, transaction_status
        end
        
        autonumber 1
        rect rgb(255, 220, 220)
            Note over Merchant,PCIPGW: Charge API mode=register3d, always verify with 3DS
            Merchant->>+PCIPGW:Call create Charge API (mode=register3d,token_id)

            PCIPGW->>-Merchant:return chargeid & redirect_url

            Merchant->>+PCIPGW:Redirect to redirect_url
            PCIPGW->>+3DS:Redirect to redirect_url
            3DS->>3DS:send and verify OTP
            3DS->>-PCIPGW:return OTP result with ECI,CAVV

            PCIPGW->>+KPGW:request charge,ECI,CAVV (amt 0 for VS,MC)
            KPGW->>+CLK:request charge
            CLK->>-KPGW:return payment result
            KPGW->>-PCIPGW:return payment result
            
            PCIPGW->>-Merchant:return chargeid & payment result
        end
        
        autonumber 1
        rect rgb(252, 255, 220)
            Note over Merchant,PCIPGW: Charge API mode=token, usually request from embedded
            Merchant->>PCIPGW:Call create Charge API (mode=token,tokenid)
            Note right of PCIPGW: ***Check if the saved card & profile have condition as below
            rect rgb(255, 220, 220)
                Note right of PCIPGW: Check MID have comp config to active cybs (for e-com)
                Note right of PCIPGW: AND matched with BP Config for card_brand(Visa,MC,JCB), card_type(credit,debit), onus/offus
                Note right of PCIPGW: AND also found CybsTokenID in the database (token_cybs.payment_instrument_id)
                
                PCIPGW->>Merchant:return chargeid & redirect_url (for cybs 3ds)
                
                Merchant->>+PCIPGW:Redirect to redirect_url
                PCIPGW->>+3DS:Redirect to redirect_url
                3DS->>3DS:send and verify OTP
                3DS->>-PCIPGW:return OTP result with ECI,CAVV
            end
            
            rect rgb(220, 255, 229)
                Note right of PCIPGW: does not meet the conditions
                
                PCIPGW->>-Merchant:return chargeid & redirect_url (for kbank 3ds)
                
                Merchant->>+PCIPGW:Redirect to redirect_url
                PCIPGW->>+3DS_CYBS:Redirect to redirect_url
                3DS_CYBS->>3DS_CYBS:send and verify OTP
                3DS_CYBS->>+CYBS:Process a payment
                CYBS->>-3DS_CYBS:return payment result
                3DS_CYBS->>-PCIPGW:return payment result
            end

            PCIPGW->>-Merchant:return chargeid & payment result
        end
```
