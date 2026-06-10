# Copilot Instructions — Backend Kotlin

Este projeto usa um fluxo leve de Spec Driven Development chamado SDD Lite.

Objetivo:
- melhorar qualidade com IA;
- reduzir ambiguidade;
- evitar excesso de tokens;
- manter mudanças pequenas, revisáveis e testáveis.

## Idioma

- Responder em português do Brasil.
- Ser objetivo, técnico e direto.
- Explicar decisões relevantes de forma curta.

## Fluxo SDD Lite

Para tarefa pequena:
- implementar diretamente;
- adicionar ou ajustar teste quando houver mudança de comportamento.

Para feature média:
- gerar Mini Spec;
- gerar plano técnico;
- implementar com base na spec e no plano.

Para demanda crítica:
- usar Discovery, Plan, Implement e Review.

## Controle de contexto

- Não analisar o projeto inteiro sem necessidade.
- Usar arquivos de referência indicados pelo usuário.
- Preferir exemplos reais do projeto a explicações genéricas.
- Não duplicar conteúdo de `docs/ai/project-brief.md` e `docs/ai/architecture-lite.md`.
- Não criar documentação extensa sem solicitação.

## Padrões Kotlin

- Usar Kotlin idiomático.
- Preferir `val`.
- Evitar `!!`.
- Evitar lógica de negócio em controller.
- Não criar abstrações prematuras.
- Não introduzir biblioteca nova sem justificar.
- Seguir o padrão existente antes de criar um novo.

## Arquitetura

Fluxo esperado:

Controller → UseCase/Service → Gateway/Client/Repository → Sistema externo/Banco

Responsabilidades:
- Controller: contrato HTTP e delegação.
- UseCase/Service: regra de negócio e orquestração.
- Gateway/Client: integração externa.
- Repository: persistência.
- Mapper/DTO: transformação entre camadas.

## Segurança

- Não logar CPF, CNPJ, token, senha, cartão ou payload sensível.
- Mascarar dados sensíveis quando necessário.
- Não remover validações existentes.
- Não expor detalhes técnicos em respostas de erro.

## Testes

Toda alteração de comportamento deve ter teste.

Prioridade:
1. teste unitário de regra de negócio;
2. teste de controller/contrato HTTP;
3. teste de client/gateway/repository quando aplicável.

## Antes de finalizar

Validar:
- código segue arquitetura existente;
- alteração está dentro do escopo;
- testes foram criados ou atualizados;
- erros foram tratados;
- logs não expõem dados sensíveis;
- resposta final resume arquivos alterados e pontos de atenção.
