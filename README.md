# Pride's Corner Farms: Ecommerce Workflow Charts

Diagrams supporting the Ecommerce Tools and Workflows summary.

**Key:** solid arrows are automated; dotted arrows are manual steps or handoffs.

1. [Order flow by channel](#order-flow-by-channel)
2. [Shared systems and support tools](#shared-systems-and-support-tools)
3. [Order flow: current state](#order-flow-current-state)
4. [Order flow: target state](#order-flow-target-state)

PNG versions are in the [images](images) folder.

## Order flow by channel

How each channel connects: inventory out, orders in, into NAV, shipping and tracking back.

```mermaid
flowchart TB
  %% Diagram 1: how each channel connects. Inventory out, orders in, tracking back.
  %% Solid arrow = automated. Dotted arrow = manual step or handoff.

  subgraph GPF["GPF site"]
    direction LR
    gA["NAV inventory<br/>Joe SQL list, emailed"] --> gB["Availability Sync"] --> gC["Matrixify"] --> gD["GPF Shopify"]
    gD -.->|order export| gE["GPF NAV importer<br/>per-order SO + bulk SO"] --> gF[("NAV")]
    gD --> gG["ShipStation"] --> gH["Tracking to<br/>GPF Shopify"]
    gD --- gI["CS: Re:amaze<br/>Dodie + PH team"]
  end

  subgraph WAL["Walmart (GPF brand)"]
    direction LR
    wA["GPF Shopify<br/>catalog"] -->|Marketplace Connect| wB["Walmart"] -->|Marketplace Connect| wC["GPF Shopify"]
    wC -.->|order export,<br/>separate customer| wD["GPF NAV importer"] --> wE[("NAV")]
    wB --> wF["ShipStation<br/>direct connection"] --> wG["Tracking to Walmart<br/>on label print"]
    wB --- wH["CS: Gemma in<br/>Seller Center"]
  end

  subgraph AMZ["Amazon Vendor Central (GPF brand)"]
    direction LR
    aA["Daily inventory<br/>file by email"] -.->|upload by hand| aB["Amazon<br/>Vendor Central"]
    aB -.->|PO download| aC["NAV importer<br/>per-order SO + bulk SO"] --> aD[("NAV")]
    aB --> aE["ShipStation<br/>direct connection"] --> aF["ASN to Amazon<br/>on label print"]
    aB --- aG["CS: Amazon<br/>Seller Central move started"]
  end

  subgraph SYNC["PCF Sync garden centers (~20 Shopify stores)"]
    direction LR
    pA["Availability Sync<br/>+ Matrixify"] --> pB["PCF Shopify"] --> pC["Syncio"] --> pD["Partner stores"]
    pD -->|orders| pE["Syncio"] --> pF["PCF Shopify"] --> pG["ShipStation"]
    pG --> pH["Order Download<br/>script"] -.->|upload| pI[("NAV")]
    pF -.->|Shopify API import<br/>Joe, in progress| pI
    pG --> pJ["Tracking to partner<br/>via Syncio"]
    pD --- pK["CS: partner<br/>claims: dfclaims@"]
  end

  subgraph FTP["FTP partners (PW, Almanac, Estabrook's, Weston, Ward's, Classy)"]
    direction LR
    fA["NAV inventory<br/>Joe SQL list"] -->|scheduled, Joe| fB["Partner FTP"] --> fC["FTP partners"]
    fC -->|order file| fD["Our FTP"] --> fE["Staging table"] -.->|Gemma processes| fF[("NAV")]
    fG["ShipStation"] --> fH["Automated FTP<br/>tracking push"]
    fG -.-> fI["ShipStation export<br/>to partner FTP"]
    fC --- fJ["CS: partner<br/>claims: dfclaims@"]
  end

  subgraph HYB["Heritage Hill, Spotswood (hybrid)"]
    direction LR
    hA["NAV inventory<br/>Joe SQL list"] -->|FTP| hB["Heritage Hill,<br/>Spotswood"]
    hB -->|ShipStation plugin| hC["ShipStation"] -.->|order report| hD["Upload"] -.-> hE[("NAV")]
    hC --> hF["Tracking to their<br/>Shopify, direct"]
  end

  subgraph EM["Email partners (Mahoneys, Frost Farms, Dietrich)"]
    direction LR
    eA["Availability list"] -.->|email| eB["Email partners"]
    eB -.->|order file| eC["Order sheet<br/>pivot table"] -.->|aggregated SO upload| eD[("NAV")]
    eE["ShipStation"] --> eF["Tracking email<br/>script"] --> eB2["Email partners"]
  end

  subgraph PA["Plant Addicts (owned, run as a partner)"]
    direction LR
    paA["NAV inventory<br/>Joe SQL list,<br/>Cross Reference"] -->|FTP| paB["Plant Addicts"]
    paB -->|orders to FTP| paC["PA Master 2026<br/>intake, changes,<br/>holiday SKUs"] --> paD["Multipack Splitter<br/>boxes + shipment #s"]
    paD -.->|upload by box type| paE[("NAV")]
    paD -.->|import file| paF["ShipStation"] --> paG["PA Tracking<br/>script"]
    paG --> paH["Master Shipments tab"]
    paG --> paI["Drive outbox<br/>daily file"] -.->|upload| paJ["PA FTP"]
    paB --- paK["CS: PA team<br/>in Re:amaze"]
  end

  subgraph HD["Home Depot"]
    direction LR
    hdA["Inventory file"] -.->|upload by hand| hdB["Home Depot"]
    hdB -->|EDI| hdC["TrueCommerce"] --> hdD["Staging table<br/>Joe"] --> hdE[("NAV")]
  end

  GPF ~~~ WAL ~~~ AMZ ~~~ SYNC ~~~ FTP ~~~ HYB ~~~ EM ~~~ PA ~~~ HD

  classDef nav fill:#e8eefc,stroke:#3b5bdb,color:#111
  classDef tool fill:#e6f7ec,stroke:#2f9e44,color:#111
  classDef plat fill:#fff4e0,stroke:#e8590c,color:#111
  classDef cs fill:#f8f9fa,stroke:#adb5bd,color:#555
  class gA,gF,wE,aD,pI,fA,fF,fE,hA,hE,eD,paA,paE,hdE,hdD nav
  class gB,gE,wD,aC,pA,pH,fH,eC,eF,paC,paD,paG,paH tool
  class gC,gD,gG,wA,wB,wC,wF,aB,aE,pB,pC,pE,pF,pG,fG,hC,eE,paF,hdC plat
  class gI,wH,aG,pK,fJ,paK cs
```

## Shared systems and support tools

Product data, inventory feeds, pick/pack/ship, invoicing, claims and compliance.

```mermaid
flowchart TB
  %% Diagram 2: shared systems, data connections and support tools.
  %% Solid arrow = automated. Dotted arrow = manual step or handoff.

  subgraph CFGL["Product data and the config sheet"]
    direction LR
    cA["NAV item export<br/>monthly"] -.->|paste| cB["AS - Reference & Config<br/>products, partners,<br/>cross refs, CAT1 states"]
    cC["NAV rack height<br/>email, daily"] --> cB
    cB --> cD["Read by: Availability Sync,<br/>PA Master, Multipack Splitter,<br/>Order Download, Rack Matcher,<br/>Shipping Invoice Report"]
  end

  subgraph BOX["Box sizes and packing slips"]
    direction LR
    bA["NAV rack height<br/>email, daily"] --> bB["ShipStation<br/>Product Sync"] -.->|review before push| bC["ShipStation<br/>box sizes +<br/>packing slip descriptions"]
  end

  subgraph INVL["Inventory feeds"]
    direction LR
    iA[("NAV inventory")] --> iB["Joe's SQL layer"] --> iX{"Which filter?"}
    iX -->|Ecomm checkbox<br/>on the item| iC["PCF ecommerce list<br/>emailed to our inbox"] --> iD["Availability Sync<br/>+ overrides tab"] --> iE["Matrixify"] --> iF["GPF Shopify"]
    iE --> iG["PCF Shopify"] --> iH["Syncio"] --> iI["PCF Sync<br/>partner stores"]
    iX -->|Ecomm checkbox<br/>on the item| iL["Other partner lists"] --> iM["FTP, email,<br/>or uploaded by hand"]
    iX -->|Cross Reference table| iJ["Plant Addicts,<br/>Proven Winners lists"] --> iK["Partner FTP"]
  end

  subgraph WH["Pick, pack, ship"]
    direction LR
    wA["Store plugins"] --> wC["ShipStation<br/>order records"]
    wB["Manual imports"] -.-> wC
    wD[("NAV wave run")] --> wE["Rack sheets printed<br/>inventory per rack"]
    wD --> wF["SQL package<br/>line report"]
    wC -->|pulls orders| wG["Rack Matcher<br/>web app"]
    wF -.->|load per wave| wG
    wG -->|matched orders<br/>as batches| wH["ShipStation<br/>batches"]
    wH -.-> wK["Label batch printed<br/>FedEx, UPS"]
    wK -.-> wI["Labels bagged with<br/>their rack sheet"]
    wE -.-> wI
    wI -.-> wJ["Brought to shipping pad,<br/>shopping racks built"]
  end

  subgraph BILL["Shipping invoices"]
    direction LR
    sA["ShipStation"] --> sB["Shipping Invoice Report<br/>freight by partner,<br/>date range"]
    sB -.->|match ShipStation orders<br/>to the bulk sales order| sC["Bulk sales order<br/>+ freight total<br/>+ fulfillment lines"]
    sC -.->|post| sD["AR invoices it"] --> sE[("Posted invoices<br/>in NAV")]
  end

  subgraph CLM["Damage claims and credits"]
    direction LR
    dA["Re:amaze<br/>dfclaims@"] --> dD["Vendor Shipping<br/>Exception tracker"]
    dB["ShipStation"] --> dD
    dC["SQL invoice lines<br/>from posted invoices"] --> dD
    dD -.->|print, hand off| dE["Accounts Receivable"]
    dF["Carrier portals<br/>Maggie, Jade"]
  end

  subgraph COMP["Compliance"]
    direction LR
    jA["ShipStation"] --> jB["Japanese Beetle<br/>pre-notification"] -.->|email + phyto certs| jC["Western state<br/>ag departments"]
    jA --> jD["Buxus NY<br/>compliance"] -.-> jE["Survey123"]
  end

  subgraph LGLL["Plant Addicts vendor feed"]
    direction LR
    lA["LGL API"] --> lB["LGL Product Sync"] --> lC["PA BigCommerce"]
  end

  CFGL ~~~ BOX ~~~ INVL ~~~ WH ~~~ BILL ~~~ CLM ~~~ COMP ~~~ LGLL

  classDef nav fill:#e8eefc,stroke:#3b5bdb,color:#111
  classDef tool fill:#e6f7ec,stroke:#2f9e44,color:#111
  classDef plat fill:#fff4e0,stroke:#e8590c,color:#111
  classDef cfg fill:#fff9db,stroke:#f08c00,color:#111,stroke-width:2px
  class cA,cC,bA,iA,iB,prA,wD,wE,wF,dC,sE nav
  class bB,iD,wG,sB,dD,jB,jD,lB tool
  class bC,iE,iF,iG,iH,prB,prE,prF,prG,wC,wH,sA,dA,dB,dF,jA,lC plat
  class cB cfg
```

## Order flow: current state

```mermaid
flowchart TB
  %% Order flow: CURRENT STATE
  %% Solid arrow = automated. Dotted arrow = manual step or handoff.

  subgraph A["1  How orders get into ShipStation and NAV"]
    direction TB

    subgraph A1["Green Promise Farms"]
      direction LR
      a1a["GPF Shopify"] -->|direct connection| a1b(["ShipStation"])
      a1a -.->|order export| a1c["GPF NAV importer<br/><i>per-order SOs for accounting<br/>+ one bulk SO</i>"] --> a1d[("NAV")]
    end

    subgraph A2["Walmart"]
      direction LR
      a2a["Walmart"] -->|direct connection| a2b(["ShipStation"])
      a2a -->|Marketplace Connect| a2c["GPF Shopify"] -.->|order export| a2d["GPF NAV importer<br/><i>separate NAV customer</i>"] --> a2e[("NAV")]
    end

    subgraph A3["Amazon Vendor Central"]
      direction LR
      a3a["Amazon"] -->|direct connection| a3b(["ShipStation"])
      a3a -.->|order download| a3c["GPF NAV importer"] --> a3d[("NAV")]
    end

    subgraph A4["PCF Sync partners"]
      direction LR
      a4a["Partner Shopify stores"] -->|Syncio| a4b["PCF Shopify"] -->|direct connection| a4c(["ShipStation<br/><i>split by partner tag</i>"]) --> a4d["Order Download report<br/><i>SKUs aggregated</i>"] -.->|upload| a4e[("NAV")]
    end

    subgraph A5["Heritage Hill, Spotswood, Hedge & Hollow"]
      direction LR
      a5a["Partner Shopify stores"] -->|ShipStation plugin| a5b(["ShipStation"]) --> a5c["Order Download report<br/><i>SKUs aggregated</i>"] -.->|upload| a5d[("NAV")]
    end

    subgraph A6["Home Depot"]
      direction LR
      a6a["Home Depot"] -->|EDI| a6b["TrueCommerce"] --> a6c["Staging table"] --> a6d(["ShipStation"])
      a6c --> a6e[("NAV")]
    end

    subgraph A7["FTP partners: Proven Winners, Almanac, Estabrook's, others"]
      direction LR
      a7a["Partner FTP"] --> a7b["Staging table"] -.->|Gemma processes| a7c[("NAV")]
      a7a -.->|manual import| a7d(["ShipStation"])
    end

    subgraph A8["Email partners: Mahoneys, Frost Farms, Dietrich"]
      direction LR
      a8a["Order file by email"] -.-> a8b["Order sheet pivot<br/><i>SKUs aggregated</i>"] -.->|upload| a8c[("NAV")]
      a8a -.->|manual import| a8d(["ShipStation"])
    end

    subgraph A9["Plant Addicts (Proven Winners occasionally)"]
      direction LR
      a9a["Orders to FTP"] --> a9b["PA Master<br/><i>intake, changes</i>"] --> a9c["Box Planner<br/><i>one order per box</i>"]
      a9c -.->|upload by box type| a9d[("NAV")]
      a9c -.->|import file| a9e(["ShipStation"])
    end
  end

  subgraph B["2  Warehouse and shipping (all channels)"]
    direction LR
    b1[("NAV sales orders")] --> b2["Warehouse shipment"] --> b3["Wave<br/><i>CAT1 runs as its own wave</i>"]
    b3 --> b4["Rack sheets printed"]
    b3 --> b5["SQL package line report<br/><i>package ID = rack</i>"]
    b6(["ShipStation<br/>awaiting shipment"]) -->|pulls orders| b7["Rack Matcher"]
    b5 -.->|load per wave| b7
    b7 --> b8(["ShipStation batches"]) --> b9["Label batch printed"] --> b10["Labels bagged<br/>with rack sheet"]
    b4 -.-> b10
    b10 -.-> b11["Shopping racks built<br/>at shipping pad"] --> b12["Carrier pickup<br/><i>FedEx, UPS</i>"]
    b12 -.-> b13["Invoicing<br/><i>freight + fulfillment lines added<br/>to the bulk SO, posted for AR</i>"]
  end

  subgraph C["3  Tracking back"]
    direction TB
    subgraph C1["GPF, Walmart, Amazon, Heritage Hill group"]
      direction LR
      c1a(["ShipStation"]) -->|store connection,<br/>ASN for Amazon| c1b["Channel updated<br/>on label print"]
    end
    subgraph C2["PCF Sync partners"]
      direction LR
      c2a(["ShipStation"]) --> c2b["PCF Shopify"] -->|Syncio| c2c["Partner stores"]
    end
    subgraph C3["Home Depot"]
      direction LR
      c3a(["ShipStation"]) --> c3b["TrueCommerce"] --> c3c["Home Depot"]
    end
    subgraph C4["Plant Addicts"]
      direction LR
      c4a(["ShipStation"]) --> c4b["PA Tracking script<br/><i>Master + daily file</i>"] --> c4c["Drive outbox"] -.->|upload| c4d["PA FTP"]
    end
    subgraph C5["Email partners"]
      direction LR
      c5a(["ShipStation"]) --> c5b["Tracking email script"] --> c5c["Partner inbox"]
    end
    subgraph C6["FTP partners"]
      direction LR
      c6a(["ShipStation"]) -.->|export| c6b["Tracking file"] -.->|upload| c6c["Partner FTP"]
    end
  end

  A1 ~~~ A2 ~~~ A3 ~~~ A4 ~~~ A5 ~~~ A6 ~~~ A7 ~~~ A8 ~~~ A9
  C1 ~~~ C2 ~~~ C3 ~~~ C4 ~~~ C5 ~~~ C6
  A ~~~ B ~~~ C

  classDef ext fill:#eef2f0,stroke:#8a938d,color:#1a1a1a
  classDef ship fill:#e6efe9,stroke:#186c3f,color:#12372a
  classDef erp fill:#eceef5,stroke:#5b6b9e,color:#1a2340
  classDef tool fill:#f6efe4,stroke:#a8752f,color:#3a2c14
  class a1a,a2a,a2c,a3a,a4a,a4b,a5a,a6a,a6b,a7a,a8a,a9a,c1b,c2b,c2c,c3b,c3c,c4d,c5c,c6c ext
  class a1b,a2b,a3b,a4c,a5b,a6d,a7d,a8d,a9e,b6,b8,c1a,c2a,c3a,c4a,c5a,c6a ship
  class a1d,a2e,a3d,a4e,a5d,a6c,a6e,a7b,a7c,a8c,a9d,b1,b2,b3,b4,b5,b13 erp
  class a1c,a2d,a3c,a4d,a5c,a8b,a9b,a9c,b7,b9,b10,b11,b12,c4b,c4c,c5b,c6b tool
```

## Order flow: target state

Bold blue boxes are new or changed from today.

```mermaid
flowchart TB
  %% Order flow: TARGET STATE
  %% Solid arrow = automated. Dotted arrow = manual step or handoff.
  %% Bold blue boxes = new or changed from today.

  subgraph A["1  How orders get in"]
    direction TB

    subgraph A1["GPF brand channels: GPF site, Walmart, Amazon (no change)"]
      direction LR
      a1a["GPF Shopify, Walmart,<br/>Amazon"] -->|direct connection| a1b(["ShipStation"])
      a1a -.->|order export| a1c["GPF NAV importer"] --> a1d[("NAV")]
    end

    subgraph A2["PCF Sync partners"]
      direction LR
      a2a["Partner Shopify stores"] -->|Syncio| a2b["PCF Shopify"] -->|Shopify API import| a2c["Staging table"]
    end

    subgraph A3["Home Depot (no change)"]
      direction LR
      a3a["Home Depot"] -->|EDI| a3b["TrueCommerce"] --> a3c["Staging table"]
    end

    subgraph A4["FTP partners: Proven Winners, Almanac, Heritage Hill group, others"]
      direction LR
      a4a["Partner FTP"] -->|picked up on a schedule| a4b["Staging table"]
    end

    subgraph A5["Plant Addicts"]
      direction LR
      a5a["Orders to FTP"] -->|picked up on a schedule| a5b["Staging table"]
    end

    subgraph A6["Email partners (no change)"]
      direction LR
      a6a["Order file by email"] -.-> a6b["Order sheet pivot"] -.->|upload| a6c[("NAV")]
      a6a -.->|manual import| a6d(["ShipStation"])
    end
  end

  subgraph S["2  Staging table (all partners except GPF brand and email)"]
    direction LR
    s1["Staging table"] --> s2["Boxing logic<br/>applied on import<br/><i>one order per box</i>"]
    s2 --> s3[("NAV sales orders")]
    s2 -->|order lines pushed| s4(["ShipStation"])
  end

  subgraph B["3  Warehouse and shipping (all channels, no change)"]
    direction LR
    b1[("NAV sales orders")] --> b2["Warehouse shipment"] --> b3["Wave<br/><i>CAT1 runs as its own wave</i>"]
    b3 --> b4["Rack sheets printed"]
    b3 --> b5["SQL package line report<br/><i>package ID = rack</i>"]
    b6(["ShipStation<br/>awaiting shipment"]) -->|pulls orders| b7["Rack Matcher"]
    b5 -.->|load per wave| b7
    b7 --> b8(["ShipStation batches"]) --> b9["Label batch printed"] --> b10["Labels bagged<br/>with rack sheet"]
    b4 -.-> b10
    b10 -.-> b11["Shopping racks built<br/>at shipping pad"] --> b12["Carrier pickup<br/><i>FedEx, UPS</i>"]
    b12 -.-> b13["Invoicing<br/><i>freight + fulfillment lines added<br/>to the bulk SO, posted for AR</i>"]
  end

  subgraph C["4  Tracking back"]
    direction TB
    subgraph C1["GPF brand channels (no change)"]
      direction LR
      c1a(["ShipStation"]) -->|store connection,<br/>ASN for Amazon| c1b["Channel updated<br/>on label print"]
    end
    subgraph C2["Staging table partners"]
      direction LR
      c2a(["ShipStation"]) --> c2b["Tracking returned<br/>automatically,<br/>same path as the order"]
      c2b --> c2c["TrueCommerce<br/>to Home Depot"]
      c2b --> c2d["Tracking file to<br/>partner FTP<br/><i>FTP partners, Plant Addicts</i>"]
      c2b --> c2e["PCF Shopify, then Syncio<br/>to partner stores"]
    end
    subgraph C3["Email partners (no change)"]
      direction LR
      c3a(["ShipStation"]) --> c3b["Tracking email script"] --> c3c["Partner inbox"]
    end
  end

  A1 ~~~ A2 ~~~ A3 ~~~ A4 ~~~ A5 ~~~ A6
  C1 ~~~ C2 ~~~ C3
  A ~~~ S ~~~ B ~~~ C

  classDef ext fill:#eef2f0,stroke:#8a938d,color:#1a1a1a
  classDef ship fill:#e6efe9,stroke:#186c3f,color:#12372a
  classDef erp fill:#eceef5,stroke:#5b6b9e,color:#1a2340
  classDef tool fill:#f6efe4,stroke:#a8752f,color:#3a2c14
  classDef newwork fill:#e8f0fb,stroke:#1a5fb4,color:#0d2f5c,stroke-width:2px
  class a1a,a2a,a2b,a3a,a3b,a4a,a5a,a6a,c1b,c2c,c3c ext
  class a1b,a6d,s4,b6,b8,c1a,c2a,c3a ship
  class a1d,a3c,a6c,s1,s3,b1,b2,b3,b4,b5,b13 erp
  class a1c,a6b,b7,b9,b10,b11,b12,c3b tool
  class a2c,a4b,a5b,s2,c2b,c2d,c2e newwork
```
