# Project Brief

## Objetivo do projeto

Backend Kotlin com Spring Boot responsável por expor APIs REST, aplicar regras de negócio, orquestrar integrações externas e retornar respostas padronizadas para consumidores internos ou externos.

## Tipo de aplicação

API REST / BFF / Microsserviço.

Ajustar conforme o projeto real.

## Stack

- Linguagem: Kotlin
- Framework: Spring Boot
- Build: Gradle
- API: REST/JSON
- Validação: Jakarta Validation
- Testes: JUnit 5 + MockK/Mockito
- Logs: SLF4J
- Observabilidade: Micrometer/Datadog/OpenTelemetry, se aplicável
- Persistência: JPA/JDBC/NoSQL, se aplicável
- Integrações: WebClient/Feign/RestClient/client interno

## Responsabilidades

Este backend deve:
- expor contratos HTTP claros;
- validar entradas;
- aplicar regras de negócio;
- integrar com sistemas externos;
- tratar erros de forma padronizada;
- registrar logs úteis;
- proteger dados sensíveis;
- manter testes automatizados.

## Fora de escopo

Este backend não deve:
- conter regra de UI;
- expor entity de banco como response;
- vazar contrato externo para consumidores;
- armazenar credenciais em código;
- criar abstrações sem necessidade.

## Regras críticas

- Não logar dados sensíveis sem máscara.
- Não alterar contrato público sem explicitar impacto.
- Não expor detalhes técnicos de integrações externas.
- Não colocar regra de negócio em controller.
- Não remover validações existentes sem justificativa.

## Padrão de resposta de erro

Seguir o padrão existente do projeto.

Quando não houver padrão, usar estrutura semelhante:

```json
{
  "code": "ERROR_CODE",
  "message": "Mensagem segura para o consumidor.",
  "correlationId": "abc-123"
}
```

## Como a IA deve usar este arquivo

Usar apenas para entender o contexto geral.

Prioridade de contexto:
1. Mini Spec da feature;
2. plano técnico;
3. `architecture-lite.md`;
4. exemplos reais do código;
5. este documento.
