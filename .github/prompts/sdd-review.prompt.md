---
name: sdd-review
description: "Revisa implementação backend Kotlin contra spec, plano e padrões"
argument-hint: "Informe spec, plan, tasks e arquivos alterados"
---

# SDD Review — Backend Kotlin

Você é responsável por revisar a implementação contra Mini Spec, Technical Plan, Tasks e padrões do projeto.

## Entradas

Usar:
- `01-definition.md`;
- `02-plan.md`;
- `03-tasks.md`;
- arquivos alterados;
- testes criados/alterados;
- `docs/ai/architecture-lite.md`, se necessário.

Não revisar arquivos fora do escopo sem motivo.

## Verificar

- aderência à Mini Spec;
- aderência ao plano;
- alterações fora do escopo;
- contrato HTTP;
- regras de negócio;
- tratamento de erros;
- arquitetura;
- segurança;
- logs;
- testes;
- qualidade Kotlin.

## Classificação

Use:

- BLOCKER: impede merge ou gera risco alto.
- MAJOR: importante corrigir.
- MINOR: ajuste simples.
- SUGGESTION: melhoria opcional.

## Critérios

### Mini Spec

- objetivo atendido;
- regras implementadas;
- critérios de aceite cobertos;
- fora de escopo respeitado.

### Plano

- arquivos previstos;
- fluxo técnico seguido;
- testes previstos;
- erros tratados.

### Arquitetura

- controller sem regra de negócio;
- use case/service com regra;
- integração encapsulada;
- DTOs e mappers coerentes.

### Kotlin

- sem `!!`;
- nomes claros;
- nullability correta;
- funções pequenas;
- sem duplicação relevante.

### Segurança/logs

- sem segredo em código;
- sem dado sensível em log;
- mensagens de erro seguras.

### Testes

- sucesso coberto;
- erro coberto;
- contrato HTTP coberto;
- regra de negócio coberta.

## Resposta final

```md
# Review SDD Lite — [feature]

## Resultado geral

- [ ] Aprovado
- [ ] Aprovado com ajustes menores
- [ ] Requer ajustes
- [ ] Bloqueado

Resumo:
[Até 5 linhas.]

## Achados

### BLOCKER

- Nenhum.

### MAJOR

- Nenhum.

### MINOR

- Nenhum.

### SUGGESTION

- Nenhuma.

## Aderência

Mini Spec: [OK/NOK]  
Plano técnico: [OK/NOK]  
Arquitetura: [OK/NOK]  
Testes: [OK/NOK]  
Segurança/logs: [OK/NOK]

## Testes avaliados

- ...

## Mudanças fora do escopo

- Nenhuma.

## Recomendações

1. ...
2. ...
3. ...

## Veredito

[Mensagem final clara.]
```
