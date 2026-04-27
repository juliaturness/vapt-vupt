vapt-vupt/
├── backend/                # Java 21 + Spring Boot 3.x
│   ├── src/main/java/br/edu/ifsc/vaptvupt/
│   │   ├── modules/        # Módulos Funcionais (Modular Monolith)
│   │   │   ├── auth/       # Segurança e JWT
│   │   │   ├── carrier/    # Gestão de Transportadoras
│   │   │   ├── driver/     # Gestão de Motoristas e Veículos
│   │   │   ├── cargo/      # Gestão de Cargas
│   │   │   ├── matching/   # Motor de Scoring e Ofertas
│   │   │   └── notification/# WebSocket e Push (FCM)
│   │   ├── shared/         # DTOs Globais, Exceptions, Utils
│   │   └── config/         # Security, Redis e Swagger Config
│   └── src/main/resources/
│       └── db/migration/   # Scripts Flyway (V1, V2...)
├── frontend/               # React 18 + TypeScript + Vite
│   ├── src/
│   │   ├── components/ui/  # Atomic Design (Shadcn/UI Style)
│   │   ├── features/       # Feature-based folder structure
│   │   ├── hooks/          # useAuth, useWebSocket, useCargo
│   │   └── store/          # Zustand (Global State)
├── docs/                   # Documentação Técnica
│   ├── adr/                # Architectural Decision Records
│   ├── api/                # Swagger/OpenAPI Specs e WebSockets
│   ├── diagrams/           # UML e ERD (Mermaid)
│   └── requirements/       # Backlog e Escopo
└── docker/                 # docker-compose.yml
