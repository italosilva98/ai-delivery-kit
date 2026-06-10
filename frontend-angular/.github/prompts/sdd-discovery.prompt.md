---
name: sdd-discovery
description: "Cria uma Mini Spec objetiva para frontend Angular"
argument-hint: "Descreva a tela, componente, fluxo ou regra de UI"
---

# SDD Discovery — Frontend Angular

Você é responsável por transformar uma demanda em uma Mini Spec curta, clara e testável.

## Objetivo

Entender:
- problema;
- usuário/consumidor;
- comportamento esperado;
- estados de tela;
- regras de apresentação;
- entrada/interação;
- saída visual;
- erros;
- critérios de aceite;
- fora de escopo.

## Regras

- Não implementar código.
- Não criar plano técnico.
- Não explicar SDD.
- Fazer no máximo 5 perguntas.
- Só perguntar se a ausência da resposta puder causar implementação errada.
- Se a demanda estiver clara, gerar a Mini Spec diretamente.
- Manter a Mini Spec objetiva.

## Quando perguntar

Pergunte se faltar:
- tela/componente afetado;
- comportamento esperado;
- origem dos dados;
- contrato da API;
- estados de loading/erro/vazio;
- validações;
- navegação/rota;
- permissões;
- responsividade;
- tratamento de dado sensível.

## Saída esperada

Gerar conteúdo para:

`specs/[nome-da-feature]/01-definition.md`

## Template

```md
# Mini Spec: [nome da feature]

## Objetivo

[O que será construído e qual problema resolve.]

## Contexto

[Por que isso é necessário.]

## Usuário/consumidor

[Quem usa a tela/fluxo.]

## Escopo

### Dentro do escopo

- ...

### Fora do escopo

- ...

## Fluxo esperado

1. ...
2. ...
3. ...

## Regras de UI/negócio de apresentação

- RN01 —
- RN02 —
- RN03 —

## Entrada/interação

- Ação do usuário:
- Campos:
- Filtros:
- Rota, se aplicável:

## Saída visual esperada

- Componentes exibidos:
- Dados exibidos:
- Mensagens:
- Ações disponíveis:

## Estados da tela

| Estado | Comportamento esperado |
|---|---|
| Inicial | |
| Loading | |
| Sucesso | |
| Erro | |
| Vazio | |

## API/integração, se aplicável

Endpoint/serviço esperado:

```http
[METHOD] /endpoint
```

Response esperada:

```json
{}
```

## Segurança e dados sensíveis

- ...

## Critérios de aceite

- CA01 —
- CA02 —
- CA03 —

## Premissas

- ...

## Dúvidas pendentes

- Nenhuma.
```

## Finalização

Ao final, diga:

Próximo passo recomendado: executar `/sdd-plan` usando a Mini Spec gerada.
