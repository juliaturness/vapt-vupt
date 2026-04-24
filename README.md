# VAPT VUPT 
> Conectando transportadoras a motoristas autônomos de forma imediata e segura.

## Sobre o Projeto

O VAPT VUPT é uma solução SaaS concebida como uma oportunidade de inovação para o setor logístico, desenvolvida no contexto acadêmico de Análise e Desenvolvimento de Sistemas do IFSC. O sistema atua como uma camada de inteligência que resolve a ineficiência na gestão de motoristas para cargas intermunicipais e estaduais.

O ecossistema divide-se em duas frentes principais:
* **Web Dashboard (Transportadoras):** Uma central de comando para gestores logísticos visualizarem, em tempo real, quais motoristas estão disponíveis para tipos específicos de coleta.
* **Aplicativo Mobile (Motoristas):** Interface para os profissionais manterem seus perfis digitais, enviarem documentos de gerenciamento de risco e receberem notificações de cargas.

## Tecnologias Utilizadas

A arquitetura do projeto foi desenhada para suportar alta disponibilidade e segurança dos dados:

* **Frontend (Web):** React 
* **Backend:** Java com Spring Boot
* **Banco de Dados:** PostgreSQL
* **Mobile (App do Motorista):** React Native
  
## Funcionalidades Principais

* **Triagem Automática:** Ao cadastrar uma carga, o sistema sugere o "match" ideal, exibindo apenas profissionais com documentação validada.
* **Geolocalização Estratégica:** O app utiliza localização em segundo plano para informar a posição do motorista à plataforma.
* **Gestão de Conformidade:** Centralização da checagem de segurança e envio de documentos para gerenciadoras de risco, eliminando o trânsito inseguro de arquivos.
* **Notificações de Prioridade:** Alertas de cargas de alta prioridade enviados diretamente para os motoristas aptos e disponíveis na região.

