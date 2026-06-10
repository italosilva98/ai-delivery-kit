---
name: sdd-plan
description: "Gera plano técnico enxuto para frontend Angular a partir de uma Mini Spec"
argument-hint: "Informe o caminho da Mini Spec e arquivos de referência"
---

# SDD Plan — Frontend Angular

Você é responsável por transformar uma Mini Spec em um plano técnico curto e executável.

## Entradas

Usar:
- Mini Spec da feature;
- `docs/ai/project-brief.md`, se necessário;
- `docs/ai/architecture-lite.md`;
- arquivos de referência indicados pelo usuário.

Não analisar o projeto inteiro sem necessidade.

## Regras

- Não implementar código.
- Não repetir a Mini Spec.
- Não criar arquitetura nova sem necessidade.
- Não inventar contrato de API.
- Se faltar informação crítica, listar pendência.
- Manter o plano objetivo.

## O plano deve definir

- tipo da mudança;
- arquivos a criar;
- arquivos a alterar;
- componentes envolvidos;
- fluxo técnico;
- rota, se aplicável;
- serviço/API, se aplicável;
- estado;
- validações;
- erros;
- acessibilidade;
- testes;
- riscos;
- tarefas.

## Saída esperada

Gerar conteúdo para:

- `specs/[feature]/02-plan.md`
- `specs/[feature]/03-tasks.md`

## Template de `02-plan.md`

```md
# Technical Plan: [feature]

## Resumo

[Abordagem em até 5 linhas.]

## Tipo da mudança

- [ ] Nova tela
- [ ] Novo componente
- [ ] Nova integração API
- [ ] Alteração de fluxo
- [ ] Formulário
- [ ] Estado/facade/store
- [ ] Bugfix
- [ ] Refactor
- [ ] Acessibilidade
- [ ] Segurança

## Arquivos de referência

- `path/file.ts` — [motivo]

## Arquivos a criar

- `path/file.ts` — [responsabilidade]

## Arquivos a alterar

- `path/file.ts` — [motivo]

## Fluxo técnico

1. Page/container inicia carregamento ou recebe ação do usuário.
2. Facade/service orquestra estado.
3. API service executa chamada, se aplicável.
4. Mapper transforma DTO em model/view model.
5. Componentes apresentam os dados.
6. Estados loading, erro, vazio e sucesso são tratados.
7. Eventos do usuário atualizam estado ou navegação.

## Rota/navegação

- Rota:
- Guards/resolvers, se aplicável:
- Query params, se aplicável:

## API/contrato, se aplicável

```http
[METHOD] /endpoint
```

Response:

```json
{}
```

## Estado da tela

- Inicial:
- Loading:
- Sucesso:
- Erro:
- Vazio:

## Validações

- ...

## Tratamento de erros

- ...

## Acessibilidade e UX

- ...

## Testes necessários

- Component:
- Service/facade/store:
- API service:
- Pipe/directive/guard:
- E2E, se aplicável:

## Riscos

- ...

## Pendências

- Nenhuma.
```

## Template de `03-tasks.md`

```md
# Implementation Tasks: [feature]

## Estrutura

- [ ] Criar/alterar rota, se aplicável.
- [ ] Criar/alterar page/container.
- [ ] Criar/alterar componentes de apresentação.
- [ ] Criar/alterar models/DTOs/mappers.

## Estado e integração

- [ ] Criar/alterar facade/service/store.
- [ ] Criar/alterar API service.
- [ ] Tratar loading, sucesso, erro e vazio.
- [ ] Tratar navegação/query params, se aplicável.

## UI/UX

- [ ] Seguir Design System.
- [ ] Implementar validações visuais.
- [ ] Garantir mensagens de erro claras.
- [ ] Considerar responsividade.
- [ ] Considerar acessibilidade básica.

## Segurança

- [ ] Remover `console.log`.
- [ ] Não expor dados sensíveis.
- [ ] Sanitizar conteúdo dinâmico, se aplicável.

## Testes

- [ ] Testar renderização.
- [ ] Testar loading.
- [ ] Testar sucesso.
- [ ] Testar erro.
- [ ] Testar vazio.
- [ ] Testar eventos do usuário.

## Revisão

- [ ] Comparar com Mini Spec.
- [ ] Comparar com Technical Plan.
- [ ] Verificar alterações fora do escopo.
```
