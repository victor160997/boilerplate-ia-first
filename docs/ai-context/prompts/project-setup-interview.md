# Prompt: Entrevista de Setup do Projeto

<!--
  QUANDO USAR: Logo após clonar este boilerplate para um novo projeto.

  COMO USAR:
  1. Abra o Claude Code na raiz do projeto
  2. Copie o bloco "Prompt" abaixo e cole no chat
  3. Responda as perguntas da IA honestamente
  4. Ao final, a IA atualizará toda a documentação automaticamente

  RESULTADO ESPERADO: Ao final da entrevista, os arquivos de documentação
  estarão preenchidos com as informações reais do seu projeto e as seções
  irrelevantes serão removidas.
-->

## Prompt

---

Você vai me ajudar a configurar este boilerplate para um novo projeto.
Seu trabalho é me **entrevistar** — faça perguntas, uma seção por vez,
espere minhas respostas antes de continuar. Não gere documentação ainda.

## Fase 1 — Identidade do projeto

Comece perguntando:
1. Qual é o nome do projeto e o que ele faz? (2-3 linhas)
2. Quem são os usuários? (ex: empresas B2B, consumidores finais, desenvolvedores)
3. Qual é o estágio atual? (ideia, MVP, produto em produção, refactor de sistema existente)
4. Existe algum prazo ou restrição importante que eu deva saber?

Após minhas respostas, confirme seu entendimento antes de continuar.

---

## Fase 2 — Tecnologias e stack

Pergunte:
1. Você já tem alguma tecnologia definida (linguagem, framework, banco)?
   Se não, apresente **3 opções de stack** adequadas ao que descreveu,
   com prós e contras de cada uma. Peça que eu escolha.
2. O projeto precisa de autenticação? Se sim:
   - Usuário/senha próprio, OAuth (Google, GitHub), magic link, ou outro?
   - Apresente **3 opções de implementação** com trade-offs.
3. Precisa de banco de dados? Qual tipo de dados você vai armazenar?
   - Se não souber, apresente **3 opções** com justificativa.
4. Vai ter múltiplos tenants/organizações ou é single-tenant?
5. Vai ter alguma integração com IA (LLM, embeddings, etc.)?
6. Precisa de filas, jobs agendados, websockets ou é só request/response?

Para cada ponto sem decisão tomada, sempre ofereça **pelo menos 3 opções**
com sua recomendação clara e o motivo.

---

## Fase 3 — Complexidade e escopo

Pergunte:
1. Qual é o nível de complexidade esperado?
   - Simples (CRUD, poucos endpoints, time solo)
   - Médio (regras de negócio, time pequeno, alguns serviços externos)
   - Alto (múltiplos serviços, times diferentes, alta carga)
2. Vai ter CI/CD e deploy automatizado? Em qual plataforma?
   Se não sabe, apresente **3 opções** de plataforma de deploy adequadas.
3. Vai precisar de monitoramento, logs centralizados, alertas?
4. Qual o tamanho do time? (solo, dupla, time pequeno 3-5, maior)
5. Existe documentação de API para consumidores externos ou é só uso interno?

---

## Fase 4 — Regras de negócio e edge cases

Pergunte:
1. Quais são as 3-5 regras de negócio mais importantes do sistema?
   (ex: "um usuário só pode ter um plano ativo", "pedidos não podem ser
   cancelados após 24h")
2. Quais são os principais fluxos de usuário? (ex: cadastro → onboarding →
   uso principal)
3. Quais são os edge cases que mais te preocupam?
   (ex: concorrência, dados inconsistentes, falhas de pagamento)
4. O que NÃO deve acontecer nunca? (ex: usuário ver dados de outro,
   cobrança duplicada)
5. Existe alguma regulamentação ou compliance relevante? (LGPD, PCI, HIPAA)

---

## Fase 5 — Confirmação e gaps

Antes de atualizar a documentação:

1. Resuma tudo que entendeu do projeto em formato estruturado
2. Liste explicitamente o que **NÃO** vai ser usado deste boilerplate:
   - Ex: "Sem multi-tenancy", "Sem CI/CD", "Sem integração com IA", etc.
3. Liste as suas **3 principais recomendações** que o usuário ainda não mencionou
   mas que provavelmente vai precisar (ex: rate limiting, soft delete, auditoria)
4. Pergunte: "Está correto? Posso atualizar a documentação agora?"

Só avance para a próxima fase com minha confirmação explícita.

---

## Fase 6 — Atualização da documentação

Após minha confirmação, atualize os seguintes arquivos com as informações reais:

### Arquivos a preencher com dados do projeto:
- `README.md` — nome, descrição, stack real, comandos reais
- `CLAUDE.md` — nome, domínio, stack, regras reais do projeto
- `docs/architecture/overview.md` — diagrama e stack reais
- `docs/domain/business-rules.md` — regras levantadas na entrevista
- `docs/domain/glossary.md` — termos do domínio mencionados
- `docs/api/endpoints.md` — endpoints principais (mesmo que sejam rascunhos)
- `docs/api/contracts.md` — entidades principais do domínio

### Arquivos a simplificar ou remover se não forem usados:
- `docs/architecture/decisions/adr-001-auth-strategy.md` → reescrever com a decisão real de auth, ou remover se não houver auth
- `docs/architecture/decisions/adr-002-multi-tenant.md` → reescrever com a decisão real, ou remover se for single-tenant
- `docs/infra/deployment.md` → simplificar para a plataforma escolhida, ou remover se não houver deploy definido
- `docs/infra/environments.md` → ajustar para os ambientes reais, ou remover se não se aplicar
- `docs/ai-context/system-prompt.md` → preencher com o domínio real, ou remover se não houver agente de IA no produto
- `CONTRIBUTING.md` → ajustar o fluxo para o tamanho de time informado

### Regras para a atualização:
- Substitua todos os `[COLCHETES]` e `{{variáveis}}` com valores reais
- Remova seções de exemplo que não se aplicam ao projeto
- Mantenha os comentários HTML `<!-- ... -->` explicativos nos arquivos
- Se um arquivo inteiro não se aplica, delete-o e remova a referência do README
- Após atualizar, liste todos os arquivos modificados e os que foram removidos

---