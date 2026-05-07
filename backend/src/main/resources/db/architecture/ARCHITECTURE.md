Aqui tens o documento Markdown (`.md`) completo e estruturado para a documentação técnica do projeto **VAPT VUPT**. Podes copiar este conteúdo e guardar num ficheiro chamado `ARCHITECTURE.md` dentro da pasta `/docs/architecture/` do teu repositório.

---

# Documentação de Arquitetura — VAPT VUPT

Esta documentação detalha a estrutura arquitetural do sistema **VAPT VUPT**, uma solução SaaS de logística para conexão entre transportadoras e motoristas autónomos. A arquitetura segue o modelo **C4 (Context, Container, Component, Code)** para facilitar a compreensão em diferentes níveis de abstração.

---

## 1. Visão Geral
O sistema funciona como uma camada de inteligência para o setor logístico, permitindo que transportadoras encontrem motoristas validados em tempo real, utilizando geolocalização e algoritmos de scoring.

---

## 2. Diagrama de Contexto (C4 Level 1)
Este nível descreve os atores que utilizam o sistema e as dependências com sistemas externos.



```mermaid
flowchart TD
    classDef actor fill:#08427b,stroke:#052e56,color:#fff,shape:circle;
    classDef system fill:#1168bd,stroke:#0b4884,color:#fff,shape:rect,rx:10,ry:10;
    classDef external fill:#999999,stroke:#666666,color:#fff,shape:rect,rx:10,ry:10;

    Carrier(("\nTransportadora")):::actor
    Driver(("\nMotorista\nAutónomo")):::actor
    Admin(("🛡\nAdministrador")):::actor

    System["VAPT VUPT\n[Sistema de Matching Logístico]\nConecta transportadoras a motoristas validados em tempo real"]:::system

    GR["Gerenciadora de Risco\n[Sistema Externo]\nAPIs de checagem de antecedentes"]:::external
    FCM["Firebase Cloud Messaging\n[Sistema Externo]\nServiço de Push Notifications"]:::external
    S3["Amazon S3\n[Sistema Externo]\nArmazenamento de CNH e CRLV"]:::external
    Maps["Google Maps\n[Sistema Externo]\nGeolocalização e rotas"]:::external

    Carrier -- "Regista cargas e gere ofertas" --> System
    Driver -- "Envia localização e aceita fretes" --> System
    Admin -- "Audita documentações" --> System

    System -- "Valida motoristas" --> GR
    System -- "Dispara alertas push" --> FCM
    System -- "Gere ficheiros seguros" --> S3
    System -- "Calcula distâncias" --> Maps
```

---

## 3. Diagrama de Contentores (C4 Level 2)
Detalhamento das tecnologias e como os principais blocos de software comunicam entre si.

```mermaid
flowchart TD
    classDef actor fill:#08427b,stroke:#052e56,color:#fff,shape:circle;
    classDef container fill:#438dd5,stroke:#1168bd,color:#fff,shape:rect,rx:10,ry:10;
    classDef db fill:#438dd5,stroke:#1168bd,color:#fff,shape:cylinder;
    classDef external fill:#999999,stroke:#666666,color:#fff,shape:rect,rx:10,ry:10;

    Carrier(("\nTransportadora")):::actor
    Driver(("\nMotorista")):::actor

    subgraph Ecossistema VAPT VUPT
        WebApp["Web Dashboard\n[React]\nInterface para gestão de cargas"]:::container
        MobileApp["App Mobile\n[React Native]\nApp do motorista"]:::container
        
        Backend["API Backend\n[Java / Spring Boot]\nMonólito modular com lógica de negócio"]:::container
        
        DB[("PostgreSQL\n[RDBMS]\nDados relacionais e persistência")]:::db
        Redis[("Redis\n[In-Memory]\nCache de geolocalização")]:::db
    end

    FCM["Firebase Cloud Messaging"]:::external
    S3["Amazon S3"]:::external

    Carrier -- "HTTPS" --> WebApp
    Driver -- "HTTPS" --> MobileApp

    WebApp -- "JSON/HTTPS" --> Backend
    MobileApp -- "JSON/HTTPS & WSS" --> Backend

    Backend -- "JDBC/TCP" --> DB
    Backend -- "RESP" --> Redis
    Backend -- "HTTPS" --> FCM
    Backend -- "HTTPS" --> S3
```

---

## 4. Diagrama de Pacotes (Backend)
O backend é estruturado como um **Monólito Modular** para garantir a manutenibilidade e possível transição para microsserviços.



```mermaid
flowchart TB
    classDef module fill:#e0e7ff,stroke:#4f46e5,color:#000;
    classDef shared fill:#fef08a,stroke:#ca8a04,color:#000;

    subgraph Monolito Modular Spring Boot
        API[API Controllers Layer]
        
        Auth[Módulo: Auth\nSegurança e JWT]:::module
        CarrierMod[Módulo: Carriers\nGestão de empresas]:::module
        DriverMod[Módulo: Drivers\nMotoristas e Veículos]:::module
        CargoMod[Módulo: Cargos\nGestão de fretes]:::module
        MatchMod[Módulo: Matching\nAlgoritmo de Scoring]:::module
        NotifMod[Módulo: Notifications\nPush e WebSockets]:::module
        
        Shared[Common / Shared\nDTOs e Exceptions]:::shared

        API --> Auth & CarrierMod & DriverMod & CargoMod & MatchMod & NotifMod
        
        MatchMod -.-> DriverMod
        MatchMod -.-> CargoMod
        MatchMod -.-> NotifMod
        
        Auth & CarrierMod & DriverMod & CargoMod & MatchMod & NotifMod --> Shared
    end
```

---

## 5. Protocolos e Comunicação

| Protocolo | Utilização | Finalidade |
| :--- | :--- | :--- |
| **REST (HTTPS)** | Geral | Operações síncronas de CRUD e autenticação. |
| **WebSocket (STOMP)** | Mobile -> Backend | Envio de coordenadas de geolocalização em tempo real. |
| **SSE** | Backend -> Web | Atualização automática do dashboard logístico. |
| **FCM Push** | Cloud -> Mobile | Notificação de cargas de alta prioridade. |

---

## 6. Modelo de Dados (ER)
Estrutura normalizada (3FN) para suporte à integridade dos dados.

```mermaid
erDiagram
    users ||--o| carrier_profiles : "1:1"
    users ||--o| driver_profiles : "1:1"
    driver_profiles ||--|{ vehicles : "1:N"
    driver_profiles ||--|{ documents : "1:N"
    carrier_profiles ||--|{ cargos : "1:N"
    cargos ||--|{ offers : "1:N"
    driver_profiles ||--|{ offers : "1:N"
    
    users {
        uuid id PK
        varchar email UK
        varchar password
        user_role role
    }
    
    driver_profiles {
        uuid id PK
        uuid user_id FK
        char11 cpf UK
        decimal latitude
        decimal longitude
        boolean disponivel
    }

    cargos {
        uuid id PK
        uuid carrier_id FK
        varchar origem_cidade
        varchar destino_cidade
        cargo_status status
    }
```

---

## 7. Decisões de Arquitetura (ADR)
* **Java 21 & Spring Boot 3**: Utilização de Virtual Threads para melhor performance em I/O.
* **Flyway**: Controlo de versão do esquema da base de dados para ambientes de staging/prod.
* **PostGIS (Futuro)**: Planeada a extensão do PostgreSQL para cálculos geográficos complexos.
* **Imutabilidade**: Uso de `Records` em Java para DTOs.
