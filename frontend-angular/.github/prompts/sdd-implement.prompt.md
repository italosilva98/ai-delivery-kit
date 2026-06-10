---
name: sdd-implement
description: "Implementa frontend Angular seguindo Mini Spec, plano e tasks"
argument-hint: "Informe spec, plan, tasks e arquivos de referência"
---

# SDD Implement — Frontend Angular

Você é responsável por implementar a feature seguindo Mini Spec, Technical Plan e Tasks.

## Entradas

Usar:
- `01-definition.md`;
- `02-plan.md`;
- `03-tasks.md`;
- `docs/ai/architecture-lite.md`;
- arquivos reais de referência indicados pelo usuário.

Não analisar o projeto inteiro sem necessidade.

## Regras

- Implementar apenas o escopo definido.
- Não alterar arquivos fora do plano sem justificar.
- Não criar arquitetura nova sem necessidade.
- Não introduzir biblioteca nova sem justificar.
- Não chamar API diretamente do componente se o padrão usa service/facade.
- Não colocar lógica complexa no template.
- Não usar `any` sem justificativa.
- Não deixar `console.log`.
- Não usar `innerHTML` sem sanitização e justificativa.
- Não expor dados sensíveis.
- Criar ou atualizar testes quando houver mudança de comportamento.

## Ordem sugerida

1. Models/DTOs/mappers.
2. API service, se aplicável.
3. Facade/service/store.
4. Page/container.
5. Presentational components.
6. Rota/guard/resolver, se necessário.
7. Template e estilos.
8. Testes.
9. Revisão final.

## Padrões

- Usar Angular idiomático.
- Preservar estilo existente.
- Preferir tipagem forte.
- Usar nomes expressivos.
- Manter componentes pequenos.
- Priorizar menor alteração possível.
- Tratar loading, erro, vazio e sucesso.
- Seguir Design System.

## Testes

Criar ou atualizar testes para:
- renderização;
- loading;
- sucesso;
- erro;
- vazio;
- validação;
- eventos do usuário;
- chamada de service/facade/API.

## Divergências

Se a Mini Spec e o Plan divergirem:
- parar a parte conflitante;
- apontar o conflito;
- sugerir ajuste.

Se o código existente divergir da arquitetura documentada:
- seguir o padrão real do código;
- informar no resumo.

## Resposta final

Responder neste formato:

```md
## Implementação concluída

### Arquivos criados

- `path/file.ts` — ...

### Arquivos alterados

- `path/file.ts` — ...

### O que foi implementado

- ...

### Testes

- `path/file.spec.ts` — ...

### Validações

- [ ] Comportamento conforme spec
- [ ] Estado loading/erro/vazio tratado
- [ ] API encapsulada em service/facade
- [ ] Template sem lógica complexa
- [ ] Sem `console.log`
- [ ] Sem dado sensível exposto
- [ ] Testes adicionados/ajustados

### Comandos sugeridos

```bash
npm test
npm run lint
npm run build
```

### Pontos de atenção

- Nenhum.

### Pendências

- Nenhuma.
```
