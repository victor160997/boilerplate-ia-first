# Guia de Contribuição

<!--
  Leitura obrigatória antes de abrir qualquer PR.
  Num projeto IA-first, contribuir inclui alimentar o contexto para a IA.
  Quando você muda o código, pergunte: "a documentação de IA ainda está correta?"
-->

## Fluxo de trabalho (TDD obrigatório)

Este projeto adota **Test-Driven Development** como prática central.
Nenhuma linha de implementação deve ser escrita sem um teste falhando antes.

```
1. Escolha/crie uma issue
2. Crie branch: feat/<id>-descricao ou fix/<id>-descricao
3. RED   — escreva o teste que descreve o comportamento esperado
4.          Confirme que ele FALHA (npm test -- --run)
5. GREEN — implemente o mínimo necessário para o teste passar
6.          Confirme que ele PASSA
7. REFACTOR — melhore o código sem quebrar os testes
8. Atualize documentação afetada (incluindo docs/ai-context/ se necessário)
9. Abra PR com template preenchido
```

> **Regra para agentes de IA:** ao receber uma tarefa de implementação,
> sempre escreva os testes primeiro, rode-os para confirmar a falha,
> depois implemente. Nunca pule esta etapa.

## Ciclo RED → GREEN → REFACTOR

```
         ┌─────────────┐
         │  Escrever   │
         │  o teste    │
         └──────┬──────┘
                │
                ▼
         ┌─────────────┐
         │  Rodar e    │◄─── DEVE falhar aqui
         │  ver falhar │
         └──────┬──────┘
                │
                ▼
         ┌─────────────┐
         │  Implementar│
         │  o mínimo   │
         └──────┬──────┘
                │
                ▼
         ┌─────────────┐
         │  Rodar e    │◄─── DEVE passar aqui
         │  ver passar │
         └──────┬──────┘
                │
                ▼
         ┌─────────────┐
         │  Refatorar  │
         │  (opcional) │
         └─────────────┘
```

## Checklist antes do PR

- [ ] Teste escrito ANTES da implementação (TDD)
- [ ] Todos os testes passando (`npm test`)
- [ ] Lint sem erros (`npm run lint`)
- [ ] Tipos sem erros (`npm run typecheck`)
- [ ] Documentação atualizada se a mudança afeta comportamento externo
- [ ] `CLAUDE.md` atualizado se a arquitetura mudou
- [ ] `docs/ai-context/codebase-map.md` atualizado se novos módulos foram criados

## Decisões arquiteturais

Toda decisão relevante de arquitetura deve ter um ADR (Architecture Decision Record)
em `docs/architecture/decisions/`. Use o template de `adr-001-auth-strategy.md`.

Não implemente mudanças arquiteturais significativas sem um ADR aprovado.

## Contexto IA-first

Neste projeto, a documentação é um cidadão de primeira classe — ela alimenta
as ferramentas de IA que usamos no desenvolvimento. Isso significa:

- Escreva comentários de código em inglês (mais compatível com LLMs)
- Mantenha o glossário (`docs/domain/glossary.md`) atualizado com novos termos
- Quando criar um padrão novo, documente-o em `docs/guides/development.md`
