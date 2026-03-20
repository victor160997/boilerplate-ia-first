# Prompt: Implementação de Feature (TDD)

<!--
  PROPÓSITO: Prompt reutilizável para solicitar a um agente de IA que
  implemente uma nova feature seguindo o fluxo TDD deste projeto.

  COMO USAR:
  1. Copie o bloco "Prompt" abaixo
  2. Substitua os campos em [COLCHETES]
  3. Cole no Claude Code ou ferramenta de IA de sua escolha

  LOCALIZAÇÃO: docs/ai-context/prompts/
  Outros prompts reutilizáveis ficam nesta pasta.
-->

## Prompt

```
Implemente a seguinte feature seguindo TDD (Red → Green → Refactor):

## Feature
[DESCRIÇÃO CLARA DO QUE PRECISA SER IMPLEMENTADO]

## Regras de negócio relacionadas
[EX: RN-001, RN-010 — consulte docs/domain/business-rules.md]

## Critérios de aceite
- [ ] [CRITÉRIO 1]
- [ ] [CRITÉRIO 2]

## Fluxo obrigatório
1. Escreva os testes primeiro (arquivo *.test.ts)
2. Rode os testes e confirme que FALHAM
3. Implemente o código mínimo para fazê-los passar
4. Rode os testes e confirme que PASSAM
5. Refatore se necessário

## Restrições
- Consulte docs/ai-context/constraints.md antes de gerar código
- Toda query ao banco deve incluir tenantId
- Use erros tipados de src/domain/errors.ts
- Sem `any` no TypeScript

## Arquivos relacionados (se souber)
[EX: src/services/user.service.ts, src/repositories/user.repository.ts]
```

---

## Exemplo preenchido

```
Implemente a seguinte feature seguindo TDD (Red → Green → Refactor):

## Feature
Endpoint para convidar um novo usuário para o tenant via email.
O convite deve expirar em 48 horas e não pode ser reutilizado.

## Regras de negócio relacionadas
- RN-001: email único por tenant
- RN-003: convites expiram em 48h
- RN-010: verificar limite de usuários do plano antes de convidar

## Critérios de aceite
- [ ] POST /tenants/:tenantId/invitations cria um convite
- [ ] Retorna 400 se o email já está no tenant (RN-001)
- [ ] Retorna 402 se o plano não permite mais usuários (RN-010)
- [ ] O convite tem status PENDING e TTL de 48h
- [ ] Só ADMIN e OWNER podem convidar

## Fluxo obrigatório
1. Escreva os testes primeiro (arquivo *.test.ts)
2. Rode os testes e confirme que FALHAM
3. Implemente o código mínimo para fazê-los passar
4. Rode os testes e confirme que PASSAM
5. Refatore se necessário

## Restrições
- Consulte docs/ai-context/constraints.md antes de gerar código
- Toda query ao banco deve incluir tenantId
- Use erros tipados de src/domain/errors.ts
- Sem `any` no TypeScript

## Arquivos relacionados
- src/services/user.service.ts (padrão a seguir)
- src/repositories/user.repository.ts (padrão a seguir)
- src/domain/errors.ts (erros disponíveis)
```
