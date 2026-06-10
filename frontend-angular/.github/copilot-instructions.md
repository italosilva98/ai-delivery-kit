# Copilot Instructions — Frontend Angular

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

## Padrões Angular

- Usar Angular moderno e idiomático.
- Preferir componentes standalone quando o projeto já adotar esse padrão.
- Preferir `OnPush` quando fizer sentido.
- Evitar lógica complexa no template.
- Evitar componentes grandes com muitas responsabilidades.
- Separar container component de presentational component quando a tela tiver regra relevante.
- Não introduzir biblioteca nova sem justificar.
- Seguir o padrão existente antes de criar um novo.

## Arquitetura

Fluxo esperado:

Component/Page → Facade/Store/Service → API Service/Repository → Backend

Responsabilidades:
- Component/Page: interação visual, bindings e eventos.
- Facade/Store/Service: estado, orquestração e regras de tela.
- API Service: comunicação HTTP.
- Model/DTO/Mapper: contratos e transformação de dados.
- Shared/UI: componentes reutilizáveis sem regra de negócio específica.

## Estado

- Usar o padrão existente do projeto.
- Para estado local simples, preferir signals, inputs, outputs ou propriedades simples.
- Para estado compartilhado, usar facade/store/service conforme padrão existente.
- Não espalhar estado global sem necessidade.
- Evitar subscriptions manuais sem controle de ciclo de vida.

## HTTP e erros

- Centralizar chamadas HTTP em services próprios.
- Não chamar HttpClient diretamente de componentes, salvo padrão explícito do projeto.
- Tratar loading, erro e estado vazio.
- Não expor mensagens técnicas ao usuário.
- Não duplicar tratamento de erro já existente.

## Segurança

- Não expor token, segredo ou dado sensível no console.
- Não deixar `console.log` em código final.
- Não usar `innerHTML` sem sanitização e justificativa.
- Evitar armazenar dados sensíveis em localStorage/sessionStorage.
- Validar e mascarar dados sensíveis quando necessário.

## Testes

Toda alteração de comportamento deve ter teste.

Prioridade:
1. teste de component/container;
2. teste de service/facade/store;
3. teste de pipe/directive/guard/interceptor quando aplicável;
4. teste E2E apenas quando o fluxo justificar.

## Antes de finalizar

Validar:
- código segue arquitetura existente;
- alteração está dentro do escopo;
- testes foram criados ou atualizados;
- loading, erro e estado vazio foram considerados;
- acessibilidade básica foi considerada;
- não há logs ou dados sensíveis expostos;
- resposta final resume arquivos alterados e pontos de atenção.
