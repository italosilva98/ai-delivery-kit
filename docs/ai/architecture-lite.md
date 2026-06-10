# Architecture Lite — Backend Kotlin

## Objetivo

Guia curto de arquitetura para orientar implementação com IA.

A IA deve priorizar o padrão real do projeto. Este documento serve como referência quando houver dúvida.

## Fluxo padrão

Controller → UseCase/Service → Gateway/Client/Repository → Sistema externo/Banco

## Estrutura sugerida

```text
src/main/kotlin/com/company/project/
  presentation/
    controller/
    dto/
    error/
  application/
    usecase/
    service/
  domain/
    model/
    exception/
  infrastructure/
    client/
    repository/
    mapper/
    config/
```

Se o projeto usa estrutura mais simples, seguir a estrutura existente.

## Controller

Responsável por:
- expor endpoint REST;
- receber request;
- validar entrada básica;
- chamar use case/service;
- retornar response DTO e status HTTP.

Não deve:
- conter regra de negócio;
- chamar repository diretamente;
- chamar client externo diretamente;
- ter lógica complexa.

## UseCase/Service

Responsável por:
- aplicar regra de negócio;
- orquestrar fluxo;
- chamar gateways/repositories;
- tratar decisões de domínio;
- retornar modelo interno ou resultado de aplicação.

Não deve:
- retornar `ResponseEntity`;
- depender de DTO de controller;
- expor detalhes de integração externa.

## Domain

Responsável por:
- modelos de domínio;
- objetos de valor;
- enums de negócio;
- exceções de domínio;
- regras puras quando aplicável.

Não deve:
- depender de Spring;
- depender de HTTP;
- depender de banco;
- depender de DTO externo.

## Gateway/Client

Responsável por:
- integrar com sistemas externos;
- encapsular detalhes técnicos;
- tratar 4xx, 5xx, timeout e payload inválido;
- mapear resposta externa para modelo interno.

Não deve:
- espalhar DTO externo pelo projeto;
- conter regra de negócio de alto nível;
- retornar erro técnico cru para o consumidor.

## Repository

Responsável por:
- persistência;
- consultas;
- transações quando aplicável.

Não deve:
- expor entity como contrato HTTP;
- conter regra de negócio complexa.

## DTOs e Mappers

- Request DTO representa entrada HTTP.
- Response DTO representa saída HTTP.
- Domain Model representa negócio.
- External DTO representa integração externa.
- Mapper transforma entre camadas.

Evitar:
- usar entity como response;
- usar DTO externo como domínio;
- duplicar transformação em vários lugares.

## Erros

Usar categorias claras:
- validação;
- negócio;
- não encontrado;
- integração;
- autorização;
- inesperado.

Mapeamento comum:
- entrada inválida: 400;
- não encontrado: 404;
- regra de negócio: 400 ou 422, conforme padrão;
- não autenticado: 401;
- não autorizado: 403;
- integração indisponível: 502/503;
- erro inesperado: 500.

## Logs

- Usar logger, nunca `println`.
- Logar falhas relevantes.
- Não logar dados sensíveis.
- Preservar correlation-id/tracing quando existir.

## Testes

Prioridade:
1. UseCase/Service;
2. Controller;
3. Gateway/Client;
4. Repository, se houver query relevante.

Cobrir:
- sucesso;
- erro de validação;
- erro de negócio;
- erro de integração;
- resposta vazia ou inconsistente.

## Checklist rápido

Antes de implementar:
- contrato está claro?
- regra de negócio está clara?
- arquivos de referência foram indicados?
- erro esperado está definido?
- teste esperado está definido?

Depois de implementar:
- controller está fino?
- regra ficou no use case/service?
- testes foram adicionados?
- logs estão seguros?
- alteração ficou dentro do escopo?
