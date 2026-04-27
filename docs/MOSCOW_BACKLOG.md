# Backlog Priorizado (MoSCoW) - MVP VAPT VUPT

## Must Have (Obrigatório para o MVP)
* Cadastro e autenticação (JWT) de Transportadoras e Motoristas.
* CRUD de veículos e perfil de disponibilidade do motorista.
* Upload de documentos do motorista (CNH, CRLV).
* Publicação de cargas com requisitos específicos pela transportadora.
* Algoritmo de matching (score por distância, tipo de veículo e validade de documentos).
* Fluxo de envio de oferta, aceite e recusa pelo motorista.
* Atualização de status da carga em tempo real (WebSocket).
* Aplicativo Mobile (PWA) para o motorista com geolocalização.

## Should Have (Importante, mas o sistema funciona sem)
* Painel de mapa interativo (Leaflet) para visualizar motoristas disponíveis.
* Notificações Push (FCM) como fallback caso o WebSocket falhe ou app esteja em background.
* Histórico e avaliações de viagens concluídas.

## Could Have (Desejável, se houver tempo)
* Funcionalidade de pré-validação de motoristas "favoritos" ou recorrentes pela transportadora.
* OCR para pré-preenchimento automático dos dados da CNH no upload.

## Won't Have (Fora do escopo do MVP atual)
* Integração via API direta com sistemas de Gerenciadoras de Risco (validação será visual/manual pelo admin no MVP).
* Módulo financeiro (pagamentos, emissão de CTe, faturamento).
* App Mobile Nativo em React Native (será PWA na primeira versão).