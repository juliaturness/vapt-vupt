# Cronograma de Desenvolvimento — VAPT VUPT

> Plano de execução fullstack para o ecossistema digital de logística ágil VAPT VUPT.  
> Stack: **React** (frontend) · **Java + Spring Boot** (backend) · **PostgreSQL** (banco de dados)  
> Modelo: **SaaS B2B** · Duração: **12 semanas**

---

## Visão Geral

| # | Fase | Período |
|---|------|---------|
| 01 | Planejamento e Levantamento de Requisitos | Semanas 1–2 |
| 02 | Arquitetura e Modelagem do Banco de Dados | Semanas 3–4 |
| 03 | Setup de Infraestrutura e CI/CD | Semana 5 |
| 04 | Backend — Auth, Motoristas e Transportadoras | Semanas 6–7 |
| 05 | Backend — Motor de Matching e Notificações | Semana 8 |
| 06 | Frontend — Dashboard Web (Transportadoras) | Semanas 9–10 |
| 07 | Frontend — Aplicativo Mobile (Motoristas) | Semana 11 |
| 08 | Testes e Documentação | Semana 12 (dias 1–3) |
| 09 | Diagramas Técnicos e Entrega Final | Semana 12 (dias 4–5) |

---

## Fase 01 — Planejamento e Levantamento de Requisitos

### Semana 1 — Escopo e Requisitos Funcionais

**Requisitos Funcionais**

- Cadastro de motoristas (perfil, documentos, disponibilidade, geolocalização)
- Cadastro de transportadoras (CNPJ, dados da empresa)
- Publicação de cargas com requisitos específicos (tipo de veículo, carga sensível, exigências de segurança)
- Motor de matching automático motorista ↔ carga
- Visualização de disponibilidade de motoristas em tempo real
- Validação documental integrada (CNH, CRLV, IPVA, documentos de segurança)
- Notificações em tempo real para motoristas e transportadoras
- Histórico de viagens e avaliações

**Requisitos Não Funcionais**

- Disponibilidade ≥ 99%
- Latência < 2s no processo de matching
- Autenticação e autorização via JWT
- Conformidade com a LGPD
- Arquitetura multi-tenant
- Escalabilidade horizontal

**Definição de Escopo MVP**

- ✅ Web Dashboard para transportadoras
- ✅ App Mobile (PWA) para motoristas
- ❌ Fora do MVP: integração direta com APIs de gerenciadoras de risco
- ❌ Fora do MVP: módulo financeiro / pagamentos

**📦 Entregas**
- Documento de requisitos funcionais e não funcionais
- Backlog inicial priorizado (método MoSCoW)

> 💡 **Boa prática:** Use User Story Mapping para visualizar o fluxo completo — transportadora publica carga → sistema faz match → motorista aceita → documentos validados → viagem iniciada.

---

### Semana 2 — Tecnologias, Padrões e Planejamento Técnico

**Stack Definido**

| Camada | Tecnologia |
|--------|-----------|
| Frontend | React 18 + TypeScript + Vite + Tailwind CSS + React Query + Zustand |
| Backend | Java 21 + Spring Boot 3.x + Spring Security + Spring Data JPA |
| Banco de dados | PostgreSQL 16 |
| Cache / Sessões | Redis |
| Armazenamento de arquivos | MinIO (local) / AWS S3 (produção) |
| Infra | Docker + GitHub Actions + Railway/Render (MVP) |

**Padrões Adotados**

- API REST com especificação OpenAPI 3.0
- Conventional Commits (`feat`, `fix`, `docs`, `refactor`, `test`, `chore`)
- Git Flow simplificado: `main`, `develop`, `feature/*`, `hotfix/*`
- Versionamento de API: `/api/v1/`

**📦 Entregas**
- ADRs (Architecture Decision Records) em `/docs/adr/`
- Repositório criado com estrutura base, README e `.gitignore`

> 💡 **Boa prática:** Documente as ADRs no próprio repositório. Isso elimina discussões futuras sobre "por que escolhemos X".

---

## Fase 02 — Arquitetura e Modelagem do Banco de Dados

### Semana 3 — Arquitetura do Sistema

**Decisão Arquitetural: Monólito Modular**

Recomendado para MVP de porte médio. Módulos internos:

- `auth` — autenticação e autorização
- `carriers` — transportadoras
- `drivers` — motoristas e veículos
- `cargos` — gestão de cargas
- `matching` — motor de matching
- `notifications` — notificações em tempo real

**Padrões do Backend**

```
Controller → Service → Repository (3 camadas)
```

- DTOs nas bordas da API (Request / Response)
- Exceptions customizadas com `@ControllerAdvice` global
- Event-driven interno para notificações (Spring Events ou Redis)

**Estrutura do Frontend**

```
/src
  /features
    /auth
    /carriers
    /cargos
    /matching
    /drivers
  /components
    /ui          # Button, Input, Modal, Table, Badge
  /hooks         # useAuth, useCargos, useWebSocket
  /lib           # axios instance, queryClient
```

- Atomic Design para componentes UI
- React Query para server state
- Zustand para client state
- Axios com interceptors para refresh de JWT

**📦 Entregas**
- Diagrama de arquitetura geral (C4 Level 1 e 2)
- Diagrama de pacotes do backend
- Estrutura de pastas definida e documentada

> 💡 **Boa prática:** Não comece com microserviços. Um monólito bem modularizado permite extrair serviços depois, sem o overhead inicial.

---

### Semana 4 — Modelagem do Banco de Dados

**Entidades Principais**

```sql
users          (id, email, password_hash, role, created_at, deleted_at)
carriers       (id, user_id FK, cnpj, razao_social, telefone)
addresses      (id, carrier_id FK, logradouro, cidade, estado, cep)
drivers        (id, user_id FK, cpf, cnh_numero, cnh_categoria, cnh_validade,
                latitude, longitude, disponivel, aprovado_gr)
vehicles       (id, driver_id FK, placa, tipo_id FK, capacidade_ton, rastreador)
vehicle_types  (id, nome, descricao)
documents      (id, driver_id FK, tipo, url, validade, status_validacao)
cargos         (id, carrier_id FK, origem, destino, tipo_carga, peso,
                data_coleta_limite, requer_escolta, requer_rastreador, status, deleted_at)
cargo_matches  (id, cargo_id FK, driver_id FK, status, criado_em)
```

**Relacionamentos**

- `users` 1:1 `carriers`
- `users` 1:1 `drivers`
- `drivers` 1:N `vehicles`
- `drivers` 1:N `documents`
- `carriers` 1:N `cargos`
- `cargos` N:M `drivers` via `cargo_matches`

**Status como ENUM**

```sql
-- documents.status_validacao
ENUM('PENDENTE', 'APROVADO', 'REJEITADO', 'EXPIRADO')

-- cargos.status
ENUM('AGUARDANDO', 'MATCHING', 'MOTORISTA_ALOCADO', 'EM_TRANSITO', 'CONCLUIDO', 'CANCELADO')

-- cargo_matches.status
ENUM('OFERTADO', 'ACEITO', 'RECUSADO', 'EXPIRADO')
```

**📦 Entregas**
- Diagrama ER (entidade-relacionamento) completo
- Script DDL inicial com migrations via Flyway
- Seed de dados para desenvolvimento

> 💡 **Boa prática:** Use Flyway para versionar migrations. Nunca altere uma migration existente — sempre crie novas.

---

## Fase 03 — Setup de Infraestrutura e CI/CD

### Semana 5 — DevOps e Ambientes

**Docker**

- `Dockerfile` multi-stage para o backend (build com Maven, runtime com JRE slim)
- `docker-compose.yml` para desenvolvimento local: `backend` + `postgresql` + `redis` + `frontend` (modo dev)

**Estratégia de Branches**

| Branch | Propósito |
|--------|-----------|
| `main` | Produção — protegida, requer PR |
| `develop` | Integração contínua |
| `feature/TICKET-descricao` | Novas funcionalidades |
| `hotfix/descricao` | Correções urgentes em produção |

PRs obrigatórios com ao menos 1 reviewer para `develop` e `main`.

**Pipelines GitHub Actions**

Pipeline de CI (em todo PR):
```
checkout → build → testes unitários → testes de integração
→ cobertura (JaCoCo ≥ 70%) → análise estática (Checkstyle)
```

Pipeline de CD:
```
merge em main → build Docker → push para registry
→ deploy automático em staging → aprovação manual → deploy em produção
```

**Ambientes**

| Ambiente | Trigger | Plataforma |
|----------|---------|------------|
| `dev` | Local | Docker Compose |
| `staging` | Merge em `develop` | Railway / Render |
| `prod` | Aprovação manual | Railway / Render |

**📦 Entregas**
- `docker-compose.yml` funcional para desenvolvimento local
- Pipelines CI/CD configurados e documentados
- Ambientes staging e produção provisionados

> 💡 **Boa prática:** Configure o CI para bloquear merge se os testes falharem. É a proteção mais barata que existe.

---

## Fase 04 — Backend: Auth, Motoristas e Transportadoras

### Semana 6 — Autenticação e Módulo de Motoristas

**Endpoints de Autenticação**

```
POST   /api/v1/auth/register         → Cadastro (DRIVER | CARRIER)
POST   /api/v1/auth/login            → Login → { accessToken, refreshToken }
POST   /api/v1/auth/refresh          → Renovar access token
POST   /api/v1/auth/logout           → Invalidar refresh token
```

Configuração JWT:
- Access token: **15 minutos**
- Refresh token: **7 dias** (armazenado no Redis)
- `UserDetailsService` customizado com Spring Security

**Endpoints de Motoristas**

```
GET    /api/v1/drivers/me                        → Perfil do motorista autenticado
PUT    /api/v1/drivers/me                        → Atualizar perfil
GET    /api/v1/drivers                           → Listar motoristas (admin)
PUT    /api/v1/drivers/{id}/availability         → Toggle disponibilidade
PUT    /api/v1/drivers/{id}/location             → Atualizar geolocalização
POST   /api/v1/drivers/me/vehicles               → Cadastrar veículo
GET    /api/v1/drivers/me/vehicles               → Listar veículos
```

**Endpoints de Documentos**

```
POST   /api/v1/drivers/me/documents              → Upload de documento (multipart/form-data)
GET    /api/v1/drivers/me/documents              → Listar documentos
PATCH  /api/v1/drivers/{id}/documents/{docId}/validate  → Validar (admin/GR)
```

**📦 Entregas**
- Auth completo com JWT e refresh token
- CRUD de motoristas e veículos com testes
- Upload e validação de documentos

> 💡 **Boa prática:** Use `@PreAuthorize("hasRole(...)")` nos controllers e teste cada endpoint com roles diferentes. Segurança deve ser testada, não assumida.

---

### Semana 7 — Módulo de Transportadoras e Cargas

**Endpoints de Transportadoras**

```
GET    /api/v1/carriers/me           → Perfil da transportadora autenticada
PUT    /api/v1/carriers/me           → Atualizar perfil
GET    /api/v1/carriers/{id}         → Dados públicos da transportadora
```

**Endpoints de Cargas**

```
POST   /api/v1/cargos                → Criar nova carga
GET    /api/v1/cargos                → Listar cargas (filtros: status, tipo, data, origem)
GET    /api/v1/cargos/{id}           → Detalhes da carga
PUT    /api/v1/cargos/{id}           → Atualizar carga
DELETE /api/v1/cargos/{id}           → Soft delete
PATCH  /api/v1/cargos/{id}/status    → Atualizar status
```

**Validações e Padrões**

- Bean Validation (`@NotNull`, `@Size`, `@Future` para datas)
- Anotação customizada `@ValidCNPJ`
- `@ControllerAdvice` global retornando RFC 7807 (Problem Details)
- Paginação via Spring `Pageable` em todos os endpoints de listagem

**📦 Entregas**
- CRUD completo de transportadoras e cargas
- Validações e tratamento de erros padronizado
- Paginação e filtros nos endpoints de listagem

> 💡 **Boa prática:** Implemente soft delete desde o início (campo `deleted_at`). Dados de frete têm valor histórico e nunca devem ser deletados fisicamente.

---

## Fase 05 — Backend: Motor de Matching e Notificações

### Semana 8 — Matching, Ofertas e WebSocket

**Motor de Matching**

```
POST   /api/v1/cargos/{id}/match/trigger   → Retorna lista ranqueada de motoristas elegíveis
```

Critérios de elegibilidade:
- `disponivel = true`
- Tipo de veículo compatível com a carga
- Documentos aprovados e não vencidos
- Flag `aprovado_gr = true` (aprovação em gerenciadora de risco)
- Geolocalização dentro do raio configurável

Algoritmo de scoring:
```
score = (peso_proximidade × distância_inversa) + score_histórico
```

**Fluxo de Ofertas**

```
POST   /api/v1/cargos/{id}/offers        → Criar oferta para motoristas elegíveis
GET    /api/v1/offers                    → Listar ofertas do motorista autenticado
POST   /api/v1/offers/{id}/accept        → Motorista aceita oferta
POST   /api/v1/offers/{id}/decline       → Motorista recusa oferta
```

**Notificações em Tempo Real**

- WebSocket com protocolo STOMP (Spring WebSocket)
- Motorista: topic `/user/{id}/notifications`
- Transportadora: topic `/topic/cargo/{id}/status`
- Fallback: push notification via Firebase Cloud Messaging (FCM)

**📦 Entregas**
- Motor de matching funcional com critérios documentados
- Fluxo completo de oferta e aceite
- WebSocket para notificações em tempo real

> 💡 **Boa prática:** O matching é o coração do produto. Escreva testes unitários extensivos para o `MatchingService` com diferentes combinações de critérios.

---

## Fase 06 — Frontend: Dashboard Web (Transportadoras)

### Semana 9 — Autenticação, Layout e Módulo de Cargas

**Setup do Projeto**

```
/src
  /features
    /auth          # login, registro, guards
    /carriers      # perfil da transportadora
    /cargos        # listagem, criação, detalhes
    /matching      # painel de matching e mapa
    /drivers       # gestão de motoristas parceiros
  /components
    /ui            # Button, Input, Modal, Table, Badge, Kanban
  /hooks           # useAuth, useCargos, useWebSocket, useMatching
  /lib
    /api.ts        # axios instance com interceptors JWT
    /queryClient.ts
```

**Fluxo de Autenticação**

- Telas de login e cadastro com validação via Zod
- Protected Routes com React Router v6
- Zustand store para usuário autenticado
- Refresh automático de token via axios interceptor

**Módulo de Cargas**

- Listagem com filtros (status, data, tipo de carga)
- Formulário multi-step de criação com validação Zod
- Kanban de status: `AGUARDANDO → MATCHING → MOTORISTA_ALOCADO → EM_TRANSITO → CONCLUÍDO`

**📦 Entregas**
- Auth completo no frontend
- Dashboard com listagem e criação de cargas
- Kanban de status de cargas

> 💡 **Boa prática:** Use React Query (TanStack Query) para todas as chamadas de API. O cache automático e o refetch eliminam a maioria dos problemas de sincronização de estado.

---

### Semana 10 — Matching em Tempo Real e Gestão de Motoristas

**Tela de Matching**

- Lista ranqueada de motoristas elegíveis com: foto, nome, tipo de veículo, distância, score, status de documentos
- Botão de enviar oferta individual ou para todos
- Status da oferta atualizado em tempo real via WebSocket

**Mapa de Motoristas Disponíveis**

- Integração com Leaflet.js (open source)
- Pins georreferenciados por localização dos motoristas
- Filtros por tipo de veículo
- Painel lateral ao clicar no pin com detalhes do motorista

**Gestão de Motoristas Parceiros**

- Lista de motoristas cadastrados com status de documentos (badge colorido)
- Ação de pré-validar motoristas frequentes

**📦 Entregas**
- Tela de matching com atualização em tempo real
- Mapa interativo de motoristas disponíveis
- Gestão de motoristas parceiros

> 💡 **Boa prática:** Trate loading e erro em todos os componentes. Use Suspense + Error Boundary. Usuário sem feedback é usuário que liga para o suporte.

---

## Fase 07 — Frontend: Aplicativo Mobile (Motoristas)

### Semana 11 — PWA Mobile para Motoristas

**Decisão Técnica (MVP)**

PWA com React (mesma base do dashboard web), com interface responsiva mobile-first. Benefícios:
- Reduz escopo em ~3 semanas comparado ao React Native
- Notificações push via Service Worker + FCM
- Instalável na tela inicial do celular
- Migração para React Native planejada para pós-MVP

**Funcionalidades do Motorista**

- Perfil com upload de documentos (câmera + galeria)
- Toggle de disponibilidade (online / offline) com geolocalização ativa
- Feed de ofertas recebidas com: origem, destino, tipo de carga, exigências de segurança
- Aceitar / recusar oferta com confirmação em 1 toque
- Histórico de viagens e avaliações recebidas

**Gestão de Documentos**

- Upload guiado de cada documento (CNH, CRLV, foto do veículo)
- Status de validade com alerta de vencimento próximo
- Notificação push quando documento é aprovado ou rejeitado

**📦 Entregas**
- PWA responsivo funcional para motoristas
- Fluxo completo de aceite de carga
- Upload e acompanhamento de documentos

> 💡 **Boa prática:** O motorista usa o app dentro de caminhões, muitas vezes com sinal fraco. Priorize performance, mensagens claras e ações com 1 toque.

---

## Fase 08 — Testes e Documentação

### Semana 12, dias 1–3

**Testes do Backend**

- Unitários com JUnit 5 + Mockito (Services, regras de matching)
- Integração com `@SpringBootTest` + Testcontainers (PostgreSQL real em container)
- Cobertura mínima: **70% nas camadas Service** (medido com JaCoCo)
- Testes de segurança:
  - Endpoints sem token → `401 Unauthorized`
  - Endpoints com role incorreta → `403 Forbidden`

**Testes do Frontend**

- Unitários com Vitest + Testing Library para componentes críticos (formulários, painel de matching)
- Testes E2E com Playwright:
  - Fluxo de login
  - Criação de carga
  - Trigger de matching e aceite de oferta

**Estratégia de Testes**

| Tipo | Ferramenta | Alvo | Cobertura mínima |
|------|-----------|------|------------------|
| Unitário backend | JUnit 5 + Mockito | Services | 70% |
| Integração backend | Testcontainers | Controllers + Repositories | Fluxos críticos |
| Unitário frontend | Vitest + Testing Library | Componentes | Formulários e estados |
| E2E | Playwright | Fluxos principais | 3 fluxos críticos |

**Documentação da API**

- Swagger / OpenAPI auto-gerado com SpringDoc
- Anotações `@Operation` e `@ApiResponse` em todos os controllers
- Acessível em: `/swagger-ui.html`

**README do Projeto**

O README deve conter:
- Visão geral do produto
- Pré-requisitos (Java 21, Node 20, Docker)
- Setup local com `docker-compose up`
- Variáveis de ambiente (com `.env.example`)
- Como rodar os testes
- Padrão de commits (Conventional Commits)

**Padrão de Commits**

```
feat(matching): adiciona algoritmo de scoring por proximidade
fix(auth): corrige expiração do refresh token
docs(readme): adiciona guia de setup local
refactor(drivers): extrai lógica de validação para DriverValidator
test(cargos): adiciona testes de integração para CargoController
chore(deps): atualiza Spring Boot para 3.3.0
```

**📦 Entregas**
- Suíte de testes com cobertura ≥ 70% no backend
- Swagger UI acessível e documentado
- README completo com guia de setup

> 💡 **Boa prática:** Conventional Commits + Semantic Release automatiza `CHANGELOG` e versionamento semântico. Configure desde o início.

---

## Fase 09 — Diagramas Técnicos e Entrega Final

### Semana 12, dias 4–5

**Diagrama de Arquitetura (C4 Level 1 e 2)**

```
Atores externos:
  - Transportadora (Web Dashboard)
  - Motorista (Mobile PWA)
  - Administrador (Web Dashboard)

Sistemas internos:
  - API Backend (Spring Boot)
  - PostgreSQL (dados relacionais)
  - Redis (cache + sessões)
  - S3 / MinIO (documentos)
  - Firebase FCM (push notifications)

Comunicações:
  - Frontend ↔ Backend: HTTPS / REST
  - Backend ↔ Frontend: WebSocket / STOMP
  - Backend ↔ Redis: TCP
  - Backend ↔ S3: HTTPS
```

**Diagrama de Classes (módulos principais)**

Módulos a documentar:
- `Cargo`, `CargoService`, `CargoRepository`, `CargoRequestDTO`, `CargoResponseDTO`
- `Driver`, `DriverService`, `DriverRepository`
- `MatchingService`, `MatchingCriteria`, `DriverScore`
- `CargoMatch`, `Offer`, `OfferService`

**Diagrama de Sequência — Fluxo Principal de Matching**

```
Transportadora → POST /cargos/{id}/match/trigger
  → MatchingService.findEligibleDrivers(cargo)
    → DriverRepository.findAvailableByVehicleType(...)
    → DocumentRepository.validateAll(driverIds)
    → MatchingService.scoreAndRank(drivers, cargo)
  → OfferService.createOffers(drivers, cargo)
    → WebSocketService.notifyDrivers(offers)
      → Motorista recebe notificação em tempo real
      → POST /offers/{id}/accept
        → CargoService.updateStatus(MOTORISTA_ALOCADO)
          → WebSocketService.notifyCarrier(cargo)
            → Transportadora recebe confirmação
```

**Refinamentos Finais**

Segurança:
- Rate limiting com Bucket4j (ex: 100 req/min por IP)
- Headers de segurança via Spring Security (`X-Frame-Options`, `X-Content-Type-Options`, `HSTS`)
- Sanitização de inputs nos DTOs

Performance:
- Índices no banco: `cargo.status`, `driver.disponivel + location`, `documents.validade`
- Cache Redis para listagem de motoristas disponíveis (TTL: 30s)

**📦 Entregas**
- Diagrama de arquitetura C4 (Level 1 e 2)
- Diagrama de classes dos módulos principais
- Diagrama de sequência do fluxo de matching
- Deploy em produção com monitoramento básico (Spring Actuator + logs estruturados)

> 💡 **Boa prática:** Use Mermaid no próprio README para os diagramas. Diagramas como código ficam versionados junto com o projeto e nunca ficam desatualizados.

---

## Checklist de Go-Live

Antes de colocar em produção, verificar:

- [ ] Variáveis de ambiente configuradas (nunca commitar `.env`)
- [ ] Migrations Flyway aplicadas no banco de produção
- [ ] Testes CI passando na branch `main`
- [ ] HTTPS configurado (TLS)
- [ ] Rate limiting ativo
- [ ] Logs estruturados (JSON) habilitados
- [ ] Spring Actuator exposto apenas internamente (`/actuator/health` para healthcheck)
- [ ] Backups automáticos do PostgreSQL configurados
- [ ] README atualizado com URL de produção
- [ ] Swagger UI desabilitado em produção (ou protegido por autenticação)

---

## Referências do Projeto

| Recurso | Link |
|---------|------|
| Spring Boot | https://spring.io/projects/spring-boot |
| Spring Security + JWT | https://spring.io/projects/spring-security |
| SpringDoc OpenAPI | https://springdoc.org |
| Flyway | https://flywaydb.org |
| Testcontainers | https://testcontainers.com |
| React Query | https://tanstack.com/query |
| Zustand | https://zustand-demo.pmnd.rs |
| Leaflet.js | https://leafletjs.com |
| Playwright | https://playwright.dev |
| Conventional Commits | https://www.conventionalcommits.org |
| Docker | https://www.docker.com |

---

*Documento gerado para o projeto VAPT VUPT — Instituto Federal de Santa Catarina, Câmpus São José.*
