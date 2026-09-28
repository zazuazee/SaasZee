# Complemento Arquitetural: SaaS e OaaS

**Versão:** 1.0  
**Data:** 2026-09-28  
**Documento-base:** [Arquitetura de Referência](../ARQUITETURA_REFERENCIA_VIBE_CODING.md)

## 1. Propósito e precedência

Este complemento adiciona requisitos para produtos entregues como serviço. Deve ser lido junto com a Arquitetura de Referência, que continua sendo a fonte dos princípios comuns de domínio, modularidade, segurança, resiliência, testes e operação. Não substitui a especificação de produto nem ADRs.

Neste conjunto de documentos, **OaaS** significa *Operations as a Service*: além de disponibilizar software, o fornecedor executa ou coordena processos operacionais para seus clientes. Se o projeto usar outra definição para OaaS, registrá-la no perfil do projeto antes de aplicar este complemento.

Uma plataforma pode oferecer SaaS sem OaaS. OaaS acrescenta pessoas, filas de trabalho, delegações, SLAs operacionais e evidências de execução; não é apenas outro nome para SaaS.

## 2. Perfil obrigatório do produto

Antes de escolher arquitetura, registrar:

- quem contrata, quem administra a conta e quem usa o produto;
- o que constitui um tenant e se há unidades, filiais ou equipes subordinadas;
- quais dados e operações são compartilhados, confidenciais ou regulados;
- regiões de hospedagem, residência, retenção e exportação de dados exigidas;
- volumes esperados, sazonalidade, disponibilidade e latência necessárias;
- modelo de planos, medição de uso, limites e política de cobrança, se houver;
- quais atividades o fornecedor executa em nome do cliente no modelo OaaS;
- requisitos de suporte, delegação, aprovação, auditoria e recuperação;
- obrigações legais e contratuais validadas para o país e setor do produto.

Fatos, hipóteses e questões em aberto devem ficar separados. Não presumir que cada cliente equivale a uma única unidade física ou que todos os clientes precisam do mesmo nível de isolamento.

## 3. Fronteiras da plataforma

Separar conceitualmente, sem exigir processos ou deploys separados:

- **Capacidades compartilhadas da plataforma:** identidade, tenants e memberships, autorização, configuração, provisionamento, planos e entitlements, medição, auditoria, notificações, suporte e administração.
- **Domínios do produto:** regras clínicas, jurídicas ou operacionais específicas de cada vertical; pertencem aos complementos de domínio.
- **Plano de controle:** administração do tenant, provisionamento, configuração e ciclo de vida.
- **Plano de dados:** registros e operações do cliente.

O plano de controle e o plano de dados podem começar no mesmo monólito modular. Separá-los fisicamente apenas diante de requisitos comprovados de segurança, disponibilidade, escala, ownership ou ciclo de vida.

Não armazenar regras profissionais nos módulos genéricos de tenant, cobrança ou identidade. Não transformar a plataforma compartilhada em um domínio universal que misture prontuários, processos jurídicos e operações de setores diferentes.

## 4. Tenant e identidade

### 4.1 Modelo conceitual

O modelo deve distinguir, quando aplicável:

- **Tenant:** organização contratante e fronteira primária de dados e configuração.
- **Unidade:** clínica, hospital, escritório, filial ou local pertencente ao tenant.
- **Usuário:** identidade individual, que pode pertencer a um ou mais tenants.
- **Membership:** vínculo explícito do usuário com um tenant, seu estado e suas permissões.
- **Papel e permissão:** capacidades concedidas no escopo apropriado, sem inferir autorização apenas do cargo.
- **Entitlement:** recurso habilitado por plano ou contrato; não substitui autorização de acesso.

A cardinalidade e os termos podem variar por produto. A decisão deve refletir a organização real do cliente.

### 4.2 Isolamento e autorização

- O tenant ativo deve ser derivado e validado pelo servidor a partir da identidade e dos vínculos autorizados; um `tenant_id` enviado pelo cliente nunca é prova de acesso.
- Cada leitura, escrita, busca, exportação e tarefa em background deve aplicar o escopo de tenant e a autorização da operação.
- Propagar contexto autenticado de tenant entre serviços e mensagens por mecanismo confiável, validável e com finalidade explícita.
- Isolar também arquivos, cache, índices de busca, eventos, relatórios, telemetria com dados identificáveis, exports e backups.
- Usar acesso negado por padrão. Autorização por papel pode ser complementada por atributos como unidade, equipe, vínculo com caso ou relação profissional.
- Row-level security, quando disponível, é uma camada adicional; não substitui autorização de aplicação, testes e controle de credenciais.
- Segredos, tokens, URLs assinadas e chaves de cache não devem permitir acesso acidental entre tenants.

### 4.3 Opções de isolamento

A decisão entre tabelas compartilhadas, schema por tenant, banco por tenant ou implantação dedicada deve ser registrada em ADR. Comparar custo, blast radius, requisitos de residência, backup/restauração, migrações, performance, operação e possibilidade de evolução. Permitir perfis distintos somente se o custo de suportá-los for justificável.

Qualquer opção escolhida deve ter uma estratégia para migrar tenants entre níveis de isolamento sem perda de autorização, auditoria ou integridade.

## 5. Ciclo de vida do tenant

Modelar estados e transições explícitos para:

- criação, validação, provisionamento e onboarding;
- configuração inicial, ativação e mudanças de plano;
- suspensão, reativação e encerramento;
- exportação, retenção, bloqueio legal quando aplicável e exclusão;
- migração de região ou nível de isolamento, quando suportada.

Cada transição deve definir autorizador, pré-condições, efeitos, notificações, idempotência, trilha de auditoria e tratamento de falha. Exclusão deve considerar cópias, índices, arquivos, backups e períodos de retenção aplicáveis; documentar limites temporais de expurgo.

## 6. Administração e suporte entre tenants

Acesso de suporte a dados de cliente deve ser excepcional, não uma permissão global permanente. Exigir finalidade registrada, identidade individual, autorização adequada, escopo mínimo, duração limitada e trilha de auditoria protegida contra alteração. Quando o risco justificar, exigir aprovação do cliente ou dupla aprovação interna.

Ferramentas internas devem separar metadados operacionais de conteúdo clínico, psicológico ou jurídico. Consultas de suporte não podem contornar políticas de domínio ou virar uma interface de acesso irrestrito ao banco.

## 7. OaaS: processos operados pelo fornecedor

Para cada serviço operacional, documentar:

- solicitante, beneficiário, responsável interno e tenant proprietário;
- estados, transições permitidas, pré-condições e critérios de conclusão;
- fila, atribuição, reatribuição, prioridade e capacidade;
- início, pausa, calendário e encerramento do relógio de SLA;
- prazo, escalonamento, notificações e procedimento de exceção;
- dados e evidências necessários, sua retenção e quem pode consultá-los;
- ações automatizadas, ações humanas e limites de autoridade;
- cancelamento, correção, retrabalho e compensação após falha.

Distinguir o estado técnico do workflow do estado de negócio apresentado ao cliente. Registrar ator, timestamp, tenant, operação e resultado de alterações relevantes. Idempotência é obrigatória para comandos que possam ser repetidos por retry, reentrega ou ação do operador.

Uma fila ou workflow engine não é requisito por si só. Selecionar a implementação conforme durabilidade, volume, retomada, auditoria, operação humana e custo.

## 8. Planos, medição e cobrança

Quando houver cobrança baseada em plano ou uso:

- separar catálogo comercial, entitlement e autorização;
- definir a unidade de medição, origem, período, arredondamento e tratamento de correções;
- tornar registros de uso rastreáveis e deduplicáveis;
- não bloquear operações clínicas, jurídicas ou de segurança de forma inesperada por divergência de medição;
- definir comportamento para limites atingidos, inadimplência, downgrade e cancelamento;
- manter dados financeiros e de uso com acesso, retenção e auditoria apropriados.

O provedor de pagamentos deve ser integrado por adapter. Não guardar dados de pagamento além do necessário e permitido pelo modelo escolhido.

## 9. Contratos e integrações

Todos os contratos devem carregar identidade de correlação e tenant quando a operação for tenant-scoped; validar a origem e a autorização desse contexto. Eventos devem evitar conteúdo sensível desnecessário, ter schema versionado e consumidor idempotente. Integrações externas devem ter timeout, limites e tratamento de indisponibilidade.

Webhooks devem ser autenticados, assinados ou validados por mecanismo equivalente, deduplicados e protegidos contra replay. Segredos e configurações de uma integração pertencem ao tenant ou à plataforma conforme ownership documentado.

## 10. Dados, privacidade e retenção

- Classificar dados por sensibilidade, finalidade e tenant; coletar o mínimo necessário.
- Definir acesso, retenção, exportação, correção e exclusão conforme contrato e legislação aplicável.
- Evitar dados de saúde, conteúdo jurídico e identificadores em logs, métricas, nomes de recursos e mensagens de erro.
- Aplicar criptografia e gestão de chaves segundo risco, requisitos e opções contratadas.
- Definir se backups são compartilhados ou segregados e como atender restauração e exclusão por tenant.
- Avaliar residência e transferência internacional de dados com especialistas responsáveis.

Este complemento não substitui análise jurídica, avaliação de impacto ou controles exigidos por reguladores e contratos.

## 11. Operação e confiabilidade

Definir SLI/SLO por produto e, quando necessário, por plano ou capacidade contratada. Medir disponibilidade percebida pelo cliente, latência, erros, backlog operacional, idade da mensagem mais antiga e cumprimento de SLA OaaS. Não publicar SLO sem definir janela, exclusões e fonte de medição.

Projetar quotas e limites para proteger o serviço contra abuso e noisy neighbors sem afetar outros tenants. Definir alertas, resposta a incidentes, comunicação ao cliente e procedimento de recuperação. Testar restauração de dados e manter a associação tenant-dado durante restore, replay e migrações.

## 12. Testes e critérios de aceite

Antes de produção, demonstrar:

- usuário não acessa tenant sem membership ativa e permissão explícita;
- tentativas de acessar recurso de outro tenant falham em API, busca, arquivos, exports, filas e jobs;
- suspensão, mudança de plano e encerramento seguem estados e efeitos documentados;
- reentrega de evento ou retry não duplica efeito de negócio nem cobrança;
- ações de suporte são limitadas, expiram e deixam auditoria;
- backup, restauração e exportação preservam fronteiras de tenant;
- quotas e falhas de um tenant não comprometem desproporcionalmente os demais;
- workflows OaaS retomam após falha e mantêm autoria, estado e evidências;
- dados sensíveis não aparecem em telemetria de rotina.

Aplique os testes relevantes ao risco e ao escopo da mudança; testes de isolamento entre tenants são obrigatórios para toda superfície que processe dados multi-tenant.

## 13. ADRs específicos

Criar ou atualizar ADR para decisões sobre definição de tenant, isolamento de dados, controle e propagação de contexto, suporte cross-tenant, residência, retenção, planos de isolamento, billing por uso, SLOs contratados e execução de workflows OaaS.

## 14. Complementos de domínio

Este complemento é comum às seguintes especificações, que não devem ser fundidas em um modelo de negócio único:

- [Hospitais veterinários e clínicas veterinárias](02_COMPLEMENTO_VETERINARIA.md)
- [Clínicas odontológicas](03_COMPLEMENTO_ODONTOLOGIA.md)
- [Consultórios psicológicos](04_COMPLEMENTO_PSICOLOGIA.md)
- [Escritórios de advocacia](05_COMPLEMENTO_ADVOCACIA.md)
