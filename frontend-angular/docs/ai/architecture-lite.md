# Architecture Lite — Frontend Angular

## Objetivo

Guia curto de arquitetura para orientar implementação com IA em projetos Angular.

A IA deve priorizar o padrão real do projeto. Este documento serve como referência quando houver dúvida.

## Fluxo padrão

Page/Container → Facade/Service/Store → API Service → Backend

## Estrutura sugerida

```text
src/app/
  core/
    interceptors/
    guards/
    services/
  shared/
    components/
    directives/
    pipes/
    utils/
  features/
    feature-name/
      pages/
      components/
      services/
      models/
      mappers/
      resolvers/
      routes.ts
```

Se o projeto usa outra estrutura, seguir a estrutura existente.

## Page/Container

Responsável por:
- montar a tela;
- coordenar componentes filhos;
- chamar facade/service;
- tratar eventos principais;
- lidar com loading, erro, vazio e sucesso.

Não deve:
- conter lógica de API diretamente;
- ter template complexo;
- concentrar muitas regras;
- duplicar lógica de outros containers.

## Presentational Component

Responsável por:
- receber dados via input;
- emitir eventos via output;
- renderizar UI;
- ser reutilizável e previsível.

Não deve:
- chamar API;
- conhecer regra de negócio externa;
- manipular estado global;
- conter efeitos colaterais complexos.

## Facade/Service/Store

Responsável por:
- orquestrar estado da tela;
- chamar API service;
- transformar dados para a UI;
- centralizar loading, erro e sucesso;
- expor dados consumíveis pelo componente.

Não deve:
- conter detalhes visuais específicos demais;
- duplicar chamadas HTTP em vários lugares;
- misturar responsabilidades de múltiplas features.

## API Service

Responsável por:
- encapsular HttpClient;
- definir endpoints;
- tipar request/response;
- tratar parâmetros;
- delegar erro para padrão existente.

Não deve:
- ser chamado diretamente por muitos componentes;
- retornar `any`;
- misturar contrato externo com modelo visual sem necessidade.

## Models, DTOs e Mappers

- DTO representa contrato da API.
- Model/ViewModel representa dado usado pela tela.
- Mapper transforma DTO em model.
- Evitar `any`.
- Evitar usar response da API diretamente em toda a UI quando houver transformação relevante.

## Estado

Usar o padrão existente:
- Signals para estado local simples ou moderno;
- RxJS quando o projeto já usar observables;
- Facade/store para estado compartilhado;
- Reactive Forms para formulários complexos.

Evitar:
- subscription manual sem unsubscribe/takeUntilDestroyed;
- estado duplicado;
- lógica espalhada entre template e component;
- efeitos colaterais difíceis de testar.

## Templates

- Evitar expressões complexas.
- Evitar múltiplas chamadas de função no template.
- Usar pipes apenas para transformação visual simples.
- Preferir componentes menores quando o template crescer demais.
- Considerar acessibilidade básica.

## Formulários

- Usar Reactive Forms quando houver validação relevante.
- Validar campos obrigatórios, formato e tamanho.
- Exibir mensagens de erro claras.
- Não confiar apenas no frontend para validações críticas.

## HTTP e erros

- Tratar loading.
- Tratar erro.
- Tratar estado vazio.
- Não exibir erro técnico ao usuário.
- Não duplicar tratamento já existente em interceptor/facade.
- Preservar correlation-id/tracing quando existir.

## Segurança

- Não deixar `console.log`.
- Não usar `innerHTML` sem sanitização.
- Não armazenar dados sensíveis sem justificativa.
- Não expor token ou dados pessoais.
- Aplicar máscara para CPF/CNPJ/telefone quando necessário.

## Testes

Prioridade:
1. Component/container;
2. Facade/service/store;
3. API service;
4. Pipe/directive/guard/interceptor;
5. E2E quando fluxo crítico justificar.

Cobrir:
- renderização inicial;
- loading;
- sucesso;
- erro;
- vazio;
- validação;
- eventos do usuário.

## Checklist rápido

Antes de implementar:
- comportamento da tela está claro?
- contrato da API está claro?
- estados loading/erro/vazio estão definidos?
- arquivos de referência foram indicados?
- testes esperados estão definidos?

Depois de implementar:
- componente não ficou grande demais?
- API está encapsulada?
- testes foram adicionados?
- não há `console.log`?
- UI segue Design System?
- alteração ficou dentro do escopo?
