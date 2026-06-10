# Project Brief

## Objetivo do projeto

Frontend Angular responsável por entregar uma interface web modular, responsiva e testável, consumindo APIs backend, aplicando regras de apresentação e oferecendo boa experiência ao usuário.

## Tipo de aplicação

SPA / Microfrontend / Portal interno / Frontend corporativo.

Ajustar conforme o projeto real.

## Stack

- Framework: Angular
- Linguagem: TypeScript
- UI: Angular Material / PrimeNG / Design System interno
- Estilo: SCSS/CSS
- Estado: Signals / RxJS / Service / Facade / Store, conforme padrão do projeto
- HTTP: HttpClient
- Formulários: Reactive Forms / Template-driven, conforme padrão
- Testes unitários: Jest/Karma/Jasmine, conforme projeto
- E2E: Cypress/Playwright, se aplicável
- Documentação UI: Storybook, se aplicável

## Responsabilidades

Este frontend deve:
- exibir telas e componentes reutilizáveis;
- consumir APIs de forma centralizada;
- tratar loading, erro, vazio e sucesso;
- aplicar regras de apresentação;
- validar formulários;
- preservar acessibilidade básica;
- manter testes automatizados;
- seguir o Design System do projeto.

## Fora de escopo

Este frontend não deve:
- conter regra de negócio crítica que pertence ao backend;
- duplicar validações sensíveis como única camada de segurança;
- acessar APIs diretamente de componentes sem padrão do projeto;
- armazenar token ou dado sensível sem justificativa;
- criar componentes genéricos sem necessidade;
- alterar contrato backend sem explicitar impacto.

## Regras críticas

- Não deixar `console.log` em código final.
- Não expor dados sensíveis em tela, logs ou storage.
- Não criar UI fora do Design System sem justificar.
- Não colocar lógica complexa no template.
- Não ignorar loading, erro e estado vazio.
- Não remover testes existentes sem justificativa.

## Padrão de experiência

Toda feature de tela deve considerar:
- estado inicial;
- loading;
- sucesso;
- erro;
- vazio;
- validação;
- acessibilidade básica;
- responsividade quando aplicável.

## Como a IA deve usar este arquivo

Usar apenas para entender o contexto geral.

Prioridade de contexto:
1. Mini Spec da feature;
2. plano técnico;
3. `architecture-lite.md`;
4. exemplos reais do código;
5. este documento.
