# Spec — Serviço de Enriquecimento de Endereços de Entrega

Oct 22, 2026 · @italo

## 1. Contexto e objetivo

O serviço recebe o endereço de entrega de um pedido e devolve a melhor versão possível dele, sem nunca trocar um endereço certo por um errado. O endereço de entrada é a verdade padrão: um campo só muda com evidência forte e sem ambiguidade. Na dúvida, o serviço não altera nada.

Hoje os endereços chegam com vários problemas, muitas vezes combinados:

- CEP inválido, inexistente ou desmembrado, com logradouro, cidade e UF corretos.
- Número e complemento embutidos no campo logradouro.
- Logradouro e cidade abreviados ("S Paulo", "R. Pres. J. Kubitschek").
- Erros de digitação em qualquer campo.

Para corrigir, o serviço combina duas fontes: o histórico de endereços do cliente (EQ3), que completa número e complemento, e a base de CEPs (CW3), que padroniza cidade e logradouro e corrige CEPs inexistentes.

## 2. Escopo e premissas

**Entrada:** id do cliente e endereço do pedido (logradouro, número, complemento, bairro, cidade, UF, CEP).

**Saída:** endereço final, status, score de confiança e trilha de auditoria de cada campo alterado.

| Dependência | O que oferece | Limitação conhecida |
| --- | --- | --- |
| API EQ3 | Todos os endereços já cadastrados do cliente | Não indica quais endereços tiveram entrega bem-sucedida |
| API CW3, rota por CEP | Endereço oficial de um CEP | CEPs gerais de município não trazem logradouro |
| API CW3, rota por logradouro | Lista de endereços para logradouro + cidade + UF | Pode devolver vários CEPs para a mesma rua (faixas de numeração) |
| Amazon Bedrock (LLM) | Parsing de logradouros que o parser determinístico não resolve | Não determinístico; usado só como fallback e sempre validado |

Premissas:

- Não há código IBGE disponível. Cidades são comparadas pelo nome.
- A UF de entrada nunca é corrigida. Se estiver errada, nenhum enriquecimento é possível.
- Fora do escopo: geocodificação, validação de bairro e revisão manual (o serviço só registra os casos para análise).

## 3. Regras de negócio invioláveis

Estas regras valem para qualquer resposta do serviço. O validador final (seção 11) as verifica de forma independente antes de devolver o resultado.

1. **UF de saída idêntica à de entrada.** Sem exceções.
2. **Cidade e logradouro de saída com similaridade alta em relação aos de entrada.** O serviço padroniza a grafia, não troca de rua nem de cidade.
3. **O CEP só muda se o de entrada for inexistente** (ausente, com formato inválido, ou não encontrado no CW3; formato inválido conta como inexistente). Um CEP que existe nunca é trocado, mesmo que conflite com o restante do endereço. CEP desmembrado também nunca é trocado.
4. **Nenhuma informação da entrada é removida.** Número e complemento (do parser ou do EQ3) só preenchem campos vazios ou substituem o valor da entrada por um que o contém e agrega informação (seção 7); o CW3 só substitui cidade, logradouro e (no caso da regra 3) CEP.
5. **O CW3 é tudo ou nada.** Se qualquer validação falhar, o serviço devolve a entrada mais o enriquecimento do EQ3, se houver.
6. **UF divergente descarta tudo.** Se o CEP de entrada existe no CW3 com outra UF, o serviço devolve a entrada pura, sem o enriquecimento do EQ3.
7. **Na dúvida, não altera.** Empate entre candidatos, ambiguidade ou score na zona cinza resultam em não alterar o campo.

## 4. Visão geral do fluxo

O pedido passa por normalização, EQ3 e uma das duas rotas do CW3, e termina em uma de três saídas. Só o caminho em que todas as validações e o validador final aprovam produz um endereço enriquecido pelo CW3.

&#91;embedded content: fluxo do serviço · 3 saídas possíveis\]

A entrada original e o resultado do EQ3 são guardados separadamente durante todo o fluxo, para que a decisão final possa escolher entre eles sem refazer nada.

## 5. Etapa 0: normalização e parsing determinístico

A etapa 0 produz uma versão normalizada e estruturada do endereço, usada só para comparação. O original é preservado intacto e é a base de qualquer resposta.

**Normalização** (aplicada à entrada e a todo endereço vindo do EQ3 e do CW3):

- Maiúsculas, sem acentos, sem pontuação redundante, espaços colapsados.
- Expansão de abreviações por dicionário versionado: R→RUA, AV→AVENIDA, TV→TRAVESSA, S/STO→SAO/SANTO, STA→SANTA, PRES→PRESIDENTE, DR→DOUTOR, CEL→CORONEL, PROF→PROFESSOR, FDS→FUNDOS, entre outras.
- Separação do tipo de via (RUA, AVENIDA, RODOVIA…) do nome da via.

**Parsing do logradouro** extrai número e complemento quando vêm embutidos:

- Número após vírgula, "Nº", "N." ou no fim do texto; "S/N" tratado como sem número.
- Complemento por palavra-chave: APTO, AP, BL, BLOCO, CASA, LOJA, SALA, FUNDOS, CONJ, LOTE, QD.
- Números que fazem parte do nome não são extraídos: seguidos de "DE" ("25 DE MARCO", "9 DE JULHO"), precedidos de "KM", rodovias ("BR-116", "SP-280") e vias numeradas ("RUA 10").

**Classificação do CEP:** `AUSENTE`, `FORMATO_INVALIDO` (diferente de 8 dígitos, 00000000, dígitos todos iguais) ou `FORMATO_VALIDO`. A existência só é confirmada no CW3.

**Status de confiança do parser.** O parser devolve `CONFIANTE` ou `INCERTO` com a lista de motivos. É `INCERTO` quando ocorre qualquer um destes sinais:

| Motivo | Exemplo |
| --- | --- |
| `TOKEN_NAO_CLASSIFICADO` | "R DAS FLORES 123 FDS" com FDS fora do dicionário |
| `MULTIPLOS_NUMEROS` | "RUA 10 123 2" |
| `CONFLITO_NUMERO` | Campo número = 45 e logradouro contém "…, 123" |
| `ABREVIACAO_DESCONHECIDA` | Token de até 4 letras fora do dicionário ("AV PRF JK") |
| `MULTIPLAS_INTERPRETACOES` | "RUA DA PAZ 9 B": número 9B ou número 9 + complemento B |
| `SEM_ESTRUTURA` | Nenhum tipo de via nem padrão reconhecido |

Exemplo de saída:

```json
{
  "tipo": "RUA", "nome": "DAS FLORES",
  "numero": "123", "complemento": null,
  "status": "INCERTO",
  "motivos": ["TOKEN_NAO_CLASSIFICADO:FDS", "MULTIPLOS_NUMEROS"]
}
```

Todo motivo é registrado em métrica. Motivos recorrentes viram novas entradas do dicionário ou novas regras, reduzindo o uso de IA com o tempo.

## 6. Fallback de IA (Bedrock) para parsing incerto

O fallback de IA entra já na primeira versão. O LLM atua só na organização do logradouro, e só quando o parser determinístico devolve `INCERTO`. Ele nunca decide se o endereço é enriquecido: a saída dele passa pelas mesmas validações que a do parser.

**Chamada:**

- Entrada: logradouro, número e complemento originais, mais os motivos de incerteza.
- Saída obrigatória em JSON estruturado (tipo, nome, número, complemento), temperatura 0.
- Prompt proíbe adicionar informação que não esteja no texto de entrada.
- Timeout curto. Em erro ou timeout, segue com o parse determinístico.

**Trava de fidelidade.** A resposta do LLM só é aceita se todo token devolvido existir na entrada, de forma literal ou como expansão de abreviação conhecida do dicionário. Qualquer token novo invalida a resposta, e o serviço segue com o parse determinístico.

**Pior caso.** Se o LLM errar o parse, os candidatos do CW3 não passam na validação e o serviço devolve entrada + EQ3, que é o comportamento seguro.

**Evolução futura (opcional, após medição):** usar o LLM também quando o parse foi `CONFIANTE`, mas o CW3 não encontrou candidato. No máximo uma nova tentativa, com as mesmas validações, porque cada tentativa extra aumenta a chance de um candidato errado passar por acaso.

**Uso offline recomendado:** analisar casos de fallback e zona cinza para sugerir abreviações e regras novas, e ajudar a rotular a base de calibração (seção 14).

## 7. Etapa 1: enriquecimento via EQ3

O EQ3 completa número e complemento a partir de endereços do cliente que sejam inequivocamente o mesmo endereço. Ele preenche campos vazios ou agrega informação a um valor que já veio na entrada, mas nunca o contradiz.

**Seleção de candidatos:**

1. Buscar todos os endereços do cliente no EQ3 e normalizar cada um (etapa 0).
2. Manter só os que têm UF idêntica à da entrada.
3. Manter só os que passam na comparação de cidade e logradouro (seção 9).

**Regras de preenchimento:**

- **Número vazio:** preenchido só se todos os candidatos compatíveis tiverem o mesmo número (consenso). Números diferentes entre candidatos = ambíguo, não preenche.
- **Complemento vazio:** preenchido só se o número da entrada (original ou recém-preenchido) bater com o do candidato e se todos os candidatos com esse número concordarem no complemento.
- **Valor existente que agrega:** o valor do EQ3 substitui o da entrada quando contém todos os tokens normalizados da entrada, na mesma ordem, e acrescenta pelo menos um. Exemplo: complemento "AP 12" vira "APTO 12 BLOCO B" (abreviações expandidas antes de comparar). Também exige consenso entre os candidatos.
- **Número existente:** só agrega se a parte numérica for idêntica e o EQ3 acrescentar um sufixo ("123" → "123 A"). Esse caso é registrado com motivo próprio (`NUMERO_SUFIXO_EQ3`) para monitoramento, porque um sufixo pode indicar outro imóvel.
- **Contradição:** se o valor do EQ3 não contém o da entrada (complemento "APTO 12" × "APTO 31"), mantém a entrada.
- **Número divergente:** se a entrada tem número e o candidato tem outro, o candidato é outro endereço na mesma rua e é descartado.
- **Parser:** número e complemento extraídos do logradouro seguem as mesmas regras e são aplicados em todas as saídas, exceto a entrada pura (`UF_DIVERGENTE_CEP`).

Como o EQ3 não informa sucesso de entrega, o consenso entre candidatos substitui esse sinal. O resultado do EQ3 é guardado separadamente da entrada original, porque pode ser descartado no fim (regra 6).

## 8. Etapas 2 e 3: roteamento e validação no CW3

O CW3 substitui cidade e logradouro pela grafia canônica e, só quando o CEP de entrada é inexistente, também o CEP. Qualquer falha de validação resulta em entrada + EQ3.

**Roteamento:**

| Situação do CEP de entrada | Rota | Pode trocar o CEP? |
| --- | --- | --- |
| `FORMATO_VALIDO` | Por CEP | Não |
| `FORMATO_VALIDO`, mas não encontrado no CW3 | Por logradouro | Sim |
| `FORMATO_VALIDO`, CW3 retorna erro "cep desmembrado" | Por logradouro | Não (só padroniza cidade e logradouro) |
| `FORMATO_VALIDO`, CW3 retorna erro "cep genérico" | Permanece na rota por CEP (passo 3 abaixo) | Não |
| `AUSENTE` ou `FORMATO_INVALIDO` | Por logradouro | Sim |

**Rota por CEP — validações, nesta ordem:**

1. Erro "cep desmembrado" → segue para a rota por logradouro apenas para padronizar cidade e logradouro. O CEP de entrada é sempre mantido.
2. UF do CW3 diferente da entrada → status `UF_DIVERGENTE_CEP`, devolve a entrada pura (descarta o EQ3).
3. Erro "cep genérico" (CEP único do município) → valida UF e cidade com os dados retornados. Se passarem, enriquece só a cidade e mantém CEP e logradouro de entrada. Se o retorno não trouxer cidade e UF, nada do CW3 é aplicado (entrada + EQ3).
4. Cidade não passa na comparação → `CONFLITO_CEP`, devolve entrada + EQ3.
5. Logradouro não passa na comparação → `CONFLITO_CEP`, devolve entrada + EQ3.
6. Tudo passa → substitui cidade e logradouro pela grafia do CW3; mantém o CEP.

**Rota por logradouro:**

1. Consultar com o logradouro normalizado (nome, sem número e complemento), a cidade e a UF de entrada.
2. Filtrar candidatos: UF idêntica, cidade e logradouro passando na comparação (seção 9).
3. Se sobrarem candidatos de CEPs diferentes para a mesma rua (faixas de numeração, lado par/ímpar), usar o número para escolher a faixa.
4. Exigir um candidato único e inequívoco: score acima do limiar alto e margem mínima para o segundo colocado.
5. Se houver candidato único → substitui cidade, logradouro e CEP. Exceção: se o CEP de entrada for desmembrado, substitui só cidade e logradouro e mantém o CEP (status ENRIQUECIDO\_SEM\_CEP).
6. Se o número não desempatar o CEP, mas todos os candidatos concordarem na grafia de cidade e logradouro e ela passar na validação de similaridade → substitui só cidade e logradouro e mantém o CEP de entrada (status `ENRIQUECIDO_SEM_CEP`). Se a grafia não for consensual → não altera, status `AMBIGUO`.
7. Sem candidato válido → `SEM_CANDIDATO`, devolve entrada + EQ3.

A rota por logradouro não consegue detectar UF errada, porque já consulta com a UF de entrada. Nesse caso ela simplesmente não encontra candidato e devolve entrada + EQ3.

## 9. Algoritmos de similaridade

A comparação usa alinhamento token a token como regra principal e score de similaridade como segundo filtro. Score puro não basta, porque cidades e ruas diferentes podem ter nomes muito parecidos.

**Casos que o score puro aprovaria por engano:**

| Entrada | Candidato | Por que é diferente |
| --- | --- | --- |
| SANTA RITA (MG) | SANTA RITA DO SAPUCAI (MG) | Municípios distintos |
| SAO JOSE (SC) | SAO JOSE DO CEDRO (SC) | Municípios distintos |
| RUA 10 | RUA 12 | Vias numeradas distintas |
| RUA SAO JOAO | AVENIDA SAO JOAO | Podem ser vias distintas na mesma cidade |

**Regra de alinhamento (cidade e nome do logradouro):**

1. Cada token da entrada precisa casar com um token do candidato, na mesma ordem.
2. Um casamento pode ser: exato; por abreviação do dicionário (S→SAO, STA→SANTA); por inicial (J→JOSE); ou por similaridade de caractere alta, para erro de digitação (CAMPINSA→CAMPINAS).
3. Nenhum token pode sobrar em nenhum dos lados. Exceção: preposições (DE, DA, DO, DAS, DOS, E).
4. Tokens numéricos precisam ser idênticos.
5. Iniciais só casam se o restante do nome casar por completo, para evitar que "J SILVA" case com qualquer "J… SILVA" sem outra evidência.

**Score composto** (calculado só para pares que passaram no alinhamento): média ponderada de Jaro-Winkler e token set ratio sobre os textos normalizados e expandidos. Os pesos e limiares saem da calibração (seção 14).

**Tipo de via:** se os dois lados têm tipo e eles divergem (RUA × AVENIDA), o par só é aceito com evidência extra: o CEP de entrada bater com o do candidato. Se um dos lados não tem tipo, o tipo não penaliza.

**Decisão:**

| Faixa de score | Decisão |
| --- | --- |
| ≥ limiar alto (ponto de partida: 0,92) e margem ≥ 0,05 para o 2º colocado | Aceita |
| Entre limiar baixo (ponto de partida: 0,80) e alto, ou margem < 0,05 | Zona cinza: não altera, registra |
| < limiar baixo | Rejeita |

Os valores 0,92, 0,80 e 0,05 são pontos de partida, a confirmar na calibração.

**Dicionário de municípios (opcional, recomendado):** um arquivo estático com os nomes dos municípios por UF, usado só para normalização. Ele confirma, por exemplo, que "S PAULO" em SP só pode ser SAO PAULO, e permite detectar quando a abreviação na entrada é ambígua dentro da UF.

## 10. Matriz de decisão de saída

A saída é sempre uma de três composições: entrada pura, entrada + EQ3, ou entrada + EQ3 + CW3. Em "entrada + EQ3", entram o número e o complemento extraídos pelo parser e os vindos do EQ3, conforme as regras da seção 7.

| Situação | Status | Saída |
| --- | --- | --- |
| CEP existe no CW3 com UF diferente | `UF_DIVERGENTE_CEP` | Entrada pura |
| CEP existe, mesma UF, cidade ou logradouro não batem | `CONFLITO_CEP` | Entrada + EQ3 |
| Rota por logradouro sem candidato válido | `SEM_CANDIDATO` | Entrada + EQ3 |
| Candidatos empatados, zona cinza ou grafia não consensual | `AMBIGUO` | Entrada + EQ3 |
| CW3 indisponível (timeout, erro) | `CW3_INDISPONIVEL` | Entrada + EQ3 |
| Validador final reprova o resultado do CW3 | `REPROVADO_VALIDADOR` | Entrada + EQ3 |
| Erro "cep genérico" sem cidade e UF no retorno | `CEP_GENERICO` | Entrada + EQ3 |
| Erro "cep genérico", UF e cidade validadas | `ENRIQUECIDO` | Entrada + EQ3 + cidade do CW3 (CEP e logradouro de entrada mantidos) |
| Rota por logradouro com CEP desmembrado, ou com grafia consensual mas faixa de CEP indefinida | `ENRIQUECIDO_SEM_CEP` | Entrada + EQ3 + cidade/logradouro do CW3 (CEP de entrada mantido) |
| Todas as validações passam (rota por CEP) | `ENRIQUECIDO` | Entrada + EQ3 + cidade/logradouro do CW3 |
| Todas as validações passam (rota por logradouro, CEP inexistente) | `ENRIQUECIDO` | Entrada + EQ3 + cidade/logradouro/CEP do CW3 |

Quando a saída é entrada + EQ3 e o EQ3 não preencheu nada, o resultado é igual à entrada. O status continua indicando o motivo, para métrica e análise.

## 11. Validador final

O validador é um componente separado que recebe só a entrada original e a saída proposta, sem depender da lógica do pipeline. Ele existe para pegar bugs e casos de borda que passaram pelas etapas anteriores.

Verificações:

- [ ] UF de saída idêntica à de entrada.
- [ ] Cidade de saída passa na comparação de alinhamento com a cidade de entrada.
- [ ] Logradouro de saída passa na comparação de alinhamento com o nome do logradouro de entrada (após a extração de número e complemento).
- [ ] CEP alterado somente se a entrada foi classificada como inexistente.
- [ ] Nenhum campo preenchido na entrada ficou vazio. Só foram substituídos cidade, logradouro e CEP (pelo CW3), o logradouro sem o número e o complemento extraídos, e número ou complemento por um valor que contém o original.
- [ ] Número e complemento novos (do parser ou do EQ3) ocupam campos vazios ou contêm todos os tokens do valor original.

Se qualquer verificação falhar, o validador descarta o enriquecimento do CW3 e devolve entrada + EQ3 com status `REPROVADO_VALIDADOR`. Se a falha envolver os campos do EQ3, devolve a entrada pura. Toda reprovação gera alerta, porque indica bug no pipeline.

## 12. Contrato da API

O serviço expõe um endpoint síncrono e sempre responde com um endereço utilizável: nos piores casos, a própria entrada.

**Request** — `POST /enderecos/enriquecer`

```json
{
  "pedidoId": "123456",
  "clienteId": "98765",
  "endereco": {
    "logradouro": "R das Flores 123 apto 45",
    "numero": null,
    "complemento": null,
    "bairro": "Centro",
    "cidade": "S Paulo",
    "uf": "SP",
    "cep": "01001-000"
  }
}
```

**Response**

```json
{
  "pedidoId": "123456",
  "status": "ENRIQUECIDO",
  "confianca": 0.96,
  "endereco": {
    "logradouro": "Rua das Flores",
    "numero": "123",
    "complemento": "Apto 45",
    "bairro": "Centro",
    "cidade": "São Paulo",
    "uf": "SP",
    "cep": "01001-000"
  },
  "enderecoOriginal": { "...": "cópia exata da entrada" },
  "alteracoes": [
    { "campo": "numero", "de": null, "para": "123", "fonte": "PARSER", "motivo": "EXTRAIDO_DO_LOGRADOURO" },
    { "campo": "complemento", "de": null, "para": "Apto 45", "fonte": "PARSER", "motivo": "EXTRAIDO_DO_LOGRADOURO" },
    { "campo": "cidade", "de": "S Paulo", "para": "São Paulo", "fonte": "CW3_CEP", "motivo": "GRAFIA_CANONICA" },
    { "campo": "logradouro", "de": "R das Flores 123 apto 45", "para": "Rua das Flores", "fonte": "CW3_CEP", "motivo": "GRAFIA_CANONICA" }
  ],
  "diagnostico": {
    "parser": { "status": "CONFIANTE", "motivos": [], "usouIA": false },
    "eq3": { "candidatos": 2, "camposPreenchidos": [] },
    "cw3": { "rota": "CEP", "candidatos": 1, "score": 0.96 }
  }
}
```

**Status possíveis:** `ENRIQUECIDO`, `ENRIQUECIDO_SEM_CEP`, `UF_DIVERGENTE_CEP`, `CONFLITO_CEP`, `SEM_CANDIDATO`, `AMBIGUO`, CEP\_GENERICO, `CW3_INDISPONIVEL`, `REPROVADO_VALIDADOR`. A fonte de cada alteração é `PARSER`, `IA_PARSER`, `EQ3`, `CW3_CEP` ou `CW3_LOGRADOURO`.

Número e complemento extraídos pelo parser e os vindos do EQ3 são aplicados em todas as saídas, exceto em `UF_DIVERGENTE_CEP`, que devolve a entrada sem nenhuma alteração.

## 13. Requisitos não funcionais

Nenhuma falha de dependência pode impedir a resposta: o serviço degrada para a entrada (ou entrada + EQ3) e registra o motivo.

| Tema | Requisito |
| --- | --- |
| Resiliência | Timeout e circuit breaker independentes para EQ3, CW3 e Bedrock. EQ3 fora → segue sem enriquecimento do EQ3. CW3 fora → `CW3_INDISPONIVEL`. Bedrock fora → usa o parse determinístico e registra o motivo. |
| Cache | Consultas ao CW3 por CEP em cache (os dados mudam pouco). Rota por logradouro em cache pela chave normalizada. |
| Idempotência | Mesma entrada produz a mesma saída, exceto quando o Bedrock é usado ou uma dependência muda de estado. |
| Auditoria | Log por pedido com entrada original, saídas parciais de cada etapa, candidatos avaliados com scores, decisão e status. Retenção de 1 ano. |
| Configuração | Limiares, pesos, dicionário de abreviações e lista de municípios versionados e alteráveis sem deploy. A versão usada vai no log. |
| Observabilidade | Métricas por status, por motivo de incerteza do parser, por rota do CW3, taxa de uso do Bedrock, latência por etapa e taxa de reprovação do validador. |
| Segurança e privacidade | Endereço é dado pessoal: logs com acesso restrito e descartados após 1 ano, em conformidade com a LGPD. O prompt do Bedrock não leva id do cliente nem outros dados além do logradouro. |

Latência alvo, volume de pedidos e TTL do cache ficam fora do escopo desta versão. Nesta entrega, logs e métricas vão para o console e o cache é em memória (seção 17); a retenção de 1 ano vale quando houver destino definitivo.

## 14. Calibração e casos de teste

Os limiares de similaridade são definidos com dados reais, não estimados. A métrica que manda é a taxa de alteração errada, que deve ficar próxima de zero, mesmo que isso reduza a taxa de enriquecimento.

**Base de calibração:**

1. Amostrar algumas centenas de pedidos reais, cobrindo todos os problemas da seção 1.
2. Rotular manualmente o endereço correto de cada um (o Bedrock pode sugerir rótulos, sempre revisados por uma pessoa).
3. Rodar o pipeline com vários conjuntos de limiares e medir, para cada um: alterações corretas, alterações erradas e casos que ficaram sem alteração.
4. Escolher os limiares com alteração errada próxima de zero e, entre esses, o de maior taxa de enriquecimento.

**Casos de teste obrigatórios:**

| Caso | Entrada (resumo) | Resultado esperado |
| --- | --- | --- |
| Cidade abreviada | cidade "S Paulo", CEP válido e coerente | `ENRIQUECIDO`, cidade "São Paulo" |
| Número no logradouro | "R das Flores 123 apto 45", número vazio | Número 123, complemento Apto 45, mesmo sem enriquecimento do CW3 |
| Número no nome da via | "Rua 25 de Março" | Nada extraído como número |
| Rodovia com km | "Rod BR-116 km 30" | Nada extraído como número |
| CEP inexistente | CEP não encontrado, logradouro/cidade/UF corretos | Rota por logradouro, CEP trocado |
| CEP com formato inválido | "1234-56", logradouro/cidade/UF corretos | Tratado como inexistente, CEP trocado |
| CEP desmembrado | CW3 retorna erro "cep desmembrado" | Rota por logradouro; cidade e logradouro padronizados, CEP mantido (`ENRIQUECIDO_SEM_CEP`) |
| CEP existente conflitante | CEP de outra rua, mesma UF | `CONFLITO_CEP`, entrada + EQ3 |
| UF divergente | CEP existe em outra UF | `UF_DIVERGENTE_CEP`, entrada pura, sem extração do parser |
| Cidade com prefixo comum | "Santa Rita" × "Santa Rita do Sapucaí" | Rejeitado no alinhamento |
| Vias numeradas | "Rua 10" × "Rua 12" | Rejeitado |
| Tipo divergente | "Rua São João" × "Av. São João" sem CEP coincidente | Rejeitado |
| Rua com várias faixas de CEP | Rua grande, número informado | CEP da faixa do número |
| Rua com várias faixas, sem número, grafia consensual | Rua grande, número vazio | `ENRIQUECIDO_SEM_CEP`: cidade e logradouro corrigidos, CEP mantido |
| Rua com várias faixas, grafia divergente | Candidatos com grafias diferentes | `AMBIGUO`, nada do CW3 aplicado |
| CEP genérico | CW3 retorna erro "cep genérico" com cidade e UF | Só a cidade enriquecida |
| CEP genérico sem dados | CW3 retorna erro "cep genérico" sem cidade e UF | `CEP_GENERICO`, entrada + EQ3 |
| EQ3 com dois números na mesma rua | Entrada sem número | Número não preenchido |
| EQ3 com número divergente | Entrada nº 45, histórico nº 123 | Complemento não aproveitado |
| EQ3 agrega complemento | Entrada "AP 12", EQ3 "APTO 12 BLOCO B" | Complemento "APTO 12 BLOCO B" |
| EQ3 contradiz complemento | Entrada "APTO 12", EQ3 "APTO 31" | Complemento da entrada mantido |
| EQ3 acrescenta sufixo ao número | Entrada "123", EQ3 "123 A" | Número "123 A", motivo `NUMERO_SUFIXO_EQ3` |
| Parser incerto | "R DAS FLORES 123 FDS CASA 2" | Chama o Bedrock; resposta validada |
| IA inventa token | Bedrock devolve palavra ausente na entrada | Resposta descartada, segue com o parse determinístico |
| Bedrock fora do ar | Timeout | Segue com o parse determinístico |
| CW3 fora do ar | Timeout | `CW3_INDISPONIVEL`, entrada + EQ3 |

## 15. Rollout, decisões e próximos passos

O serviço entra em produção primeiro em shadow mode, e cada fase só avança quando a anterior mostra taxa de alteração errada próxima de zero.

**Fases:**

1. **Fase 1 — shadow mode.** Pipeline completo, já com o Bedrock no parsing, calculando e registrando sem alterar os pedidos. Medir status, motivos de incerteza do parser, taxa de uso e de descarte das respostas do Bedrock e divergências contra a base rotulada.
2. **Fase 2 — ativação.** Aplicar os resultados nos pedidos. Acompanhar a taxa de insucesso de entrega e devolução por status.
3. **Fase 3 (opcional) — nova tentativa com IA** após falha no CW3, se os dados justificarem.

**Decisões tomadas:**

| Questão | Decisão |
| --- | --- |
| Número e complemento na saída entrada + EQ3 | Aplicados (parser e EQ3), inclusive valores que agregam informação a um campo já preenchido |
| CEP com formato inválido | Conta como inexistente e pode ser trocado |
| Grafia consensual sem faixa de CEP definida | Corrige cidade e logradouro se passar na similaridade e mantém o CEP (`ENRIQUECIDO_SEM_CEP`) |
| `UF_DIVERGENTE_CEP` e `CONFLITO_CEP` | Só métrica, sem revisão manual |
| Latência alvo, volume e TTL do cache | Fora do escopo desta versão |
| Retenção dos logs de auditoria | 1 ano |
| IA no parser do logradouro | Entra na primeira versão |

**Próximos passos:**

- [ ] Montar a base de calibração rotulada a partir do modelo massa\_teste.csv. Preencher as restrições técnicas (seção 17) e obter os contratos reais do EQ3 e do CW3.
- [ ] Revisar o dicionário de abreviações (rascunho em abreviacoes.csv) e baixar a lista de municípios do IBGE.
- [ ] Confirmar com o time do CW3 se a resposta com erro "cep genérico" traz cidade e UF.
- [ ] Escolher o modelo do Bedrock e escrever o prompt, o schema JSON e a trava de fidelidade de tokens.
- [ ] Implementar parser, comparador e validador final com os casos de teste da seção 14.

## 16. Contratos das APIs externas (provisórios)

Os contratos abaixo são genéricos e servem para os mocks e para os testes até chegarem os contratos reais. Por isso, cada integração deve ficar isolada num adaptador que converte a resposta externa para um modelo interno único. Trocar pelo contrato real deve exigir mudança só no adaptador.

**Modelo interno de endereço** (o que todos os adaptadores devolvem):

```json
{
  "logradouro": "string",
  "numero": "string | null",
  "complemento": "string | null",
  "bairro": "string | null",
  "cidade": "string",
  "uf": "string (2 letras)",
  "cep": "string (8 dígitos) | null"
}
```

### EQ3 — endereços do cliente

`GET /clientes/{clienteId}/enderecos`

```json
{
  "clienteId": "98765",
  "enderecos": [
    {
      "id": "end-001",
      "logradouro": "Rua das Flores",
      "numero": "123",
      "complemento": "Apto 45 Bloco B",
      "bairro": "Centro",
      "cidade": "São Paulo",
      "uf": "SP",
      "cep": "01001000",
      "dataCadastro": "2025-03-10T14:22:00Z"
    }
  ]
}
```

| Resposta | Tratamento |
| --- | --- |
| 200 com lista vazia | Sem enriquecimento do EQ3 |
| 404 (cliente sem endereços) | Sem enriquecimento do EQ3 |
| 5xx ou timeout | Segue sem EQ3 e registra o motivo |

### CW3 — busca por CEP

`GET /cep/{cep}`

Sucesso:

```json
{
  "cep": "01001000",
  "tipoLogradouro": "Rua",
  "logradouro": "Rua das Flores",
  "bairro": "Centro",
  "cidade": "São Paulo",
  "uf": "SP",
  "erro": null
}
```

CEP desmembrado e CEP genérico (cidade e UF podem vir ou não; a confirmar com o time do CW3):

```json
{ "cep": "01001000", "erro": "CEP_DESMEMBRADO" }
```

```json
{ "cep": "99999000", "cidade": "Município Exemplo", "uf": "MG", "logradouro": null, "erro": "CEP_GENERICO" }
```

| Resposta | Tratamento |
| --- | --- |
| 200 com `erro` nulo | Validações da rota por CEP |
| 200 com `erro = CEP_DESMEMBRADO` | Rota por logradouro, CEP mantido |
| 200 com `erro = CEP_GENERICO` | Passo 3 da rota por CEP |
| 404 | CEP inexistente → rota por logradouro, pode trocar o CEP |
| 5xx ou timeout | `CW3_INDISPONIVEL` |

### CW3 — busca por logradouro

`GET /enderecos?logradouro={nome}&cidade={cidade}&uf={uf}`

```json
{
  "resultados": [
    {
      "cep": "01001000",
      "tipoLogradouro": "Rua",
      "logradouro": "Rua das Flores",
      "bairro": "Centro",
      "cidade": "São Paulo",
      "uf": "SP",
      "faixa": { "inicio": 1, "fim": 999, "lado": "IMPAR" }
    },
    {
      "cep": "01001001",
      "tipoLogradouro": "Rua",
      "logradouro": "Rua das Flores",
      "bairro": "Centro",
      "cidade": "São Paulo",
      "uf": "SP",
      "faixa": { "inicio": 2, "fim": 1000, "lado": "PAR" }
    }
  ]
}
```

`faixa` é nula quando o CEP cobre a rua inteira. `lado` aceita `PAR`, `IMPAR` ou `AMBOS`. Lista vazia → `SEM_CANDIDATO`; 5xx ou timeout → `CW3_INDISPONIVEL`.

### Bedrock — parsing do logradouro

Entrada enviada ao modelo (sem id do cliente nem outros dados pessoais):

```json
{
  "logradouro": "R DAS FLORES 123 FDS CASA 2",
  "numero": null,
  "complemento": null,
  "motivosIncerteza": ["TOKEN_NAO_CLASSIFICADO:FDS"]
}
```

Saída exigida (JSON puro, validado por schema):

```json
{
  "tipo": "RUA",
  "nome": "DAS FLORES",
  "numero": "123",
  "complemento": "FUNDOS CASA 2"
}
```

Rascunho do prompt de sistema:

```text
Você organiza logradouros brasileiros em campos. Responda somente com JSON
no formato {"tipo", "nome", "numero", "complemento"}. Use apenas palavras
que existem no texto de entrada; você pode expandir abreviações (R→RUA,
FDS→FUNDOS), mas nunca acrescentar, corrigir ou inventar nomes, números ou
cidades. Números que fazem parte do nome da via ("25 DE MARCO", "KM 30",
"BR-116") ficam no nome. Se não for possível separar com segurança, devolva
o texto inteiro em "nome" e os demais campos nulos.
```

A resposta que falhar no schema ou na trava de fidelidade (seção 6) é descartada.

## 17. Restrições técnicas

Esta entrega é a aplicação completa, rodando localmente. Infraestrutura, CI/CD e destino definitivo dos logs ficam fora do escopo por enquanto; o destino futuro é a AWS.

| Tema | Decisão |
| --- | --- |
| Linguagem | Python (versão a definir) |
| Framework da API | FastAPI, com Pydantic para validar o schema da resposta do Bedrock e os modelos dos adaptadores |
| Modelo de IA | Claude Sonnet 5.5 via Bedrock; região a definir (confirmar a disponibilidade do modelo nela) |
| Execução | Local, subindo com um único comando; sem infraestrutura nesta entrega (destino futuro: AWS ECS) |
| EQ3 e CW3 locais | Mocks baseados nos contratos da seção 16 e nos dados da massa de teste, trocáveis pelas APIs reais por configuração |
| Bedrock local | Duas opções por configuração: chamada real (credenciais AWS da máquina) ou desligado, usando só o parser determinístico |
| Configuração | Arquivos locais versionados (limiares, pesos, abreviações, municípios), carregados na inicialização |
| Cache | Em memória |
| Logs e métricas | Console, em JSON estruturado (um evento de auditoria por pedido) |
| CI/CD | Fora do escopo |
| Testes e lint | pytest para testes; ruff para lint (sugestão) |

## 18. Insumos para a equipe

Três arquivos acompanham a spec. Os dois primeiros são carregados como configuração versionada, nunca escritos no código.

| Insumo | Arquivo | Situação |
| --- | --- | --- |
| Dicionário de abreviações | `abreviacoes.csv` | Rascunho entregue; revisar e completar com os casos reais |
| Lista de municípios por UF | `municipios.csv` (nome, UF) | Baixar da API pública de localidades do IBGE; o código IBGE pode ser descartado |
| Massa de teste rotulada | `massa_teste.csv` | Modelo entregue com exemplos fictícios; completar com 50 a 100 casos reais anonimizados |

**Dicionário de abreviações.** Cada linha tem a abreviação, a expansão, o contexto em que vale e uma marca de ambiguidade. O contexto evita expansões erradas:

- `TIPO_VIA`: só vale como primeiro token do logradouro ("AL" é ALAMEDA ali, mas pode ser a UF de Alagoas em outro lugar).
- `NOME`: vale dentro do nome da via ou da cidade ("STA" → SANTA).
- `COMPLEMENTO`: só vale no complemento ("AP" → APARTAMENTO).
- `AMBIGUA = sim`: o token tem mais de um significado possível ("N" pode ser NOSSA, NORTE ou NÚMERO). O parser só expande quando o contexto resolve; caso contrário, marca `ABREVIACAO_DESCONHECIDA`.

**Massa de teste.** Cada linha traz a entrada, as respostas simuladas do EQ3 e do CW3 (referência para os mocks) e a saída esperada. A massa rotulada é a verdade do tester: o código se ajusta a ela, nunca o contrário.

## 19. Critérios de aceite

A entrega só é aceita com zero alterações erradas na massa rotulada. Taxa de enriquecimento baixa é aceitável; endereço errado não é.

**Corretude**

- [ ] Todos os casos de teste da seção 14 passam.
- [ ] Zero alterações erradas na massa rotulada (seção 18).
- [ ] As invariantes da seção 3 são verificadas por testes baseados em propriedades (property-based), com milhares de endereços gerados aleatoriamente: UF nunca muda, CEP só muda quando inexistente, nenhum campo preenchido fica vazio.
- [ ] O validador final tem testes próprios, independentes do pipeline, incluindo saídas propositalmente inválidas.

**Robustez**

- [ ] Com EQ3, CW3 ou Bedrock fora do ar, a API responde com o fallback e o status corretos, sem erro para o chamador.
- [ ] Respostas do Bedrock fora do schema ou com tokens inventados são descartadas.
- [ ] Trocar o contrato provisório pelo real de uma API exige mudança só no adaptador dela.

**Operação**

- [ ] Limiares, pesos e dicionários alteráveis por arquivo de configuração, sem mudar código, com a versão usada registrada em cada log.
- [ ] Log de auditoria completo por pedido (entrada, etapas, candidatos, scores, decisão) e métricas da seção 13, emitidos no console em JSON estruturado.
- [ ] A aplicação sobe localmente com um único comando, usando os mocks de EQ3 e CW3 e o Bedrock real ou desligado, conforme o README.
- [ ] Chave de configuração para operar em shadow mode (calcula e registra, sem devolver o endereço alterado).
- [ ] O prompt enviado ao Bedrock não contém id do cliente nem outros dados além do logradouro, número e complemento.

## 20. Guia para a equipe de agentes

Esta seção vale como instrução para todos os agentes. A regra principal: na dúvida, o agente escolhe a opção mais conservadora e registra a decisão; nunca supõe em silêncio.

**Regras para todos os agentes**

1. As regras da seção 3 são inegociáveis. Nenhuma otimização, refatoração ou "melhoria" pode relaxá-las.
2. Ambiguidade ou lacuna na spec é registrada no arquivo DECISOES.md do repositório, com a opção adotada (sempre a mais conservadora) e o motivo. O responsável humano revisa esse arquivo na entrega.
3. Limiares (0,92, 0,80, 0,05), pesos e dicionários são configuração, nunca constantes no código.
4. Os contratos da seção 16 são provisórios: toda integração passa por um adaptador.
5. Nenhum dado real de cliente sai do ambiente autorizado. A massa de teste usa dados anonimizados.
6. Na dúvida entre enriquecer e não enriquecer, o código não enriquece.

**Papéis**

| Agente | Responsabilidade | Não faz |
| --- | --- | --- |
| PM | Quebra a spec em tarefas na ordem de construção abaixo, mantém o DECISOES.md e a rastreabilidade tarefa → seção da spec | Decidir regra de negócio sem o responsável humano |
| Arquiteto | Define módulos, adaptadores, modelo interno e configuração; registra as decisões no DECISOES.md | Alterar regras da seção 3 ou contratos sem aprovação |
| Desenvolvedor | Implementa cada etapa como módulo isolado e testável | Ajustar testes ou massa rotulada para o código passar |
| Tester | Escreve os testes a partir da spec e da massa rotulada, antes de ver o código; mantém os testes baseados em propriedades | Derivar o resultado esperado do comportamento do código |
| Revisor | Revisa cada entrega contra as regras da seção 3, o validador final e os critérios de aceite, além da qualidade do código | Aprovar entrega com teste desativado ou limiar alterado no código |

**Entrega**

A solução é entregue completa, de uma vez, rodando localmente. Internamente, a equipe constrói nesta ordem, porque cada parte depende da anterior e é testada antes de seguir:

1. Arquitetura (módulos, adaptadores, modelo interno, configuração), registrada em DECISOES.md.
2. Parser e normalização, com o dicionário de abreviações e os motivos de incerteza.
3. Comparador de similaridade (alinhamento de tokens e score).
4. Validador final, com testes próprios.
5. Adaptador e mock do EQ3.
6. Adaptador e mock do CW3 (as duas rotas e os erros).
7. Fallback com o Bedrock, com schema, trava de fidelidade e modo desligado.
8. API FastAPI, logs em JSON no console e chave de shadow mode.

A entrega deve conter:

- Código da aplicação e dos mocks.
- Testes (casos da seção 14, massa de teste e testes baseados em propriedades), todos passando.
- README com como instalar, configurar e rodar localmente, como rodar os testes e como trocar mocks por APIs reais.
- DECISOES.md com as decisões tomadas e as suposições feitas.
- Relatório do revisor confirmando cada critério de aceite da seção 19.

O revisor só libera a entrega quando todos os critérios de aceite passam. O responsável humano faz a revisão final, com atenção especial ao parser, ao comparador e ao DECISOES.md, que é onde mora o risco de endereço errado.
