---
name: sdd-plan
description: "Gera plano técnico enxuto para backend Kotlin a partir de uma Mini Spec"
argument-hint: "Informe o caminho da Mini Spec e arquivos de referência"
---

# SDD Plan — Backend Kotlin

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
- Não inventar contrato.
- Se faltar informação crítica, listar pendência.
- Manter o plano objetivo.

## O plano deve definir

- tipo da mudança;
- arquivos a criar;
- arquivos a alterar;
- fluxo técnico;
- contrato HTTP;
- validações;
- erros;
- logs/segurança;
- testes;
- riscos;
- tarefas.

## Saída esperada

Gerar conteúdo para:

- `specs/[feature]/02-plan.md`
- `specs/[feature]/03-tasks.md`

## Template de `02-plan.md`


# Technical Plan: [feature]

## Resumo

[Abordagem em até 5 linhas.]

## Tipo da mudança

- [ ] Nova rota
- [ ] Nova regra
- [ ] Nova integração
- [ ] Persistência
- [ ] Bugfix
- [ ] Refactor
- [ ] Segurança
- [ ] Observabilidade

## Arquivos de referência

- `path/File.kt` — [motivo]

## Arquivos a criar

- `path/File.kt` — [responsabilidade]

## Arquivos a alterar

- `path/File.kt` — [motivo]

## Fluxo técnico

1. Controller recebe a requisição.
2. Controller valida entrada básica.
3. UseCase/Service aplica regra.
4. Gateway/Client/Repository executa integração ou persistência.
5. Mapper transforma dados.
6. Erros seguem padrão global.
7. Controller retorna response.

## Contrato HTTP
[METHOD] /v1/[resource]


Response:

{}


Erros:

| Cenário | Status | Código |
|---|---:|---|

## Validações

- ...

## Tratamento de erros

- ...

## Logs e segurança

- ...

## Testes necessários

- Unitários:
- Controller:
- Client/Gateway:
- Repository:

## Riscos

- ...

## Pendências

- Nenhuma.


## Template de `03-tasks.md`


# Implementation Tasks: [feature]

## Contrato

- [ ] Criar/alterar DTOs.
- [ ] Criar/alterar controller.
- [ ] Validar status HTTP.

## Regra de negócio

- [ ] Criar/alterar use case/service.
- [ ] Implementar regras RN relevantes.
- [ ] Criar exceptions, se necessário.

## Integração/persistência

- [ ] Criar/alterar client/gateway/repository.
- [ ] Criar mappers necessários.
- [ ] Tratar erros técnicos.

## Segurança/logs

- [ ] Garantir logs seguros.
- [ ] Mascarar dados sensíveis, se aplicável.

## Testes

- [ ] Testar sucesso.
- [ ] Testar validação.
- [ ] Testar regra de negócio.
- [ ] Testar erro de integração, se aplicável.

## Revisão

- [ ] Comparar com Mini Spec.
- [ ] Comparar com Technical Plan.
- [ ] Verificar alterações fora do escopo.


## Finalização

Ao final, diga:

Próximo passo recomendado: executar `/sdd-implement` usando o plan e tasks geradas.
