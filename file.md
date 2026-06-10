# SDD Lite Kit para Backend Kotlin

Estrutura recomendada:

```text
.github/
  copilot-instructions.md
  prompts/
    sdd-discovery.prompt.md
    sdd-plan.prompt.md
    sdd-implement.prompt.md
    sdd-review.prompt.md

docs/
  ai/
    project-brief.md
    architecture-lite.md

specs/
  .gitkeep
```

---

# Arquivo: `.github/copilot-instructions.md`

````md
# Copilot Instructions — Backend Kotlin SDD Lite

## Objetivo destas instruções

Este projeto usa um fluxo leve de Spec Driven Development chamado **SDD Lite**.

O objetivo é melhorar a qualidade das entregas com IA sem transformar cada tarefa em um processo pesado, caro em tokens ou burocrático.

Use estas instruções para manter consistência, qualidade, segurança e aderência aos padrões do projeto.

---

## Idioma e estilo de resposta

- Responda em português do Brasil.
- Seja técnico, objetivo e direto.
- Evite explicações longas quando a tarefa for simples.
- Quando houver decisão técnica relevante, explique brevemente o motivo.
- Ao implementar, prefira respostas com:
  - arquivos alterados;
  - resumo do que foi feito;
  - testes criados/alterados;
  - pontos de atenção.

---

## Princípios do SDD Lite

Para demandas pequenas e claras:
- Não gere spec completa.
- Implemente diretamente com teste e revisão.

Para demandas médias ou ambíguas:
- Gere ou atualize uma Mini Spec antes de planejar.
- Gere um plano técnico curto antes de implementar.
- Implemente apenas o que estiver no plano.

Para demandas críticas:
- Faça Discovery, Plan, Implement e Review.
- Trate segurança, contrato, logs, observabilidade e testes como obrigatórios.

Classificação sugerida:

```text
Pequena: bug local, ajuste simples, teste isolado, refactor pequeno.
Média: nova rota, nova regra de negócio, nova integração simples.
Crítica: segurança, contrato público, dado sensível, regra financeira, múltiplos sistemas.
````

---

## Controle de tokens e contexto

* Não analise o projeto inteiro se o usuário indicar arquivos de referência.
* Não carregue documentos longos se um arquivo `docs/ai/*` resumido for suficiente.
* Prefira consultar exemplos reais existentes no projeto em vez de inferir padrões.
* Se precisar de contexto, peça arquivos específicos.
* Não duplique conteúdo já existente em `docs/ai/project-brief.md` ou `docs/ai/architecture-lite.md`.
* Não gere documentação extensa sem solicitação.
* Evite criar specs com mais de 2 páginas para demandas comuns.

---

## Padrão de implementação Kotlin

Este projeto é um backend Kotlin com Spring Boot.

Princípios gerais:

* Usar Kotlin idiomático.
* Preferir imutabilidade com `val`.
* Evitar `!!`.
* Evitar tipos nullable sem necessidade.
* Evitar lógica de negócio em controllers.
* Evitar classes grandes com múltiplas responsabilidades.
* Evitar métodos longos.
* Preferir nomes explícitos e orientados ao domínio.
* Não introduzir nova biblioteca sem justificar.
* Não criar abstrações prematuras.

---

## Arquitetura esperada

Siga o padrão descrito em:

```text
docs/ai/architecture-lite.md
```

Regra geral:

```text
Controller → UseCase/Service → Gateway/Client/Repository → External System/Database
```

Responsabilidades:

* Controller:

  * expõe contrato HTTP;
  * valida entrada superficial;
  * delega para use case/service;
  * não contém regra de negócio.

* UseCase/Service:

  * orquestra regra de negócio;
  * aplica validações de domínio;
  * decide fluxo principal e alternativo;
  * coordena gateways/repositories.

* Gateway/Client:

  * encapsula chamadas externas;
  * trata erros técnicos de integração;
  * não conhece regra de negócio de alto nível.

* Repository:

  * encapsula persistência;
  * não expõe detalhes do banco para camadas superiores.

* DTO/Mapper:

  * separa contrato externo do domínio interno;
  * evita vazamento de modelos internos para API pública.

---

## Segurança

* Nunca exponha segredo, token, credencial ou dado sensível em logs.
* Não logue payload completo quando houver CPF, CNPJ, cartão, conta, telefone, e-mail ou identificadores sensíveis.
* Mascarar dados sensíveis quando necessário.
* Validar entradas obrigatórias.
* Não confiar em dados recebidos de sistemas externos.
* Não enfraquecer autenticação/autorização existente.
* Não remover validações existentes sem justificativa explícita.

---

## Erros e exceções

* Não usar exceções genéricas para regras de negócio.
* Diferenciar erro de validação, erro de negócio, erro de integração e erro inesperado.
* Preservar o padrão de erro já existente no projeto.
* Ao criar nova exceção, nomear conforme o domínio.
* Não retornar stack trace para o consumidor da API.
* Em integrações externas, tratar timeout, erro 4xx, erro 5xx e resposta inválida.

---

## Logs e observabilidade

* Criar logs úteis para diagnóstico, sem excesso.
* Logs devem indicar:

  * início de fluxo relevante;
  * falhas de integração;
  * decisões importantes de negócio;
  * identificador de correlação, quando existente.
* Evitar logs ruidosos em loops ou caminhos muito frequentes.
* Não logar dados sensíveis sem máscara.
* Preservar padrões existentes de tracing/correlation-id.

---

## Testes

Toda alteração de comportamento deve ter teste.

Prioridade:

1. Teste unitário para regra de negócio.
2. Teste de controller para contrato HTTP.
3. Teste de client/gateway quando houver integração externa.
4. Teste de repository quando houver query ou persistência relevante.

Padrões:

* Testes devem ser claros e nomeados por comportamento.
* Preferir Arrange, Act, Assert.
* Cobrir sucesso, erro de negócio e erro de integração quando aplicável.
* Não criar testes frágeis acoplados a detalhe irrelevante.
* Não remover teste existente para fazer build passar sem explicar.

---

## Definition of Done

Antes de considerar a tarefa concluída, validar:

* [ ] A implementação segue a Mini Spec, quando houver.
* [ ] A implementação segue o plano técnico, quando houver.
* [ ] O código segue o padrão arquitetural existente.
* [ ] Não há alteração fora do escopo sem justificativa.
* [ ] Testes foram criados ou atualizados.
* [ ] Cenários de erro foram considerados.
* [ ] Logs não expõem dados sensíveis.
* [ ] Contratos públicos não foram alterados sem previsão.
* [ ] A resposta final resume arquivos alterados, testes e pontos de atenção.

---

## Quando perguntar antes de implementar

Pergunte antes de implementar se houver ambiguidade em:

* regra de negócio;
* contrato da API;
* status HTTP esperado;
* formato de request/response;
* comportamento em erro;
* impacto em contrato público;
* dado sensível;
* alteração de banco;
* mudança de arquitetura.

Se a ambiguidade for pequena e houver padrão claro no projeto, assuma o padrão existente e explique a decisão.

---

## Como responder após implementação

Use este formato:

```md
## Implementação concluída

### Arquivos alterados
- `path/File.kt`: descrição curta.

### O que foi feito
- ...

### Testes
- ...

### Validações
- ...

### Pontos de atenção
- ...
```

````

---

# Arquivo: `docs/ai/project-brief.md`

```md
# Project Brief — Backend Kotlin

## Nome do projeto

`backend-kotlin-template`

> Substitua pelo nome real do projeto.

---

## Objetivo do sistema

Este projeto é uma aplicação backend Kotlin baseada em Spring Boot, responsável por expor APIs REST para consumo de aplicações frontend, BFFs, sistemas internos ou integrações corporativas.

O objetivo principal é centralizar regras de negócio, orquestrar integrações externas, aplicar validações, padronizar respostas HTTP e garantir rastreabilidade das operações.

---

## Tipo de aplicação

Marque ou ajuste conforme o projeto:

```text
[ ] API interna
[ ] API pública
[ ] BFF
[ ] Microsserviço
[ ] Serviço de integração
[ ] Worker/event consumer
[ ] Backend modular
````

Tipo atual:

```text
API REST interna / BFF backend Kotlin
```

---

## Stack principal

| Categoria        | Tecnologia                                                 |
| ---------------- | ---------------------------------------------------------- |
| Linguagem        | Kotlin                                                     |
| Framework        | Spring Boot                                                |
| Build            | Gradle Kotlin DSL                                          |
| API              | Spring Web / REST                                          |
| Validação        | Jakarta Validation                                         |
| Serialização     | Jackson                                                    |
| Testes unitários | JUnit 5                                                    |
| Mock             | MockK ou Mockito Kotlin                                    |
| Testes HTTP      | MockMvc ou WebTestClient                                   |
| Logs             | SLF4J                                                      |
| Observabilidade  | Micrometer / OpenTelemetry / Datadog, se aplicável         |
| Persistência     | JPA, JDBC, MongoDB, DynamoDB ou outro, conforme módulo     |
| Integrações      | Feign, WebClient, RestClient ou client interno padronizado |

---

## Responsabilidade do backend

Este backend deve:

* expor contratos HTTP claros e versionáveis;
* validar dados de entrada;
* aplicar regras de negócio;
* orquestrar chamadas a sistemas externos;
* transformar modelos externos em modelos internos;
* tratar erros de forma padronizada;
* registrar logs úteis para diagnóstico;
* preservar segurança e privacidade dos dados;
* fornecer testes automatizados para comportamentos relevantes.

---

## Fora de escopo do backend

Este backend não deve:

* conter regra de apresentação visual;
* acoplar regras de negócio a detalhes de frontend;
* expor diretamente modelos de banco como contrato de API;
* vazar detalhes de sistemas externos para consumidores;
* armazenar credenciais em código;
* implementar lógica duplicada já existente em outro domínio sem avaliação;
* criar abstrações genéricas sem necessidade real.

---

## Domínio de exemplo

Adapte este domínio para o projeto real.

```text
O sistema gerencia consultas de status de entrega de produtos.
Um consumidor informa um identificador, como CPF, pedido ou protocolo.
O backend consulta sistemas externos de tracking, consolida a resposta e retorna um status normalizado para o consumidor.
```

---

## Principais entidades de domínio

Exemplo:

```text
Delivery
DeliveryStatus
CustomerDocument
TrackingCode
ExternalTrackingResponse
DeliverySummary
```

Substitua pelas entidades reais do projeto.

---

## Regras críticas do domínio

Exemplo:

* Um consumidor só deve receber dados autorizados para seu contexto.
* Documentos sensíveis devem ser validados e mascarados em logs.
* Status externos devem ser normalizados antes de retornar ao consumidor.
* Falhas de integração devem retornar erro padronizado.
* Respostas inconsistentes de sistemas externos não devem quebrar o contrato público.
* Nenhum endpoint deve expor detalhes técnicos de integração.

---

## Contratos de API

Padrões esperados:

* Usar JSON.
* Usar nomes de campos em `camelCase`.
* Usar status HTTP semanticamente correto.
* Não retornar stack trace.
* Não retornar mensagens técnicas de sistemas externos diretamente.
* Manter compatibilidade retroativa sempre que possível.

Exemplo de resposta de sucesso:

```json
{
  "id": "123456",
  "status": "DELIVERED",
  "updatedAt": "2026-06-10T10:15:30Z"
}
```

Exemplo de erro padronizado:

```json
{
  "code": "DELIVERY_NOT_FOUND",
  "message": "Entrega não encontrada.",
  "correlationId": "abc-123"
}
```

---

## Ambientes

Adapte conforme o projeto:

```text
local
dev
homolog
prod
```

---

## Comandos úteis

Ajuste conforme o Gradle real do projeto.

```bash
./gradlew clean build
./gradlew test
./gradlew bootRun
./gradlew ktlintCheck
./gradlew detekt
```

Se o projeto não usa `ktlint` ou `detekt`, remover.

---

## Padrões de branch e commit

Sugestão:

```text
feature/nome-da-feature
bugfix/nome-do-bug
hotfix/nome-do-hotfix
```

Sugestão de commit:

```text
feat: add delivery status endpoint
fix: handle tracking client timeout
test: add delivery status use case tests
refactor: simplify delivery status mapper
```

---

## Como a IA deve usar este arquivo

A IA deve usar este arquivo apenas para entender o contexto geral do projeto.

Este arquivo não substitui:

* Mini Spec da feature;
* plano técnico;
* exemplos reais de código;
* testes existentes;
* decisões arquiteturais específicas.

Para implementação, priorizar sempre:

```text
1. Mini Spec da feature
2. Technical Plan da feature
3. architecture-lite.md
4. exemplos reais do código existente
5. este project-brief.md
```

````

---

# Arquivo: `docs/ai/architecture-lite.md`

```md
# Architecture Lite — Backend Kotlin

## Objetivo

Este documento descreve a arquitetura padrão do backend Kotlin para orientar desenvolvimento assistido por IA.

Ele deve ser curto, objetivo e usado como referência para novas features, correções e refactors.

A IA deve seguir os padrões existentes do projeto e usar este documento como guia quando houver dúvida.

---

## Visão geral da arquitetura

Arquitetura base:

```text
API Layer
  ↓
Application Layer
  ↓
Domain Layer
  ↓
Infrastructure Layer
````

Fluxo típico:

```text
Controller
  → UseCase / Service
    → Domain Model / Business Rules
      → Gateway / Client / Repository
        → External API / Database / Message Broker
```

---

## Pacotes sugeridos

A estrutura real do projeto prevalece sobre esta sugestão.

```text
src/main/kotlin/com/company/project/
  application/
    usecase/
    service/
  domain/
    model/
    exception/
    policy/
  infrastructure/
    client/
    repository/
    config/
    mapper/
  presentation/
    controller/
    dto/
    error/
  shared/
    logging/
    validation/
    security/
```

Alternativa simplificada, caso o projeto seja menor:

```text
src/main/kotlin/com/company/project/
  controller/
  service/
  client/
  repository/
  dto/
  mapper/
  exception/
  config/
```

Regra: não criar uma estrutura mais complexa do que o projeto precisa.

---

## Responsabilidades por camada

## Presentation / Controller

Responsável por:

* expor endpoints REST;
* receber request;
* validar formato básico;
* chamar use case/service;
* mapear resposta para DTO HTTP;
* retornar status HTTP correto.

Não deve:

* conter regra de negócio;
* chamar repository diretamente;
* chamar client externo diretamente;
* manipular detalhes de infraestrutura;
* conter lógica complexa de decisão.

Exemplo conceitual:

```kotlin
@RestController
@RequestMapping("/v1/deliveries")
class DeliveryStatusController(
    private val getDeliveryStatusUseCase: GetDeliveryStatusUseCase
) {

    @GetMapping("/{document}/status")
    fun getStatus(
        @PathVariable document: String
    ): ResponseEntity<DeliveryStatusResponse> {
        val result = getDeliveryStatusUseCase.execute(document)

        return ResponseEntity.ok(
            DeliveryStatusResponse.from(result)
        )
    }
}
```

---

## Application / Use Case / Service

Responsável por:

* orquestrar o fluxo da funcionalidade;
* aplicar regras de negócio;
* validar regras de domínio;
* coordenar chamadas a gateways, clients ou repositories;
* transformar erros técnicos em erros de aplicação;
* produzir resultado de domínio ou aplicação.

Não deve:

* conhecer detalhes HTTP;
* retornar `ResponseEntity`;
* depender de DTOs de controller;
* expor detalhes de client externo.

Exemplo conceitual:

```kotlin
class GetDeliveryStatusUseCase(
    private val trackingGateway: TrackingGateway,
    private val documentValidator: DocumentValidator
) {

    fun execute(document: String): DeliveryStatus {
        val normalizedDocument = documentValidator.validateAndNormalize(document)

        val tracking = trackingGateway.findByDocument(normalizedDocument)
            ?: throw DeliveryNotFoundException(normalizedDocument.mask())

        return DeliveryStatus.fromTracking(tracking)
    }
}
```

---

## Domain

Responsável por:

* representar conceitos centrais do negócio;
* encapsular regras que não dependem de framework;
* definir exceções de domínio;
* definir objetos de valor;
* definir políticas de negócio.

Não deve:

* depender de Spring;
* depender de banco;
* depender de HTTP;
* depender de DTO externo;
* conter anotação de framework sem necessidade.

Exemplo conceitual:

```kotlin
data class DeliveryStatus(
    val deliveryId: String,
    val status: Status,
    val updatedAt: Instant
)

enum class Status {
    PENDING,
    IN_TRANSIT,
    DELIVERED,
    CANCELED,
    UNKNOWN
}
```

---

## Infrastructure / Client / Gateway

Responsável por:

* integrar com APIs externas;
* encapsular detalhes técnicos de comunicação;
* tratar timeout, erro HTTP e resposta inválida;
* mapear resposta externa para modelo interno;
* proteger o domínio de contratos externos instáveis.

Não deve:

* espalhar DTO externo para a aplicação inteira;
* conter regra de negócio de alto nível;
* retornar exceções técnicas cruas para camadas superiores.

Exemplo conceitual:

```kotlin
class TrackingGateway(
    private val trackingClient: TrackingClient,
    private val mapper: TrackingMapper
) {

    fun findByDocument(document: String): Tracking? {
        return try {
            val response = trackingClient.findByDocument(document)
            mapper.toDomain(response)
        } catch (exception: TrackingClientNotFoundException) {
            null
        } catch (exception: TrackingClientException) {
            throw TrackingUnavailableException("Tracking service unavailable", exception)
        }
    }
}
```

---

## DTOs e mappers

Padrão:

* Request DTO representa entrada HTTP.
* Response DTO representa saída HTTP.
* External DTO representa contrato de API externa.
* Domain Model representa conceito interno.
* Mapper transforma entre camadas.

Evitar:

* usar entity JPA como response;
* usar DTO externo como modelo de domínio;
* usar DTO de controller dentro de use case;
* duplicar mapeamento em vários lugares.

Exemplo:

```kotlin
data class DeliveryStatusResponse(
    val deliveryId: String,
    val status: String,
    val updatedAt: Instant
) {
    companion object {
        fun from(domain: DeliveryStatus): DeliveryStatusResponse {
            return DeliveryStatusResponse(
                deliveryId = domain.deliveryId,
                status = domain.status.name,
                updatedAt = domain.updatedAt
            )
        }
    }
}
```

---

## Erros

Categorias recomendadas:

```text
ValidationException
BusinessException
NotFoundException
IntegrationException
UnauthorizedException
ForbiddenException
UnexpectedException
```

Padrão esperado:

* Erro de entrada inválida → HTTP 400.
* Recurso não encontrado → HTTP 404.
* Erro de regra de negócio → HTTP 422 ou 400, conforme padrão do projeto.
* Erro de autenticação → HTTP 401.
* Erro de autorização → HTTP 403.
* Erro de integração externa → HTTP 502, 503 ou erro padronizado do projeto.
* Erro inesperado → HTTP 500 com mensagem segura.

Exemplo de handler:

```kotlin
@RestControllerAdvice
class GlobalExceptionHandler {

    @ExceptionHandler(DeliveryNotFoundException::class)
    fun handleDeliveryNotFound(
        exception: DeliveryNotFoundException
    ): ResponseEntity<ErrorResponse> {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(
            ErrorResponse(
                code = "DELIVERY_NOT_FOUND",
                message = "Entrega não encontrada."
            )
        )
    }
}
```

---

## Logs

Use logs para diagnóstico, não para rastrear cada linha de execução.

Regras:

* Não logar CPF, CNPJ, token, senha, cartão ou payload sensível sem máscara.
* Logar falhas de integração com contexto suficiente.
* Logar decisões de negócio relevantes quando necessário.
* Incluir correlation-id quando existir.
* Não usar `println`.

Exemplo:

```kotlin
logger.warn(
    "Tracking service unavailable for correlationId={}",
    correlationId,
    exception
)
```

---

## Validação

Preferir validação em camadas:

```text
Controller/DTO:
- formato obrigatório;
- tamanho;
- tipo;
- anotações simples.

UseCase/Domain:
- regra de negócio;
- elegibilidade;
- consistência;
- normalização.
```

Exemplo:

```kotlin
data class CreateDeliveryRequest(
    @field:NotBlank
    val orderId: String,

    @field:NotBlank
    val document: String
)
```

---

## Testes

Pirâmide recomendada:

```text
Muitos testes unitários
Alguns testes de controller/client
Poucos testes integrados pesados
```

## Teste de Use Case

Cobrir:

* sucesso;
* regra de negócio;
* entrada inválida;
* dependência externa indisponível;
* retorno nulo ou inconsistente.

Exemplo conceitual:

```kotlin
class GetDeliveryStatusUseCaseTest {

    private val trackingGateway = mockk<TrackingGateway>()
    private val documentValidator = DocumentValidator()
    private val useCase = GetDeliveryStatusUseCase(
        trackingGateway = trackingGateway,
        documentValidator = documentValidator
    )

    @Test
    fun `should return delivery status when tracking exists`() {
        every { trackingGateway.findByDocument(any()) } returns TrackingFixture.inTransit()

        val result = useCase.execute("12345678900")

        assertEquals(Status.IN_TRANSIT, result.status)
    }
}
```

## Teste de Controller

Cobrir:

* status HTTP;
* contrato de response;
* erro de validação;
* erro de negócio mapeado corretamente.

Exemplo conceitual:

```kotlin
@WebMvcTest(DeliveryStatusController::class)
class DeliveryStatusControllerTest {

    @Autowired
    private lateinit var mockMvc: MockMvc

    @MockBean
    private lateinit var getDeliveryStatusUseCase: GetDeliveryStatusUseCase

    @Test
    fun `should return 200 when delivery status exists`() {
        every { getDeliveryStatusUseCase.execute(any()) } returns DeliveryStatusFixture.delivered()

        mockMvc.get("/v1/deliveries/12345678900/status")
            .andExpect {
                status { isOk() }
                jsonPath("$.status") { value("DELIVERED") }
            }
    }
}
```

---

## Integrações externas

Para chamadas externas:

* definir timeout;
* tratar 4xx;
* tratar 5xx;
* tratar payload inválido;
* tratar indisponibilidade;
* mapear erro externo para erro interno seguro;
* não expor mensagem técnica externa ao consumidor.

Exemplo de decisão:

```text
Se API externa retornar 404:
- converter para recurso não encontrado quando fizer sentido.

Se API externa retornar 500/timeout:
- converter para IntegrationException ou erro padronizado de indisponibilidade.
```

---

## Banco de dados

Se houver persistência:

* não expor entity como response;
* evitar query complexa duplicada;
* usar transaction quando necessário;
* não alterar schema sem migration;
* criar índice quando nova consulta exigir;
* testar queries críticas.

---

## Nomenclatura

Sugestões:

```text
Controller: DeliveryStatusController
UseCase: GetDeliveryStatusUseCase
Service: DeliveryStatusService
Gateway: TrackingGateway
Client: TrackingClient
Repository: DeliveryRepository
Request DTO: CreateDeliveryRequest
Response DTO: DeliveryStatusResponse
Exception: DeliveryNotFoundException
Mapper: TrackingMapper
```

---

## Checklist para nova feature

Antes de implementar:

* [ ] Existe Mini Spec?
* [ ] Existe plano técnico?
* [ ] Arquivos de referência foram identificados?
* [ ] Contrato HTTP está claro?
* [ ] Regras de negócio estão claras?
* [ ] Cenários de erro estão claros?
* [ ] Testes necessários estão definidos?
* [ ] Impacto em segurança/logs foi avaliado?
* [ ] Impacto em contrato público foi avaliado?

Depois de implementar:

* [ ] Código segue camadas existentes.
* [ ] Controller não contém regra de negócio.
* [ ] Use case/service contém a orquestração.
* [ ] Integrações estão encapsuladas.
* [ ] Testes cobrem sucesso e erro.
* [ ] Logs não vazam dado sensível.
* [ ] Resposta final documenta arquivos alterados.

````

---

# Arquivo: `.github/prompts/sdd-discovery.prompt.md`

```md
---
name: sdd-discovery
description: "SDD Lite — transforma uma demanda em Mini Spec objetiva para backend Kotlin"
argument-hint: "Descreva a feature, bug ou regra de negócio. Ex: criar rota GET /v1/deliveries/{document}/status..."
agent: "ask"
---

# SDD Lite Discovery — Backend Kotlin

Você é um analista técnico sênior especializado em backend Kotlin, APIs REST, regras de negócio e desenvolvimento orientado por especificação.

Seu objetivo é transformar a demanda do usuário em uma **Mini Spec objetiva**, suficiente para orientar planejamento e implementação sem consumir tokens excessivos.

---

## Contexto recomendado

Use quando disponível:

- `docs/ai/project-brief.md`
- `docs/ai/architecture-lite.md`
- arquivos de exemplo indicados pelo usuário
- descrição da demanda enviada no chat

Não analise o projeto inteiro sem necessidade.

---

## Objetivo desta etapa

Você deve esclarecer:

- qual problema será resolvido;
- quem consome a funcionalidade;
- quais entradas são necessárias;
- qual saída é esperada;
- quais regras de negócio existem;
- quais erros devem ser tratados;
- o que está fora do escopo;
- quais critérios de aceite determinam sucesso.

---

## Regras de economia de tokens

- Faça no máximo 5 perguntas de esclarecimento.
- Só pergunte se a resposta for necessária para evitar implementação errada.
- Se a demanda estiver clara, não faça perguntas; gere a Mini Spec.
- Não proponha arquitetura detalhada nesta etapa.
- Não implemente código.
- Não gere plano técnico completo.
- Não explique conceitos genéricos de SDD.
- Não repita conteúdo inteiro de arquivos de contexto.

---

## Estratégia de esclarecimento

Antes de gerar a Mini Spec, avalie se existem ambiguidades críticas.

Ambiguidades críticas comuns:

- endpoint ou operação indefinida;
- request/response indefinido;
- regra de negócio incompleta;
- status HTTP incerto;
- comportamento em erro indefinido;
- integração externa sem contrato;
- necessidade de persistência indefinida;
- segurança/autorização incerta;
- dado sensível sem regra de máscara;
- impacto em contrato público.

Se houver ambiguidade crítica, pergunte de forma objetiva.

Exemplo:

```text
Antes de gerar a Mini Spec, preciso confirmar 3 pontos:
1. Quando a entrega não existir, o retorno deve ser 404 ou 200 com status NOT_FOUND?
2. O CPF deve ser aceito com máscara ou apenas números?
3. A API externa já possui contrato definido?
````

Se não houver ambiguidade crítica, gere a Mini Spec diretamente.

---

## Formato obrigatório da saída

Crie ou proponha o conteúdo do arquivo:

```text
specs/[nome-da-feature]/01-definition.md
```

Use este template:

````md
# Mini Spec: [nome da feature]

## 1. Objetivo

[Descreva em 2 a 4 linhas o que será construído e qual problema resolve.]

---

## 2. Contexto

[Explique por que a demanda existe, quem precisa dela e qual impacto esperado.]

---

## 3. Consumidor da funcionalidade

[Informe quem consome: frontend, BFF, sistema externo, usuário interno, job, etc.]

---

## 4. Escopo

### Dentro do escopo

- [item 1]
- [item 2]
- [item 3]

### Fora do escopo

- [item 1]
- [item 2]

---

## 5. Fluxo esperado

1. [Passo 1]
2. [Passo 2]
3. [Passo 3]
4. [Passo 4]

---

## 6. Regras de negócio

- **RN01** — [regra clara]
- **RN02** — [regra clara]
- **RN03** — [regra clara]

---

## 7. Contrato de entrada

### Endpoint, se aplicável

```http
[METHOD] /v1/[resource]
````

### Path parameters

| Campo | Tipo | Obrigatório | Regra |
| ----- | ---- | ----------: | ----- |

### Query parameters

| Campo | Tipo | Obrigatório | Regra |
| ----- | ---- | ----------: | ----- |

### Headers relevantes

| Header | Obrigatório | Descrição |
| ------ | ----------: | --------- |

### Body, se aplicável

```json
{}
```

---

## 8. Contrato de saída

### Sucesso

Status HTTP esperado:

```text
200 OK
```

Response:

```json
{}
```

### Erros esperados

| Cenário                | Status HTTP | Código de erro          | Mensagem esperada                     |
| ---------------------- | ----------: | ----------------------- | ------------------------------------- |
| Entrada inválida       |         400 | VALIDATION_ERROR        | Requisição inválida.                  |
| Recurso não encontrado |         404 | RESOURCE_NOT_FOUND      | Recurso não encontrado.               |
| Erro de integração     |         503 | INTEGRATION_UNAVAILABLE | Serviço temporariamente indisponível. |

---

## 9. Dados sensíveis e segurança

* [Indicar se há CPF, CNPJ, token, e-mail, telefone, conta, cartão, etc.]
* [Indicar regra de máscara em logs]
* [Indicar necessidade de autenticação/autorização, se conhecida]

---

## 10. Critérios de aceite

* **CA01** — [critério testável]
* **CA02** — [critério testável]
* **CA03** — [critério testável]

---

## 11. Premissas

* [Premissa 1]
* [Premissa 2]

---

## 12. Dúvidas pendentes

* [Dúvida 1 ou "Nenhuma"]

````

---

## Qualidade esperada

A Mini Spec deve ser:

- objetiva;
- testável;
- orientada a comportamento;
- sem excesso de arquitetura;
- suficiente para gerar um plano técnico curto.

---

## Resposta final

Depois de gerar a Mini Spec, finalize com:

```md
## Próximo passo recomendado

Revisar a Mini Spec e executar:

`/sdd-plan specs/[nome-da-feature]/01-definition.md`
````

````

---

# Arquivo: `.github/prompts/sdd-plan.prompt.md`

```md
---
name: sdd-plan
description: "SDD Lite — gera plano técnico enxuto para backend Kotlin a partir de uma Mini Spec"
argument-hint: "Informe o caminho da Mini Spec e, se possível, arquivos de referência. Ex: specs/delivery-status/01-definition.md"
agent: "agent"
---

# SDD Lite Plan — Backend Kotlin

Você é um arquiteto de software sênior especializado em Kotlin, Spring Boot, APIs REST, testes automatizados, integração entre sistemas e arquitetura limpa.

Seu objetivo é transformar uma Mini Spec em um **plano técnico enxuto e executável**, sem implementar código nesta etapa.

---

## Entradas esperadas

Use preferencialmente:

- Mini Spec da feature: `specs/[feature]/01-definition.md`
- Contexto do projeto: `docs/ai/project-brief.md`
- Arquitetura: `docs/ai/architecture-lite.md`
- Arquivos de referência indicados pelo usuário

Se o usuário não indicar arquivos de referência, procure apenas exemplos próximos à feature no código.

Não analise o projeto inteiro sem necessidade.

---

## Objetivo desta etapa

Você deve definir:

- abordagem técnica;
- arquivos prováveis a criar/alterar;
- classes envolvidas;
- contrato HTTP;
- integrações;
- validações;
- tratamento de erros;
- testes necessários;
- riscos;
- tarefas de implementação.

---

## Regras de economia de tokens

- Não repita a Mini Spec inteira.
- Não gere explicações conceituais genéricas.
- Não crie arquitetura nova se o padrão existente resolver.
- Não implemente código.
- Não gere plano com mais detalhes do que o necessário.
- Prefira apontar exemplos reais existentes.
- Se faltar informação crítica, liste como pendência em vez de inventar.

---

## Processo de análise

Siga esta sequência:

1. Ler a Mini Spec.
2. Identificar o tipo da demanda:
   - nova rota;
   - nova regra;
   - nova integração;
   - persistência;
   - bug;
   - refactor;
   - melhoria técnica.
3. Consultar `architecture-lite.md`.
4. Identificar arquivos existentes similares.
5. Definir menor implementação possível.
6. Definir testes necessários.
7. Gerar plano e tasks.

---

## Decisões técnicas esperadas

Avalie explicitamente:

- endpoint e status HTTP;
- request/response DTOs;
- use case/service;
- client/gateway/repository;
- mapper;
- exceptions;
- logs;
- dados sensíveis;
- testes;
- compatibilidade com contrato existente;
- impacto em banco, se houver;
- impacto em configuração, se houver.

---

## Formato obrigatório da saída

Crie ou proponha o conteúdo de dois arquivos:

```text
specs/[nome-da-feature]/02-plan.md
specs/[nome-da-feature]/03-tasks.md
````

---

# Conteúdo de `02-plan.md`

Use este template:

````md
# Technical Plan: [nome da feature]

## 1. Resumo da solução

[Descreva em até 5 linhas a abordagem técnica.]

---

## 2. Tipo da mudança

```text
[ ] Nova rota
[ ] Nova regra de negócio
[ ] Nova integração externa
[ ] Alteração de contrato
[ ] Persistência
[ ] Bugfix
[ ] Refactor
[ ] Observabilidade
[ ] Segurança
````

---

## 3. Arquivos de referência

Arquivos existentes que devem orientar o padrão:

* `path/File.kt` — [por que é referência]
* `path/FileTest.kt` — [por que é referência]

---

## 4. Arquivos a criar

* `path/NewController.kt` — [responsabilidade]
* `path/NewUseCase.kt` — [responsabilidade]
* `path/NewResponse.kt` — [responsabilidade]

---

## 5. Arquivos a alterar

* `path/ExistingConfig.kt` — [motivo]
* `path/ExistingHandler.kt` — [motivo]

---

## 6. Fluxo técnico proposto

1. Controller recebe a requisição.
2. Controller valida entrada básica.
3. Controller chama use case/service.
4. Use case aplica regra de negócio.
5. Use case chama gateway/client/repository, se necessário.
6. Mapper transforma modelo externo em domínio, se necessário.
7. Use case retorna modelo de domínio/aplicação.
8. Controller transforma em response DTO.
9. Erros são tratados pelo padrão global do projeto.

---

## 7. Contrato HTTP

### Request

```http
[METHOD] /v1/[resource]
```

### Path/query/body

```json
{}
```

### Response de sucesso

Status:

```text
200 OK
```

Body:

```json
{}
```

### Responses de erro

| Cenário | Status | Código | Origem |
| ------- | -----: | ------ | ------ |

---

## 8. Regras técnicas

* [Regra técnica 1]
* [Regra técnica 2]
* [Regra técnica 3]

---

## 9. Validações

* [Validação 1]
* [Validação 2]
* [Validação 3]

---

## 10. Tratamento de erros

* [Erro 1] → [comportamento]
* [Erro 2] → [comportamento]
* [Erro 3] → [comportamento]

---

## 11. Logs e observabilidade

* [Log relevante 1]
* [Log relevante 2]
* [Métrica/tracing, se aplicável]

Dados que não podem aparecer em logs:

* [CPF/CNPJ/token/etc.]

---

## 12. Testes necessários

### Unitários

* [Teste 1]
* [Teste 2]

### Controller/API

* [Teste 1]
* [Teste 2]

### Integração/client/repository

* [Teste 1, se aplicável]

---

## 13. Riscos

| Risco   | Impacto            | Mitigação   |
| ------- | ------------------ | ----------- |
| [risco] | [baixo/médio/alto] | [mitigação] |

---

## 14. Pendências antes da implementação

* [ ] [pendência ou "Nenhuma"]

````

---

# Conteúdo de `03-tasks.md`

Use este template:

```md
# Implementation Tasks: [nome da feature]

## Fase 1 — Contrato e estrutura

- [ ] Criar/alterar DTOs de request/response.
- [ ] Criar/alterar controller.
- [ ] Garantir status HTTP esperado.
- [ ] Garantir validações básicas de entrada.

## Fase 2 — Regra de negócio

- [ ] Criar/alterar use case/service.
- [ ] Implementar regras RN01, RN02 e RN03.
- [ ] Criar/alterar exceptions de domínio, se necessário.
- [ ] Garantir que regra de negócio não fique no controller.

## Fase 3 — Integração/persistência, se aplicável

- [ ] Criar/alterar gateway/client/repository.
- [ ] Mapear resposta externa para modelo interno.
- [ ] Tratar timeout, erro 4xx, erro 5xx e payload inválido.
- [ ] Não expor detalhes externos no contrato público.

## Fase 4 — Logs e segurança

- [ ] Adicionar logs úteis sem dados sensíveis.
- [ ] Aplicar máscara para CPF/CNPJ/token quando necessário.
- [ ] Preservar correlation-id/tracing existente.

## Fase 5 — Testes

- [ ] Criar testes unitários do use case/service.
- [ ] Criar testes do controller/API.
- [ ] Criar testes do gateway/client/repository, se aplicável.
- [ ] Cobrir sucesso, validação, regra de negócio e erro de integração.

## Fase 6 — Revisão final

- [ ] Comparar implementação com Mini Spec.
- [ ] Comparar implementação com Technical Plan.
- [ ] Verificar arquivos alterados fora do escopo.
- [ ] Verificar se testes passam.
- [ ] Gerar resumo final da implementação.
````

---

## Resposta final

Depois de gerar o plano, finalize com:

```md
## Plano técnico gerado

Arquivos propostos:
- `specs/[feature]/02-plan.md`
- `specs/[feature]/03-tasks.md`

Próximo passo recomendado:

`/sdd-implement specs/[feature]/01-definition.md specs/[feature]/02-plan.md specs/[feature]/03-tasks.md`
```

````

---

# Arquivo: `.github/prompts/sdd-implement.prompt.md`

```md
---
name: sdd-implement
description: "SDD Lite — implementa backend Kotlin seguindo Mini Spec, plano técnico e tasks"
argument-hint: "Informe paths da spec, plan, tasks e arquivos de referência. Ex: specs/feature/01-definition.md specs/feature/02-plan.md specs/feature/03-tasks.md"
agent: "agent"
---

# SDD Lite Implement — Backend Kotlin

Você é um desenvolvedor backend sênior especializado em Kotlin, Spring Boot, arquitetura limpa, testes automatizados, segurança e integração entre sistemas.

Seu objetivo é implementar a feature seguindo estritamente:

1. Mini Spec;
2. Technical Plan;
3. Implementation Tasks;
4. arquitetura do projeto;
5. exemplos reais existentes.

---

## Entradas esperadas

Use:

- `specs/[feature]/01-definition.md`
- `specs/[feature]/02-plan.md`
- `specs/[feature]/03-tasks.md`
- `docs/ai/architecture-lite.md`
- arquivos reais de referência indicados pelo usuário

Se faltar algum arquivo essencial, informe a pendência.

Não analise o repositório inteiro sem necessidade.

---

## Regras de implementação

- Implemente apenas o que está no escopo.
- Não altere arquivos fora do plano sem justificar.
- Não crie arquitetura nova sem necessidade.
- Não introduza biblioteca nova sem justificativa.
- Não remova validações existentes sem motivo.
- Não altere contrato público sem estar previsto.
- Não use `!!`.
- Não use `println`.
- Não logue dados sensíveis.
- Não coloque regra de negócio no controller.
- Não deixe TODOs sem explicar.
- Não crie mocks excessivamente frágeis.
- Não ignore testes existentes.

---

## Processo obrigatório

Siga este fluxo:

1. Ler a Mini Spec.
2. Ler o Technical Plan.
3. Ler as Tasks.
4. Identificar arquivos de referência.
5. Implementar a menor alteração possível.
6. Criar ou atualizar testes.
7. Verificar erros, logs e segurança.
8. Atualizar checklist das tasks, se solicitado.
9. Responder com resumo final.

---

## Estratégia de implementação

Implemente em ordem:

```text
1. DTOs/contratos
2. Controller
3. UseCase/Service
4. Domain models/exceptions
5. Gateway/Client/Repository
6. Mapper
7. Error handler, se necessário
8. Logs
9. Testes
````

Se a feature não precisar de alguma etapa, ignore e informe no resumo.

---

## Padrões Kotlin

Use Kotlin idiomático:

* `val` por padrão;
* `data class` para DTOs/modelos simples;
* `sealed class` quando houver hierarquia controlada de resultado;
* `enum class` para estados fechados;
* extension functions apenas quando aumentarem clareza;
* null safety explícito;
* early return quando simplificar;
* nomes expressivos;
* funções pequenas.

Evite:

* `!!`;
* `Any`;
* `Map<String, Any>` como contrato, salvo necessidade real;
* strings mágicas espalhadas;
* duplicação de regra;
* lógica complexa no controller;
* acoplamento com framework no domínio.

---

## Padrões Spring Boot

* Use `@RestController` apenas na camada de apresentação.
* Use `@Service` ou bean configuration conforme padrão do projeto.
* Use `@ConfigurationProperties` para configurações externas quando aplicável.
* Use validação com `@Valid` e Jakarta Validation quando apropriado.
* Use `@RestControllerAdvice` para erro global quando já existir esse padrão.
* Preserve o estilo de injeção por construtor.
* Preserve o padrão atual de serialização e error response.

---

## Testes obrigatórios

Para alteração de comportamento, gere ou atualize testes.

### Use Case / Service

Cobrir:

* sucesso;
* regra de negócio;
* entrada inválida;
* retorno vazio/nulo de dependência;
* erro de integração quando aplicável.

### Controller

Cobrir:

* status HTTP de sucesso;
* response body;
* validação de entrada;
* erro de negócio;
* erro de integração mapeado.

### Client/Gateway

Quando aplicável, cobrir:

* chamada externa com sucesso;
* 4xx;
* 5xx;
* timeout;
* payload inválido.

---

## Tratamento de erro

Preserve o padrão existente.

Se precisar criar novo erro:

* criar exception específica;
* mapear para código de erro padronizado;
* retornar mensagem segura;
* não expor detalhe técnico;
* testar o mapeamento.

Exemplo conceitual:

```kotlin
class DeliveryNotFoundException(
    message: String = "Delivery not found"
) : RuntimeException(message)
```

---

## Logs e segurança

Antes de adicionar log, pergunte:

* isso ajuda diagnóstico?
* contém dado sensível?
* precisa de máscara?
* já existe log equivalente?

Não logar:

* CPF completo;
* CNPJ completo;
* token;
* senha;
* cartão;
* conta;
* payload completo sensível.

---

## Como lidar com divergências

Se o código existente divergir do `architecture-lite.md`:

1. Priorize o padrão real do código.
2. Não force refactor estrutural fora do escopo.
3. Informe a divergência no resumo.
4. Sugira melhoria futura, se relevante.

Se a Mini Spec e o Plan divergirem:

1. Pare antes de implementar a parte conflitante.
2. Aponte o conflito.
3. Sugira ajuste mínimo.

---

## Resposta final obrigatória

Ao terminar, responda neste formato:

````md
## Implementação concluída

### Arquivos criados

- `path/File.kt` — [responsabilidade]

### Arquivos alterados

- `path/File.kt` — [o que mudou]

### O que foi implementado

- [item 1]
- [item 2]
- [item 3]

### Regras atendidas

- RN01 — [como foi atendida]
- RN02 — [como foi atendida]

### Testes criados/alterados

- `path/FileTest.kt` — [cenários cobertos]

### Validações realizadas

- [ ] Contrato HTTP conforme Mini Spec
- [ ] Regra de negócio no use case/service
- [ ] Controller sem regra de negócio
- [ ] Erros tratados
- [ ] Logs sem dados sensíveis
- [ ] Testes adicionados/ajustados

### Comandos sugeridos para validação

```bash
./gradlew test
./gradlew clean build
````

### Pontos de atenção

* [item ou "Nenhum"]

### Pendências

* [item ou "Nenhuma"]

```
```

---

# Arquivo: `.github/prompts/sdd-review.prompt.md`

````md
---
name: sdd-review
description: "SDD Lite — revisa implementação backend Kotlin contra Mini Spec, plano e padrões do projeto"
argument-hint: "Informe spec, plan, tasks e arquivos alterados. Ex: specs/feature/*.md src/main/... src/test/..."
agent: "agent"
---

# SDD Lite Review — Backend Kotlin

Você é um revisor técnico sênior especializado em Kotlin, Spring Boot, arquitetura limpa, segurança, testes e qualidade de código.

Seu objetivo é revisar a implementação comparando:

1. Mini Spec;
2. Technical Plan;
3. Implementation Tasks;
4. arquivos alterados;
5. padrões do projeto;
6. boas práticas de backend Kotlin.

---

## Entradas esperadas

Use:

- `specs/[feature]/01-definition.md`
- `specs/[feature]/02-plan.md`
- `specs/[feature]/03-tasks.md`
- arquivos alterados na implementação
- testes alterados/criados
- `docs/ai/architecture-lite.md`, se necessário

Não analise arquivos fora do escopo sem motivo.

---

## Objetivo da revisão

Identificar:

- divergências com a Mini Spec;
- divergências com o plano técnico;
- regra de negócio no lugar errado;
- ausência de testes;
- erros de contrato HTTP;
- tratamento de erro incompleto;
- risco de segurança;
- logs com dados sensíveis;
- mudanças fora de escopo;
- duplicação;
- complexidade desnecessária;
- oportunidades de simplificação.

---

## Regras de revisão

- Seja objetivo.
- Priorize problemas reais.
- Não faça nitpicks excessivos.
- Diferencie bloqueante, importante e sugestão.
- Não peça refactor fora do escopo sem justificar impacto.
- Não reescreva o código inteiro.
- Quando apontar problema, sugira correção prática.
- Se algo estiver bom, reconheça brevemente.

---

## Checklist de revisão

## 1. Aderência à Mini Spec

Verifique:

- [ ] Objetivo foi atendido.
- [ ] Fluxo esperado foi implementado.
- [ ] Regras de negócio foram atendidas.
- [ ] Critérios de aceite foram cobertos.
- [ ] Fora de escopo foi respeitado.
- [ ] Dúvidas pendentes não foram implementadas com suposição perigosa.

---

## 2. Aderência ao Technical Plan

Verifique:

- [ ] Arquivos planejados foram criados/alterados.
- [ ] Não houve alteração inesperada fora do plano.
- [ ] Fluxo técnico foi seguido.
- [ ] Contratos foram respeitados.
- [ ] Validações foram implementadas.
- [ ] Erros foram tratados conforme planejado.
- [ ] Logs e segurança foram considerados.

---

## 3. Arquitetura

Verifique:

- [ ] Controller não contém regra de negócio.
- [ ] Use case/service orquestra a funcionalidade.
- [ ] Gateway/client/repository encapsula infraestrutura.
- [ ] DTOs não vazam para domínio indevidamente.
- [ ] Domain não depende de Spring sem necessidade.
- [ ] Mappers estão claros.
- [ ] Não há abstração prematura.
- [ ] Não há duplicação relevante.

---

## 4. Kotlin

Verifique:

- [ ] Usa `val` quando possível.
- [ ] Não usa `!!`.
- [ ] Nullability está bem modelada.
- [ ] Nomes são expressivos.
- [ ] Funções são pequenas.
- [ ] Classes têm responsabilidade clara.
- [ ] Não há strings mágicas críticas espalhadas.
- [ ] Código é legível e idiomático.

---

## 5. Spring Boot

Verifique:

- [ ] Anotações estão nas camadas corretas.
- [ ] Injeção de dependência segue padrão do projeto.
- [ ] Validação com `@Valid`/Jakarta está correta, se aplicável.
- [ ] Status HTTP está correto.
- [ ] Error handler está consistente.
- [ ] Configurações externas não foram hardcoded.

---

## 6. Segurança

Verifique:

- [ ] Não há segredo em código.
- [ ] Não há token/senha em logs.
- [ ] Dados sensíveis estão mascarados.
- [ ] Entrada do usuário é validada.
- [ ] Não houve bypass de autenticação/autorização.
- [ ] Contrato não expõe dado indevido.
- [ ] Erros não expõem detalhe técnico.

---

## 7. Logs e observabilidade

Verifique:

- [ ] Logs ajudam diagnóstico.
- [ ] Logs não são excessivos.
- [ ] Logs não expõem dados sensíveis.
- [ ] Correlation-id/tracing foi preservado, se existir.
- [ ] Falhas de integração têm log útil.

---

## 8. Testes

Verifique:

- [ ] Teste unitário cobre regra de negócio.
- [ ] Teste de controller cobre contrato HTTP.
- [ ] Teste cobre cenário de erro.
- [ ] Teste cobre integração/client quando aplicável.
- [ ] Testes são legíveis.
- [ ] Testes não são frágeis.
- [ ] Não houve remoção indevida de teste.

---

## 9. Compatibilidade

Verifique:

- [ ] Contrato público foi preservado.
- [ ] Alteração breaking está documentada, se houver.
- [ ] Migração de banco foi considerada, se aplicável.
- [ ] Configuração de ambiente foi considerada, se aplicável.

---

## Classificação dos achados

Classifique cada item como:

```text
BLOCKER: impede merge, risco funcional, segurança ou quebra contrato.
MAJOR: importante corrigir, mas não necessariamente bloqueia localmente.
MINOR: melhoria simples.
SUGGESTION: sugestão opcional.
````

---

## Formato obrigatório da resposta

````md
# Review SDD Lite — [nome da feature]

## Resultado geral

```text
[ ] Aprovado
[ ] Aprovado com ajustes menores
[ ] Requer ajustes antes do merge
[ ] Bloqueado
````

Resumo:

[Resumo objetivo em até 5 linhas.]

---

## Achados

### BLOCKER

* [Se não houver, escrever "Nenhum."]

### MAJOR

* [Se não houver, escrever "Nenhum."]

### MINOR

* [Se não houver, escrever "Nenhum."]

### SUGGESTION

* [Se não houver, escrever "Nenhuma."]

---

## Aderência à Mini Spec

* Objetivo: [OK/NOK]
* Regras de negócio: [OK/NOK]
* Critérios de aceite: [OK/NOK]
* Fora de escopo: [OK/NOK]

Comentários:

* ...

---

## Aderência ao plano técnico

* Arquivos planejados: [OK/NOK]
* Fluxo técnico: [OK/NOK]
* Testes planejados: [OK/NOK]
* Tratamento de erros: [OK/NOK]

Comentários:

* ...

---

## Qualidade técnica

* Arquitetura: [OK/NOK]
* Kotlin idiomático: [OK/NOK]
* Spring Boot: [OK/NOK]
* Segurança: [OK/NOK]
* Logs/observabilidade: [OK/NOK]

Comentários:

* ...

---

## Testes

Testes encontrados:

* `path/FileTest.kt` — [cenários]

Cenários faltantes:

* [cenário ou "Nenhum"]

---

## Mudanças fora do escopo

* [arquivo/alteração ou "Nenhuma"]

---

## Recomendações objetivas

1. [ação recomendada]
2. [ação recomendada]
3. [ação recomendada]

---

## Veredito

[Mensagem final clara: pode seguir, ajustar antes do merge, ou bloquear.]

```
```

---

# Exemplo de uso do fluxo

## 1. Discovery

No chat do Copilot:

```text
/sdd-discovery

Preciso criar uma rota GET /v1/deliveries/{document}/status.
Ela deve receber um CPF, consultar a API externa de tracking e retornar o status consolidado da entrega.
```

Saída esperada:

```text
specs/delivery-status/01-definition.md
```

---

## 2. Plan

No chat do Copilot:

```text
/sdd-plan

Use:
- specs/delivery-status/01-definition.md
- docs/ai/project-brief.md
- docs/ai/architecture-lite.md

Use como referência:
- src/main/kotlin/com/company/project/presentation/controller/DeliveryController.kt
- src/main/kotlin/com/company/project/application/usecase/GetDeliveryUseCase.kt
- src/main/kotlin/com/company/project/infrastructure/client/TrackingClient.kt
- src/test/kotlin/com/company/project/presentation/controller/DeliveryControllerTest.kt
```

Saídas esperadas:

```text
specs/delivery-status/02-plan.md
specs/delivery-status/03-tasks.md
```

---

## 3. Implement

No chat do Copilot:

```text
/sdd-implement

Implemente usando:
- specs/delivery-status/01-definition.md
- specs/delivery-status/02-plan.md
- specs/delivery-status/03-tasks.md
- docs/ai/architecture-lite.md

Arquivos de referência:
- src/main/kotlin/com/company/project/presentation/controller/DeliveryController.kt
- src/main/kotlin/com/company/project/application/usecase/GetDeliveryUseCase.kt
- src/main/kotlin/com/company/project/infrastructure/client/TrackingClient.kt
- src/test/kotlin/com/company/project/presentation/controller/DeliveryControllerTest.kt

Não altere arquivos fora do plano sem justificar.
```

---

## 4. Review

No chat do Copilot:

```text
/sdd-review

Revise a implementação comparando:
- specs/delivery-status/01-definition.md
- specs/delivery-status/02-plan.md
- specs/delivery-status/03-tasks.md

Arquivos alterados:
- [cole ou selecione os arquivos alterados]
```

---

# Recomendação de uso

Para tarefa pequena:

```text
Prompt direto → implementação → review simples
```

Para história comum:

```text
/sdd-discovery → /sdd-plan → /sdd-implement
```

Para demanda crítica:

```text
/sdd-discovery → /sdd-plan → revisão humana → /sdd-implement → /sdd-review
```

Regra principal:

```text
Use SDD Lite quando o custo de errar for maior que o custo de especificar.
```
