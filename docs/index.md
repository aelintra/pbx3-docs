# PBX3 documentation

Operator and installer guides for **PBX3** — a multi-tenant phone system you can run as a single node or as a fleet.

!!! tip "Site status"
    Navigation and structure are live for review. Most pages are still placeholders; the system schematic below is the first real content.

## Big picture

```mermaid
flowchart TB
    subgraph phones["Phones and softphones"]
      P1["Handset"]
      P2["Softphone"]
    end

    SBC{{"SBC<br/>SIP edge · single front door"}}

    subgraph fleet["PBX instances"]
      direction LR
      N1["Instance A<br/>Asterisk + API + tenants"]
      N2["Instance B<br/>Asterisk + API + tenants"]
      N3["Instance C<br/>Asterisk + API + tenants"]
    end

    S3[("Org S3 store<br/>catalog · backups · recordings")]
    GK["Gatekeeper<br/>fleet control plane"]
    SPA["Admin SPA<br/>instance + Fleet mode"]
    CAR[["PSTN / Carriers"]]

    P1 --> SBC
    P2 --> SBC
    SBC -->|"routes by tenant domain"| N1
    SBC --> N2
    SBC --> N3
    SBC <-->|"trunks"| CAR

    N1 -. "backups / meta" .-> S3
    N2 -. "backups / meta" .-> S3
    N3 -. "backups / meta" .-> S3

    GK -->|"catalog truth"| S3
    GK -->|"repoint / project"| SBC
    GK -.->|"fleet APIs"| fleet
    SPA -->|"instance Sanctum"| fleet
    SPA -->|"fleet token"| GK

    classDef edge fill:#f4d,stroke:#333,color:#000;
    class SBC edge;
```

**Calls** go Phones → **SBC** → **instances**. **S3** and the **Gatekeeper** are control-plane memory and orchestration — calls keep working if they are down.

Start with [What is PBX3?](getting-started/what-is-pbx3.md).
