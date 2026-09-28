# Arquitetura de Referência de Software para Vibe Coding

**Versão:** 1.1\
**Objetivo:** servir como contexto arquitetural permanente para um
agente de IA responsável por projetar, implementar, testar, revisar e
evoluir um sistema de software.

------------------------------------------------------------------------

## 1. Propósito deste documento

Este documento define os princípios, limites, padrões e decisões
arquiteturais que devem orientar a construção do sistema por meio de
**vibe coding**, desenvolvimento assistido por agentes de IA ou
combinação de desenvolvimento humano e IA.

O agente de IA deve tratar este documento como uma **Architecture
Constitution**: as decisões aqui descritas são padrões obrigatórios,
salvo quando uma decisão posterior, explicitamente registrada como ADR
(Architecture Decision Record), substituir uma regra.

O objetivo não é impor uma lista fixa de tecnologias. O objetivo é
garantir que o software permaneça:

-   modular;
-   testável;
-   observável;
-   seguro;
-   resiliente;
-   evolutivo;
-   desacoplado;
-   operacionalmente sustentável;
-   preparado para crescimento;
-   compreensível por humanos e agentes de IA.

> **Princípio central:** primeiro definir domínio, limites, contratos,
> dados e comportamento; somente depois escolher tecnologias.

------------------------------------------------------------------------

# 2. Princípios fundamentais

## 2.1 Domínio antes da tecnologia

Não iniciar uma decisão arquitetural pela escolha de framework, banco,
broker ou cloud.

A ordem preferencial é:

``` text
Domínio
  ↓
Casos de uso
  ↓
Bounded Contexts
  ↓
Contratos
  ↓
Dados
  ↓
Comunicação
  ↓
Consistência
  ↓
Resiliência
  ↓
Segurança
  ↓
Observabilidade
  ↓
Infraestrutura
  ↓
Tecnologias
```

## 2.2 Simplicidade antes da distribuição

Não criar um microsserviço apenas porque a tecnologia permite.

A arquitetura deve preferir:

**Modular Monolith → serviços independentes quando houver justificativa
real.**

A distribuição física deve ocorrer quando houver benefícios claros,
como:

-   escala independente;
-   ciclo de vida independente;
-   necessidade de isolamento;
-   equipes independentes;
-   requisitos de disponibilidade distintos;
-   tecnologias ou workloads incompatíveis;
-   necessidade de deployment independente.

Evitar o **Distributed Monolith**.

## 2.3 Baixo acoplamento

Módulos não devem acessar diretamente:

-   banco de outro domínio;
-   classes internas de outro módulo;
-   detalhes de infraestrutura de outro módulo;
-   credenciais ou configurações privadas de outro componente.

A comunicação deve ocorrer por:

-   interfaces;
-   casos de uso;
-   APIs;
-   eventos;
-   contratos explícitos.

## 2.4 Alta coesão

Cada módulo deve possuir uma responsabilidade de negócio clara.

Se um módulo acumular responsabilidades não relacionadas, considerar sua
divisão.

## 2.5 Fail gracefully

A falha de uma dependência externa não deve automaticamente derrubar o
sistema inteiro.

Para cada dependência, avaliar o risco e o comportamento necessário. Não
adicionar todos os mecanismos abaixo por padrão:

-   timeout;
-   retry;
-   circuit breaker;
-   fallback;
-   bulkhead;
-   idempotência;
-   degradação controlada.

## 2.6 Segurança por padrão

Segurança não é uma etapa posterior.

Toda funcionalidade nova deve considerar:

-   autenticação;
-   autorização;
-   validação de entrada;
-   proteção de dados;
-   auditoria;
-   secrets;
-   logs seguros;
-   rate limiting;
-   princípio do menor privilégio.

## 2.7 Observabilidade desde o início

Todo fluxo relevante deve poder responder:

-   o que aconteceu?
-   onde aconteceu?
-   com qual usuário/processo?
-   quando aconteceu?
-   quanto demorou?
-   qual dependência falhou?
-   qual foi o resultado?

## 2.8 Código deve ser compreensível por IA e humanos

O código deve privilegiar:

-   nomes explícitos;
-   funções pequenas;
-   módulos bem delimitados;
-   contratos claros;
-   documentação curta e contextual;
-   testes automatizados;
-   baixa complexidade acidental.

Evitar abstrações prematuras.

## 2.9 Aplicabilidade e níveis de exigência

As palavras **DEVE**, **RECOMENDA-SE** e **PODE** indicam, respectivamente,
requisito obrigatório, padrão recomendado e opção condicionada ao contexto.
Uma recomendação condicional não deve ser implementada sem que seu gatilho
seja identificado.

Antes de propor arquitetura ou infraestrutura, registrar um perfil conciso
do sistema com, no mínimo:

-   usuários, fluxos críticos e limites de domínio conhecidos;
-   requisitos de disponibilidade, latência, volume e crescimento, quando
  conhecidos;
-   classificação dos dados, requisitos de privacidade e obrigações
  regulatórias aplicáveis;
-   equipe responsável, capacidade operacional, orçamento e restrições
  tecnológicas;
-   estado atual da arquitetura, contratos existentes e ADRs relevantes.

O que não for conhecido deve ser marcado como hipótese ou questão em aberto,
em vez de ser convertido em requisito presumido. Para cada componente
opcional, o agente deve registrar o problema que resolve, a alternativa mais
simples considerada e o custo operacional introduzido. Se não houver
benefício verificável, deve preferir a alternativa mais simples.

Segurança, autorização, integridade dos dados e proteção de dados são
requisitos de base, proporcionais ao risco e ao tipo de dado; não são
justificativa para adotar automaticamente uma tecnologia específica. Os
demais mecanismos devem ser aplicados conforme os gatilhos descritos neste
documento e os requisitos do sistema.

------------------------------------------------------------------------

# 3. Arquitetura lógica de referência

A arquitetura deve ser organizada conceitualmente nas seguintes camadas:

``` text
┌──────────────────────────────────────────────────────────────┐
│                         CLIENTES                             │
│             Web / Mobile / Chat / Integrações               │
└──────────────────────────────┬───────────────────────────────┘
                               │
                         CDN / WAF / Edge
                               │
                         Load Balancer
                               │
                       ┌───────▼────────┐
                       │   API GATEWAY  │
                       └───────┬────────┘
                               │
                  ┌────────────┼────────────┐
                  │            │            │
                  ▼            ▼            ▼
                BFF Web    BFF Mobile   BFF Chat
                  │            │            │
                  └────────────┼────────────┘
                               │
               ┌───────────────▼────────────────┐
               │         APPLICATION CORE       │
               │                                │
               │ DDD / Bounded Contexts         │
               │                                │
               │ ┌───────┐ ┌───────┐ ┌───────┐│
               │ │Dom A  │ │Dom B  │ │Dom C  ││
               │ └───────┘ └───────┘ └───────┘│
               └───────────────┬────────────────┘
                               │
                ┌──────────────┴──────────────┐
                │                             │
           Síncrono                      Assíncrono
                │                             │
                ▼                             ▼
            REST/gRPC                    Event Broker
                                              │
                                      ┌───────┼───────┐
                                      ▼       ▼       ▼
                                    Worker  Worker    AI
                                                        │
                                      ┌─────────────────┤
                                      ▼                 ▼
                                Workflow Engine     AI Gateway
                                                        │
                                                        ▼
                                                  LLM / RAG / Tools

┌────────────────────────────────────────────────────────────────────┐
│                           DATA PLATFORM                             │
│ PostgreSQL │ Redis │ Object Storage │ Event Store │ Analytics      │
└────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────┐
│                         CROSS-CUTTING                               │
│ IAM │ Secrets │ Audit │ Security │ Config │ Resilience │ Governance │
└────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────┐
│                        OBSERVABILITY                                │
│ OpenTelemetry │ Logs │ Metrics │ Traces │ SLO │ Alerting           │
└────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────┐
│                           PLATFORM                                  │
│ Containers │ Kubernetes* │ IaC │ CI/CD │ Registry │ Backup │ DR    │
└────────────────────────────────────────────────────────────────────┘
```

`Kubernetes` é opcional e deve ser adotado apenas quando houver
justificativa operacional.

O diagrama é conceitual: Gateway, BFF, broker, workflow engine, AI Gateway e
componentes de dados separados são opções, não etapas obrigatórias de toda
implementação.

------------------------------------------------------------------------

# 4. Estrutura interna da aplicação

A estrutura lógica preferencial de cada domínio é:

``` text
domain/
├── domain-model/
├── application/
├── infrastructure/
└── interfaces/
```

Uma implementação típica:

``` text
src/
├── modules/
│   ├── users/
│   │   ├── domain/
│   │   ├── application/
│   │   ├── infrastructure/
│   │   └── interfaces/
│   │
│   ├── orders/
│   │   ├── domain/
│   │   ├── application/
│   │   ├── infrastructure/
│   │   └── interfaces/
│   │
│   └── notifications/
│       ├── domain/
│       ├── application/
│       ├── infrastructure/
│       └── interfaces/
│
├── shared/
└── bootstrap/
```

## Regra

`shared/` não deve virar um depósito de funcionalidades.

Somente elementos realmente transversais devem ser compartilhados.

------------------------------------------------------------------------

# 5. Domain-Driven Design

O sistema deve ser dividido por **Bounded Contexts**, e não por
tecnologias.

Exemplo:

``` text
Sistema
├── Identity
├── Users
├── Atendimento
├── Billing
├── Notifications
├── Reporting
└── Integrations
```

Cada contexto possui:

-   modelo próprio;
-   regras próprias;
-   casos de uso;
-   contratos;
-   eventos;
-   fronteiras de dados.

## 5.1 Regra de propriedade

Cada dado deve possuir um **owner** claro.

Um domínio não deve modificar diretamente o banco de outro domínio.

Errado:

``` text
Domínio A
   ↓
SELECT tabela_do_Domínio_B
```

Preferir:

``` text
Domínio A
   ↓
API / Use Case / Evento
   ↓
Domínio B
```

Em um monólito modular, a propriedade deve ser verificável por convenções
explícitas: módulo responsável por cada tabela, migrations sob controle
desse módulo e acesso a outros domínios somente por interfaces públicas.
Quando viável, reforçar essas fronteiras com regras de dependência, testes
de arquitetura ou schemas separados. Compartilhar o mesmo processo ou banco
não transfere a propriedade dos dados.

------------------------------------------------------------------------

# 6. Modular Monolith como ponto de partida

Quando não houver necessidade explícita de distribuição:

``` text
Application
├── Domain A
├── Domain B
├── Domain C
├── Integration
└── Notification
```

Cada módulo deve possuir fronteiras internas equivalentes às de um
futuro serviço.

Isso permite posteriormente:

``` text
Modular Monolith
       │
       ├──► Serviço A
       ├──► Serviço B
       └──► Serviço C
```

sem reescrever o domínio.

------------------------------------------------------------------------

# 7. API Gateway

Entradas externas devem possuir uma camada de entrada apropriada. Adotar um
API Gateway dedicado quando houver necessidade concreta de políticas
centralizadas, roteamento entre serviços, quotas ou proteção de múltiplos
consumidores. Em aplicações menores, o ingress, o proxy da plataforma ou a
própria aplicação podem atender a essas necessidades sem um Gateway
dedicado.

Responsabilidades:

-   autenticação inicial;
-   roteamento;
-   rate limiting;
-   quotas;
-   versionamento;
-   observabilidade;
-   proteção contra abuso;
-   políticas de acesso de borda.

O Gateway **não deve conter regra de negócio**.

------------------------------------------------------------------------

# 8. BFF --- Backend for Frontend

Quando diferentes canais possuírem necessidades significativamente
diferentes, utilizar BFFs.

``` text
Web ───────► BFF Web
Mobile ────► BFF Mobile
Chat ──────► BFF Chat
                  │
                  ▼
            APIs de domínio
```

O BFF deve adaptar a experiência do canal sem duplicar regras de
negócio.

------------------------------------------------------------------------

# 9. Comunicação

## 9.1 Comunicação síncrona

Utilizar REST ou gRPC quando:

-   é necessária resposta imediata;
-   a operação é simples;
-   o consumidor precisa do resultado naquele momento.

## 9.2 Comunicação assíncrona

Utilizar eventos/mensagens quando:

-   o processamento pode ocorrer posteriormente;
-   há múltiplos consumidores;
-   é necessário desacoplamento temporal;
-   o processo pode ser longo;
-   o sistema precisa absorver picos de carga.

``` text
Producer
   │
   ▼
Event Broker
   │
   ├──► Consumer A
   ├──► Consumer B
   └──► Consumer C
```

------------------------------------------------------------------------

# 10. Mensageria

O broker não deve ser tratado apenas como uma fila genérica.

Definir:

-   nome do evento;
-   versão;
-   produtor;
-   consumidores;
-   schema;
-   política de retenção;
-   retry;
-   DLQ;
-   idempotência;
-   ordenação, quando necessária;
-   observabilidade.

Assumir entrega **pelo menos uma vez** salvo garantia explícita e comprovada
da plataforma. Consumidores devem tolerar duplicatas por idempotência ou
deduplicação. Definir compatibilidade de schema, chave de ordenação quando
necessária e política de retenção/reprocessamento antes de publicar eventos
que sustentem fluxos críticos. Não prometer exatamente uma vez de ponta a
ponta sem demonstrar como os efeitos externos também são protegidos.

## 10.1 Dead Letter Queue

Mensagens que não podem ser processadas após as tentativas definidas
devem ir para DLQ.

Nunca descartar silenciosamente mensagens importantes.

DLQs devem ter alerta, responsável, prazo de retenção e procedimento seguro
de inspeção e reprocessamento. O reprocessamento não pode duplicar efeitos
de negócio.

## 10.2 Retry

Retry deve ser limitado e controlado.

Aplicar retry somente a falhas transitórias e operações repetíveis com
segurança. Definir limite de tentativas e prazo total compatível com o
timeout do fluxo; usar backoff exponencial com jitter. Não fazer retry em
paralelo em várias camadas sem orçamento coordenado, pois isso pode ampliar
a carga durante uma falha.

Preferir:

``` text
Retry
 ↓
Backoff
 ↓
Retry
 ↓
Backoff maior
 ↓
DLQ
```

Evitar loops infinitos.

------------------------------------------------------------------------

# 11. Outbox Pattern

Quando uma operação precisa atualizar o banco e publicar um evento,
utilizar Outbox quando a consistência exigir.

``` text
TRANSAÇÃO
├── Atualiza dados
└── Registra evento na Outbox
          │
          ▼
    Event Publisher
          │
          ▼
      Event Broker
```

Isso evita a inconsistência:

``` text
Banco = sucesso
Broker = falha
```

------------------------------------------------------------------------

# 12. Workflow Engine

Não usar o broker como substituto de um motor de workflow.

Processos longos ou duráveis, com retomada após falha, espera por interação
humana, compensações ou coordenação entre sistemas devem ter estado e
transições explícitos. Usar um workflow engine quando persistência,
visibilidade operacional e recuperação do processo justificarem a
dependência; fluxos curtos e locais podem ser modelados como casos de uso ou
máquinas de estado na própria aplicação.

Exemplo:

``` text
Receber solicitação
       ↓
Validar
       ↓
Consultar sistema A
       ↓
Aguardar
       ↓
Consultar sistema B
       ↓
Aprovação
       ↓
Executar
       ↓
Finalizar
```

O workflow deve controlar:

-   estado;
-   transições;
-   timeout;
-   retry;
-   compensação;
-   aprovação;
-   retomada após falha.

------------------------------------------------------------------------

# 13. Resiliência

Toda chamada externa DEVE ter timeout explícito. Para os demais mecanismos,
avaliar a falha que se pretende conter e o custo da proteção:

### Timeout

Nunca aguardar indefinidamente.

### Retry

Somente para falhas transitórias, com limite, backoff e jitter. A operação
deve ser idempotente ou usar uma chave de idempotência; retries devem
respeitar o prazo total do caso de uso.

### Circuit Breaker

Usar quando chamadas repetidas a uma dependência instável puderem consumir
recursos ou propagar falhas. Definir limiares e comportamento de recuperação;
não substituir timeout ou tratamento de erro.

### Bulkhead

Isolar pools, filas ou limites de concorrência quando recursos
compartilhados puderem permitir que um workload esgote a capacidade de
outros.

### Fallback

Oferecer comportamento alternativo somente quando preservar semântica e
segurança. Não transformar indisponibilidade em sucesso aparente nem
retornar dados obsoletos sem que isso seja aceitável para o caso de uso.

### Idempotência

Uma mesma requisição processada duas vezes não deve produzir efeitos
indevidos.

------------------------------------------------------------------------

# 14. Arquitetura de dados

A escolha do armazenamento deve partir do comportamento dos dados.

## 14.1 PostgreSQL

Preferencial para dados transacionais estruturados.

## 14.2 Redis

Para:

-   cache;
-   sessões;
-   dados temporários;
-   mecanismos de coordenação.

Não utilizar Redis como banco principal sem justificativa explícita.

## 14.3 Object Storage

Para:

-   arquivos;
-   documentos;
-   imagens;
-   grandes objetos.

## 14.4 Document Database

Utilizar apenas quando o modelo documental justificar sua adoção.

## 14.5 Analytics

Não executar consultas analíticas pesadas diretamente no banco
transacional quando isso puder afetar a operação.

Preferir:

``` text
OLTP
 ↓
CDC / Events
 ↓
Data Platform
 ↓
Analytics / BI / ML
```

------------------------------------------------------------------------

# 15. Consistência de dados

Diferenciar explicitamente:

-   consistência forte;
-   consistência eventual;
-   transação local;
-   transação distribuída.

Evitar transações distribuídas quando eventos e compensações forem
suficientes.

Preferir:

``` text
Transação local
       +
Evento
       +
Processamento idempotente
```

------------------------------------------------------------------------

# 16. Integrações externas

Nunca espalhar chamadas a terceiros pelo código de domínio.

Criar adapters:

``` text
Domain
  │
  ▼
Integration Port
  │
  ▼
Adapter
  │
  ▼
External API
```

Cada integração deve encapsular:

-   autenticação;
-   transformação;
-   timeout;
-   retry;
-   circuit breaker;
-   rate limit;
-   tratamento de erros;
-   versionamento;
-   observabilidade.

O domínio não deve conhecer detalhes da API externa.

------------------------------------------------------------------------

# 17. Segurança

## 17.1 Identity

Preferir padrões como:

-   OAuth 2.x;
-   OpenID Connect;
-   tokens de curta duração;
-   MFA quando aplicável.

## 17.2 Authorization

Utilizar RBAC, ABAC ou política equivalente conforme a complexidade.

Nunca assumir que autenticação implica autorização.

## 17.3 Least Privilege

Cada usuário, serviço e agente deve possuir somente as permissões
necessárias.

## 17.4 Secrets

Secrets nunca devem ficar:

-   no código;
-   no Git;
-   em imagens Docker;
-   em arquivos públicos;
-   em logs.

Utilizar Secret Manager/Vault/KMS ou mecanismo equivalente.

## 17.5 Encryption

Considerar:

-   TLS em trânsito;
-   criptografia em repouso;
-   gestão adequada de chaves.

## 17.6 Audit

Operações sensíveis devem gerar registros de auditoria.

Auditoria não deve ser confundida com log técnico.

------------------------------------------------------------------------

# 18. Zero Trust

Não confiar em um serviço simplesmente porque ele está dentro da rede.

Cada comunicação relevante deve considerar:

``` text
Quem é?
Pode executar?
Sobre qual recurso?
Qual operação?
Em qual contexto?
```

Quando apropriado, utilizar:

-   identidade de workload;
-   mTLS;
-   políticas de rede;
-   autorização serviço-a-serviço.

------------------------------------------------------------------------

# 19. Observabilidade

A arquitetura deve adotar uma abordagem de observabilidade baseada em:

-   Logs;
-   Metrics;
-   Traces.

Preferir OpenTelemetry como camada de instrumentação quando compatível
com a stack.

## 19.1 Correlation ID

Todo fluxo distribuído deve propagar contexto de rastreamento ou um
identificador de correlação consistente entre serviços, mensagens e
integrações. Em sistemas locais, não criar uma infraestrutura distribuída
apenas para cumprir esta regra.

``` text
Request
 ↓
Gateway
 ↓
Backend
 ↓
Broker
 ↓
Worker
 ↓
External API
```

O mesmo contexto de correlação deve ser preservado.

## 19.2 Métricas

Monitorar, no mínimo:

-   throughput;
-   latência;
-   erros;
-   saturação;
-   filas;
-   consumo de recursos;
-   dependências externas.

## 19.3 SLI/SLO

Definir indicadores e objetivos mensuráveis.

Exemplo:

``` text
SLI:
percentual de requisições concluídas em menos de 2 segundos.

SLO:
99,5%.
```

------------------------------------------------------------------------

# 20. Auditoria

Criar trilha de auditoria para operações relevantes.

Exemplo:

``` text
actor
action
resource
timestamp
source
result
correlation_id
```

Evitar armazenar dados sensíveis desnecessários.

------------------------------------------------------------------------

# 21. Inteligência Artificial

IA deve ser tratada como componente arquitetural controlado, não como
acesso irrestrito ao sistema.

Arquitetura conceitual:

``` text
Application
     │
     ▼
AI Gateway
     │
     ├── Model A
     ├── Model B
     └── Local Model
          │
          ▼
    AI Orchestrator
          │
     ┌────┼────┐
     ▼    ▼    ▼
   Tools RAG Agents
```

## 21.1 AI Gateway

Deve controlar:

-   seleção de modelo;
-   autenticação;
-   quotas;
-   custos;
-   observabilidade;
-   fallback;
-   políticas.

## 21.2 Agents

Agentes devem possuir permissões explícitas.

Nunca conceder acesso irrestrito.

Exemplo:

``` text
Agent A
├── pode consultar clientes
├── pode gerar relatório
├── não pode excluir dados
└── não pode executar operação financeira
```

Ferramentas devem aplicar autorização no servidor a cada operação; instruções
no prompt não são controle de acesso. Validar argumentos e saídas antes de
efetivar ações, limitar dados enviados a modelos externos e exigir
confirmação humana para operações irreversíveis ou de alto impacto. Definir
limites de custo e comportamento para indisponibilidade do modelo.

## 21.3 Prompt injection

Tratar conteúdo externo como não confiável.

Não permitir que conteúdo recuperado por RAG ou ferramentas altere
silenciosamente as políticas do agente.

## 21.4 RAG

Separar:

-   ingestão;
-   indexação;
-   recuperação;
-   geração;
-   avaliação.

## 21.5 Avaliação

Modelos e prompts devem possuir testes e avaliações antes de serem
considerados aptos para produção.

Avaliar, conforme o risco, qualidade da resposta, segurança, vazamento de
dados, uso de ferramentas, latência e custo. Registrar critérios de aceite e
reavaliar quando modelo, prompt, ferramentas ou fontes de dados mudarem.

------------------------------------------------------------------------

# 22. CI/CD

Pipeline mínimo:

``` text
Commit
  ↓
Build
  ↓
Unit Tests
  ↓
Integration Tests
  ↓
Contract Tests
  ↓
SAST
  ↓
SCA / Dependency Scan
  ↓
Container Scan
  ↓
Quality Gate
  ↓
Artifact Registry
  ↓
Deploy
```

Nenhum agente deve considerar uma funcionalidade concluída somente
porque o código compila.

------------------------------------------------------------------------

# 23. Testes

A estratégia deve possuir diferentes níveis:

``` text
                 E2E
                 ▲
              Contract
                 ▲
            Integration
                 ▲
                Unit
```

Prioridade:

1.  testes unitários de domínio;
2.  testes de aplicação;
3.  testes de integração;
4.  testes de contrato;
5.  testes E2E para fluxos críticos.

Não utilizar E2E como substituto de testes unitários.

------------------------------------------------------------------------

# 24. Infrastructure as Code

Infraestrutura deve ser reproduzível.

Evitar configuração manual como dependência permanente.

Definir infraestrutura através de código quando possível.

Exemplos de responsabilidades:

-   rede;
-   banco;
-   filas;
-   storage;
-   observabilidade;
-   secrets;
-   compute;
-   permissões.

------------------------------------------------------------------------

# 25. Containers

Cada container deve:

-   possuir responsabilidade clara;
-   ser imutável;
-   executar com usuário não privilegiado quando possível;
-   não armazenar estado persistente localmente;
-   possuir health checks;
-   possuir limites de recursos;
-   possuir imagem mínima e atualizada.

------------------------------------------------------------------------

# 26. Kubernetes

Kubernetes é opcional.

Só utilizar quando a complexidade justificar.

Se utilizado, considerar:

-   namespaces;
-   resource requests/limits;
-   readiness probes;
-   liveness probes;
-   autoscaling;
-   network policies;
-   secrets;
-   observabilidade;
-   rolling deployments.

Não introduzir Kubernetes apenas para parecer "cloud native".

------------------------------------------------------------------------

# 27. Deployments

Para sistemas críticos, considerar:

-   rolling deployment;
-   blue/green;
-   canary;
-   feature flags;
-   rollback automático.

A estratégia deve ser escolhida de acordo com risco e complexidade.

------------------------------------------------------------------------

# 28. Disaster Recovery

Definir explicitamente:

## RPO

Quantidade máxima de dados que pode ser perdida.

## RTO

Tempo máximo aceitável para recuperação.

A arquitetura deve contemplar:

-   backups;
-   restauração testada;
-   replicação quando necessária;
-   recuperação de infraestrutura;
-   documentação de procedimentos;
-   testes periódicos de disaster recovery.

Backup que nunca foi restaurado não deve ser considerado comprovadamente
confiável.

------------------------------------------------------------------------

# 29. Gestão de configuração

Separar:

``` text
Código
≠
Configuração
≠
Secret
```

Configurações por ambiente devem ser externalizadas.

Secrets devem possuir tratamento específico.

------------------------------------------------------------------------

# 30. Feature Flags

Funcionalidades de alto risco devem poder ser ativadas ou desativadas
sem novo deployment quando apropriado.

Utilizar feature flags para:

-   rollout gradual;
-   testes controlados;
-   rollback funcional;
-   funcionalidades experimentais.

Evitar acumular flags antigas indefinidamente.

------------------------------------------------------------------------

# 31. Versionamento de APIs

APIs públicas devem possuir estratégia explícita de evolução.

Exemplo:

``` text
/v1/resource
/v2/resource
```

Evitar breaking changes silenciosas.

Contratos devem possuir testes automatizados.

------------------------------------------------------------------------

# 32. Contratos de eventos

Eventos devem ser considerados contratos.

Exemplo conceitual:

``` json
{
  "event_type": "OrderCreated",
  "version": 1,
  "event_id": "...",
  "occurred_at": "...",
  "correlation_id": "...",
  "payload": {}
}
```

Consumidores devem tolerar evolução compatível do schema.

------------------------------------------------------------------------

# 33. ADR --- Architecture Decision Records

Toda decisão arquitetural relevante deve gerar ADR.

Template:

``` markdown
# ADR-XXX: Título

## Status
Proposed | Accepted | Superseded | Rejected

## Contexto
Por que a decisão é necessária?

## Decisão
O que foi decidido?

## Alternativas
Quais alternativas foram consideradas?

## Consequências
Quais benefícios e custos surgem?

## Data
YYYY-MM-DD
```

O agente deve consultar ADRs antes de propor mudanças que possam
contradizê-los.

------------------------------------------------------------------------

# 34. Regras específicas para o agente de IA

Estas regras são obrigatórias durante o desenvolvimento assistido por
IA.

## 34.1 Contextualizar antes de implementar

Antes de decidir uma arquitetura, o agente deve identificar o perfil do
sistema descrito na seção 2.9. Distinguir fatos, requisitos, hipóteses e
questões em aberto. Não preencher lacunas com infraestrutura ou requisitos
de escala presumidos.

## 34.2 Não implementar cegamente

Antes de modificar o sistema:

1.  localizar o módulo;
2.  entender o fluxo;
3.  identificar dependências;
4.  verificar contratos;
5.  verificar testes;
6.  verificar ADRs;
7.  só então alterar o código.

## 34.3 Não criar tecnologia sem necessidade

Não adicionar:

-   novo banco;
-   novo framework;
-   novo broker;
-   novo microsserviço;
-   nova biblioteca;

sem justificar a necessidade.

## 34.4 Não duplicar regra de negócio

Se uma regra já existe, reutilizar o caso de uso ou domínio existente.

Não copiar e adaptar.

## 34.5 Não acessar banco de outro domínio

Respeitar ownership.

## 34.6 Não colocar regra de negócio em controller

Controllers devem tratar:

-   entrada;
-   autenticação/autorização;
-   validação superficial;
-   chamada do caso de uso;
-   resposta.

A regra pertence ao domínio/aplicação.

## 34.7 Não colocar regra de negócio em infraestrutura

Adapters, repositories e clients externos não devem possuir regras de
negócio.

## 34.8 Toda funcionalidade nova deve possuir testes

O agente deve criar ou atualizar testes relacionados à alteração.

## 34.9 Não considerar "compila" como "pronto"

Uma implementação só pode ser considerada concluída após:

-   testes;
-   validação de contrato;
-   análise de segurança relevante;
-   observabilidade;
-   tratamento de erros;
-   documentação necessária.

## 34.10 Tornar decisões verificáveis

Para cada decisão condicional relevante, registrar o gatilho, a alternativa
considerada e a justificativa. Quando viável, impor fronteiras de módulos e
contratos por testes automatizados, regras de dependência ou validações de
pipeline. Não criar verificações sem risco concreto que elas reduzam.

------------------------------------------------------------------------

# 35. Definition of Done arquitetural

Uma funcionalidade é considerada pronta quando:

-   [ ] domínio está corretamente delimitado;
-   [ ] caso de uso está identificado;
-   [ ] regras de negócio estão no local correto e testes relevantes cobrem
  o comportamento alterado;
-   [ ] autorização e validação de entrada foram tratadas quando aplicáveis;
-   [ ] erros e efeitos sobre dados sensíveis foram avaliados;
-   [ ] chamadas externas têm timeout e política de repetição segura;
-   [ ] contratos de API ou eventos foram atualizados e testados quando
  alterados;
-   [ ] logs, métricas e rastreamento foram atualizados quando necessários
  para operar o fluxo;
-   [ ] testes de integração, contrato ou ponta a ponta foram adicionados
  quando o risco e a fronteira alterada justificarem;
-   [ ] documentação e ADR foram atualizados quando necessários.

Os itens condicionais devem ser marcados como não aplicáveis com uma
justificativa breve, não implementados mecanicamente. Requisitos de
segurança, autorização e integridade aplicáveis ao fluxo não podem ser
dispensados por conveniência.

------------------------------------------------------------------------

# 36. Anti-padrões proibidos

O agente deve evitar:

### Big Ball of Mud

Código sem fronteiras claras.

### Distributed Monolith

Muitos serviços fortemente acoplados.

### Shared Database

Múltiplos domínios escrevendo diretamente nas mesmas tabelas sem
ownership.

### God Service

Um serviço que concentra praticamente todo o sistema.

### God Object

Uma classe com responsabilidades excessivas.

### Smart Controller

Controller contendo regra de negócio.

### Smart Gateway

Gateway contendo lógica de negócio.

### Shared Everything

Todos os módulos dependendo das mesmas classes internas.

### Retry Storm

Retries simultâneos sobre uma dependência indisponível.

### Infinite Queue

Filas crescendo indefinidamente sem controle.

### Silent Failure

Erro sem log, métrica, evento ou tratamento.

### Secret in Code

Credenciais armazenadas no código ou repositório.

### AI with unrestricted access

Agente de IA com permissões superiores às necessárias.

### Technology Driven Architecture

Escolher arquitetura para justificar tecnologia, em vez de escolher
tecnologia para atender à arquitetura.

------------------------------------------------------------------------

# 37. Estratégia de evolução

A arquitetura deve evoluir incrementalmente.

Preferência:

``` text
MVP
 ↓
Modular Monolith
 ↓
Observabilidade
 ↓
Resiliência
 ↓
Escala
 ↓
Extração seletiva de serviços
 ↓
Distribuição somente quando necessária
```

Não começar com dezenas de serviços.

------------------------------------------------------------------------

# 38. Critérios para extrair um microsserviço

Antes de separar um módulo, responder:

1.  O domínio possui fronteira clara?
2.  Possui ciclo de vida próprio?
3.  Precisa escalar independentemente?
4.  Precisa de deployment independente?
5.  Tem requisitos de disponibilidade diferentes?
6.  Pode possuir ownership independente?
7.  A comunicação distribuída é justificável?
8.  O custo operacional compensa?
9.  Existe observabilidade suficiente?
10. Existe estratégia de consistência e falha?

Uma necessidade comprovada de escala, isolamento, ciclo de vida ou
disponibilidade independente pode justificar a extração. Uma resposta
positiva isolada não basta: avaliar o custo de operação, a latência,
consistência, observabilidade, ownership e capacidade da equipe para operar
o serviço. Se não houver benefício mensurável que compense esses custos,
manter o módulo no monólito modular.

------------------------------------------------------------------------

# 39. Princípio de evolução tecnológica

Tecnologias são substituíveis.

Os seguintes elementos devem ser mais estáveis:

``` text
Domínio
Contratos
Casos de uso
Regras de negócio
Eventos
Políticas
```

Os seguintes elementos devem ser substituíveis quando necessário:

``` text
Framework
Banco
Broker
Cloud
Container runtime
LLM
Fornecedor externo
Ferramenta de observabilidade
```

A arquitetura deve proteger o domínio dessas mudanças.

------------------------------------------------------------------------

# 40. Resultado arquitetural esperado

Ao final, o sistema deve possuir estas propriedades:

``` text
                    ┌──────────────────────┐
                    │       DOMÍNIO        │
                    │                      │
                    │  Alta coesão         │
                    │  Baixo acoplamento   │
                    │  Regras isoladas     │
                    └──────────┬───────────┘
                               │
                 ┌─────────────┼─────────────┐
                 ▼             ▼             ▼
              APIs          Eventos       Workflows
                 │             │             │
                 └─────────────┼─────────────┘
                               ▼
                       Integrações / Dados
                               │
                 ┌─────────────┼─────────────┐
                 ▼             ▼             ▼
             Segurança    Observabilidade  Resiliência
                 │             │             │
                 └─────────────┼─────────────┘
                               ▼
                         Infraestrutura
```

O objetivo final não é maximizar o número de componentes.

É maximizar:

**coesão + baixo acoplamento + testabilidade + resiliência + segurança +
observabilidade + capacidade de evolução.**

------------------------------------------------------------------------

# 41. Regra final para o agente

Antes de implementar qualquer decisão relevante, o agente deve perguntar
internamente:

> **"Esta solução reduz ou aumenta o acoplamento? Esta decisão mantém o
> domínio protegido? O que acontece quando esta dependência falhar? Como
> vou observar o comportamento? Como vou testar? Como evoluirei esta
> parte depois?"**

Se uma implementação não tiver boas respostas para essas perguntas, ela
deve ser revista antes de ser incorporada à arquitetura.

------------------------------------------------------------------------

# 42. Prioridade das decisões

Quando houver conflito entre objetivos, utilizar esta ordem:

1.  **Correção do domínio**
2.  **Segurança**
3.  **Integridade dos dados**
4.  **Resiliência**
5.  **Observabilidade**
6.  **Testabilidade**
7.  **Manutenibilidade**
8.  **Escalabilidade**
9.  **Performance**
10. **Otimização de custo**
11. **Conveniência tecnológica**

Segurança, autorização, privacidade e integridade dos dados são condições
mínimas de aceite, não itens que possam ser sacrificados por desempenho,
conveniência ou disponibilidade. Requisitos mensuráveis do produto devem
orientar os trade-offs restantes; na ausência deles, registrar a hipótese e
evitar otimizações prematuras.

Performance ou escala nunca devem justificar a introdução de
complexidade arquitetural sem evidência de necessidade.

------------------------------------------------------------------------

# 43. Resumo executivo

A arquitetura de referência segue o princípio:

``` text
DDD
 +
Modularidade
 +
APIs
 +
Eventos
 +
Workflows
 +
Resiliência
 +
Security by Design
 +
Observabilidade
 +
Data Ownership
 +
CI/CD
 +
IaC
 +
AI Governance
```

A implementação deve começar simples, mas com **fronteiras arquiteturais
fortes**.

O sistema deve poder evoluir de:

``` text
Modular Monolith
```

para:

``` text
Distributed Architecture
```

sem exigir uma reconstrução completa.

A tecnologia deve servir à arquitetura --- e não o contrário.
