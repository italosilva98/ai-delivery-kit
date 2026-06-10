---
name: sdd-implement
description: "Implementa backend Kotlin seguindo Mini Spec, plano e tasks"
argument-hint: "Informe spec, plan, tasks e arquivos de referência"
---

# SDD Implement — Backend Kotlin

Você é responsável por implementar a feature seguindo Mini Spec, Technical Plan e Tasks.

## Entradas

Usar:
- `01-definition.md`;
- `02-plan.md`;
- `03-tasks.md`;
- `docs/ai/architecture-lite.md`;
- arquivos reais de referência indicados pelo usuário.

Não analisar o projeto inteiro sem necessidade.

## Regras

- Implementar apenas o escopo definido.
- Não alterar arquivos fora do plano sem justificar.
- Não criar arquitetura nova sem necessidade.
- Não introduzir biblioteca nova sem justificar.
- Não colocar regra de negócio em controller.
- Não usar `!!`.
- Não usar `println`.
- Não logar dados sensíveis.
- Não alterar contrato público sem previsão.
- Criar ou atualizar testes quando houver mudança de comportamento.

## Ordem sugerida

1. DTOs/contratos.
2. Controller.
3. UseCase/Service.
4. Domain model/exception.
5. Gateway/Client/Repository.
6. Mapper.
7. Error handler, se necessário.
8. Testes.
9. Revisão final.

## Padrões

- Usar Kotlin idiomático.
- Preferir `val`.
- Usar nomes expressivos.
- Manter funções pequenas.
- Preservar estilo existente.
- Priorizar menor alteração possível.

## Testes

Criar ou atualizar testes para:
- sucesso;
- validação;
- regra de negócio;
- erro de integração;
- contrato HTTP.

## Divergências

Se a Mini Spec e o Plan divergirem:
- parar a parte conflitante;
- apontar o conflito;
- sugerir ajuste.

Se o código existente divergir da arquitetura documentada:
- seguir o padrão real do código;
- informar no resumo.

## Resposta final

Responder neste formato:

```md
## Implementação concluída

### Arquivos criados

- `path/File.kt` — ...

### Arquivos alterados

- `path/File.kt` — ...

### O que foi implementado

- ...

### Testes

- `path/FileTest.kt` — ...

### Validações

- [ ] Contrato conforme spec
- [ ] Regra no use case/service
- [ ] Controller sem regra de negócio
- [ ] Erros tratados
- [ ] Logs seguros
- [ ] Testes adicionados/ajustados

### Comandos sugeridos

```bash
./gradlew test
./gradlew clean build
```

### Pontos de atenção

- Nenhum.

### Pendências

- Nenhuma.
```
