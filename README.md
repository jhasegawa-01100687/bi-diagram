# bi-diagram

```mermaid
flowchart BT
    subgraph AWS["AWS"]
        direction LR
        Core["基幹システム等<br/>業務データ発生元"]
        WOS["WOS<br/>ECデータ発生元"]
        POSITIVE["POSITIVE<br/>人事系データ発生元"]
        Unikage["ユニケージ（旧）<br/>業務データ配信"]
        Mart["マート<br/>Access ODBC接続用"]
        DMT["DMT<br/>Cognos用"]

        WOS --> Unikage
        Core --> Unikage
        Unikage --> Mart
        Unikage --> DMT
    end

    subgraph GCP["Google Cloud"]
        direction TB

        subgraph pj-hc-data-hub["pj-hc-data-hub"]
            DH_Cold["Cold層"]
            DH_Warm["Warm層"]
            DH_Cold --> DH_Warm
        end

        subgraph pj-hc-bi-public["pj-hc-bi-public"]
            PUB_Hot["Hot層"]
            PUB_Out["API / Looker Studio<br/>一般ユーザー向け"]
            PUB_Hot --> PUB_Out
        end

        subgraph pj-hc-bi-secure["pj-hc-bi-secure"]
            SEC_Hot["Hot層"]
            SEC_Out["Looker Studio<br/>人事ユーザー向け"]
            SEC_Hot --> SEC_Out
        end

        subgraph pj-fcl-xxx["pj-fcl-xxx"]
            FCL_Warm["Warm層"]
            FCL_Hot["Hot層"]
            FCL_Out["Looker Studio"]
            FCL_Warm --> FCL_Hot --> FCL_Out
        end

        subgraph pj-core-system-bi["pj-core-system-bi"]
            direction TB
            subgraph Stream2["2系統目（後続で構築）"]
                S2_Cold["Cold層<br/>（Raw Ingest）"]
                S2_Warm["Warm層<br/>（業務定義確定）<br/>売上･原価･粗利･返品等 = SSOT"]
                S2_Hot["Hot層<br/>（表示最適化）<br/>日次/週次/期間軸<br/>YoY/MoM/WoW ワイドテーブル"]
                S2_Cold --> S2_Warm --> S2_Hot
            end
            subgraph Stream1["1系統目（優先構築）"]
                S1_Cold["Cold層"]
                S1_Warm["Warm層"]
                S1_Hot["Hot層"]
                S1_Cold --> S1_Warm --> S1_Hot
            end
            CORE_Out["アクション提案AI<br/>Looker Studio 一般向け"]
            S1_Hot --> CORE_Out
            S2_Hot -.-> CORE_Out
        end

        subgraph world-group["world-group-230204（既存・要棚卸）"]
            WG_Cold["Cold層"]
            WG_Warm["Warm層"]
            WG_Hot["Hot層"]
            WG_Out["Looker Studio"]
            WG_Cold --> WG_Warm --> WG_Hot --> WG_Out
        end
    end

    %% データフロー（AWS → GCP）
    Unikage --> WG_Cold
    S2_Cold --> FCL_Warm
    DH_Warm --> PUB_Hot
    DH_Warm --> SEC_Hot
    Mart -->|Datastream<br/>段階的に廃止| S1_Cold
    DMT -->|Datastream<br/>段階的に廃止| S1_Cold
    Core -.->|将来の正規ルート| S2_Cold
    POSITIVE -->|Datastream| DH_Cold
```
