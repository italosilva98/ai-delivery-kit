# ROLE & OBJECTIVE
Você é um Staff Software Engineer especialista em AI Agents, Developer Platforms e Full-Stack TypeScript.
Sua missão é implementar, do zero até o estado 100% funcional na máquina local do usuário, o **ClawPortal / Agent Orchestrator**: um Internal Developer Portal (IDP) corporativo de squads autônomas de engenharia de software (estilo Jarvis-Claw).

Não gere mocks fictícios nem código com `// TODO`. Crie a estrutura completa, integração Git local (`simple-git`), execução de agentes via **Claude CLI** e **Devin CLI** instalados na máquina do usuário, banco de dados persistente com Prisma + SQLite, streaming em tempo real (Server-Sent Events) e a interface organizada em **Menus Separados Horizontais** com explorador de arquivos e visualizador de código dedicados.

---

# 1. VISÃO DO PRODUTO & ARQUITETURA DE MENUS (PADRÃO GROK / CLAWPORTAL)
O sistema possui duas telas principais:

### A. Catálogo Global de Aplicações
- Grid com os cards de repositórios conectados, badge de status (`Idle`, `Working`, `PR Open`), stacks tecnológicas detectadas e total de tarefas entregues.
- Barra de busca rápida em tempo real por nome ou tecnologia.
- Modal de importação (via URL do GitHub) ou criação de repositório novo do zero.

### B. Workspace do Repositório (Menus Horizontais)
No topo do workspace, exibir o botão `← Voltar ao Catálogo de Aplicações`, nome do projeto, badge de status (`Idle`), URL do repositório no GitHub, branch ativa (`main`) e atalhos rápidos (`Pedir Alteração à Squad`, `Abrir no GitHub`, `Excluir`).

A navegação dentro do repositório é dividida em **5 abas horizontais independentes**:

1. **`Histórico & Linha do Tempo (N)`:**
   - Lista cronológica de todas as sprints executadas pela Squad.
   - Cada card exibe: tipo da tarefa (`FEATURE`, `BUGFIX`, `REFACTOR`, `TESTS`), status (`COMPLETED`, `RUNNING`, `FAILED`), resumo da entrega (`Entrega: Módulo ... implementado com sucesso`), branch gerada, commit hash e link para o Pull Request.

2. **`Comando para a Squad`:**
   - **Coluna Esquerda (Instrução):** Seletor de objetivo em botões/chips (`Nova Feature`, `Bugfix`, `Refatoração`, `Testes`), campo de texto detalhado com contador de caracteres (`0/8000`), chips de sugestões rápidas de prompt e botão de disparo `Disparar Squad (Claude CLI & Devin)`.
   - **Coluna Direita (Pipeline & Terminal em Tempo Real):** Indicadores visuais dos 4 nós da Squad (`PM`, `Arquiteto`, `Coder`, `QA`), badge de status do container (`STANDBY`, `PROVISIONANDO`, `EXECUTANDO`, `CONCLUÍDO`) e janela de terminal com streaming SSE dos logs e do comando CLI executado.

3. **`Agentes da Squad`:**
   - Grid em 4 colunas apresentando os agentes da equipe:
     - **Agente PM:** Planejamento e checklist técnico (sem ferramentas de escrita).
     - **Agente Arquiteto:** Mapeamento de arquivos (`list_directory`, `read_file`, `search_code`).
     - **Agente Coder:** Implementação de código (`read_file`, `write_file`, `replace_in_file`) acionando Claude CLI ou Devin CLI.
     - **Agente QA:** Validação de build/testes (`run_command` com `npm run build`, `pytest`, `tsc`) e abertura de Pull Request.

4. **`Memória & Arquitetura`:**
   - Visualizador e editor dedicado do arquivo `AI_CONTEXT.md` com persistência direta no repositório local e no banco SQLite.

5. **`Arquivos & Diff`:**
   - **Seção Superior (Diff da última tarefa):** Exibe o `git diff` gerado pela última sprint aprovada pelo QA.
   - **Seção Inferior (Explorador de Arquivos):** Layout split com **Árvore de Diretórios Hierárquica** (pastas expansíveis como `banco/`, `servidor/`, `tela/` e arquivos raiz) à esquerda e **Visualizador de Conteúdo** à direita exibindo o código-fonte real com tamanho em bytes (`643 B`).

---

# 2. MOTORES DE EXECUÇÃO: CLAUDE CLI & DEVIN CLI
A Squad Dev deve priorizar as ferramentas de linha de comando já autenticadas e instaladas no sistema operacional do usuário:

1. **Claude CLI (`claude`):**
   - Utilizado primordialmente para raciocínio analítico, planejamento e edição de arquivos via flag não-interativa:
     ```bash
     claude -p "<prompt com contexto do repo>"
     ```
   - Executado dentro do diretório do workspace (`cwd: workspacePath`).

2. **Devin CLI (`devin`):**
   - Utilizado para tarefas de implementação profunda, correções de bugs complexos e execuções autônomas que demandam testes e navegação extensiva:
     ```bash
     devin run "<prompt de implementação>"
     ```

3. **Configuração Flexível (`.env`):**
   - `DEFAULT_CLI_ENGINE="claude"` (opções: `"claude"`, `"devin"`, `"hybrid"` onde PM/Arquiteto rodam via Claude e Coder/QA rodam via Devin).
   - `USE_CLAUDE_CLI="true"`
   - `USE_DEVIN_CLI="true"`
   - Suporte opcional a APIs (`ANTHROPIC_API_KEY`, `OPENAI_API_KEY`) e fallback para o **Modo Autônomo Local com Git real** caso os CLIs ou chaves estejam indisponíveis no momento do teste.

---

# 3. SCHEMA DO BANCO DE DADOS (`prisma/schema.prisma`)
Mantenha a persistência em SQLite (`file:./dev.db`):

```prisma
datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model Project {
  id              String   @id @default(cuid())
  name            String
  description     String?
  githubUrl       String
  defaultBranch   String   @default("main")
  localPath       String
  techStack       String   @default("[]")
  status          String   @default("IDLE") // IDLE, CLONING, WORKING, PR_OPEN, ERROR
  aiContext       String   @default("")
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  tasks           Task[]
}

model Task {
  id              String     @id @default(cuid())
  projectId       String
  project         Project    @relation(fields: [projectId], references: [id], onDelete: Cascade)
  title           String
  prompt          String
  taskType        String     @default("FEATURE") // FEATURE, BUGFIX, REFACTOR, TESTS
  status          String     @default("QUEUED")  // QUEUED, RUNNING, COMPLETED, FAILED
  branchName      String?
  commitHash      String?
  prUrl           String?
  summary         String?
  filesModified   String     @default("[]")
  diffContent     String?
  createdAt       DateTime   @default(now())
  completedAt     DateTime?
  logs            AgentLog[]
}

model AgentLog {
  id              String   @id @default(cuid())
  taskId          String
  task            Task     @relation(fields: [taskId], references: [id], onDelete: Cascade)
  sender          String   @default("AGENT") // AGENT, USER, TOOL, SYSTEM
  agentRole       String   // SYSTEM, PM, ARCHITECT, CODER, QA
  messageType     String   @default("CHAT")  // THOUGHT, ACTION, RESULT, CHAT
  message         String
  toolUsed        String?
  detail          String?
  createdAt       DateTime @default(now())
}
```

---

# 4. IMPLEMENTAÇÃO DO ORQUESTRADOR (`src/lib/squad/orchestrator.ts`) E TOOLS (`tools.ts`)
- No `tools.ts`, implemente os executores de linha de comando:
  - `runClaudeCLI(prompt: string, workspacePath: string)`
  - `runDevinCLI(prompt: string, workspacePath: string)`
  - Ferramentas nativas do Node.js: `readFile`, `writeFile`, `runCommand` (com `child_process`).
- No `orchestrator.ts`, orquestre o pipeline emitindo logs tipados para cada canal de agente:
  1. **PM:** Emite `THOUGHT` detalhando a análise do pedido e em seguida emite `CHAT` com o escopo e diretrizes.
  2. **ARCHITECT:** Inspeciona arquivos, emite `THOUGHT` e `ACTION` (tool: `search_code`), finalizando com o plano de arquivos para o Coder.
  3. **CODER:** Aciona o **Claude CLI** ou **Devin CLI** para aplicar as mudanças de código no disco. Se ambos os CLIs falharem ou não estiverem instalados, aciona a IA configurada ou o fallback autônomo local. Emite `ACTION` (tool: `write_file`) e atualiza o `AI_CONTEXT.md`.
  4. **QA:** Analisa o `git diff`, executa validação de sintaxe e faz o `git commit` gerando o hash e a branch de entrega.

---

# 5. ROTAS DE API NECESSÁRIAS
- `GET /api/projects`: Lista projetos com contagem de tasks e dados essenciais.
- `POST /api/projects`: Importa do GitHub via `git clone` para `./workspaces/<nome>` ou inicializa novo repositório local.
- `GET /api/projects/[id]`: Retorna o projeto com todo o histórico de tasks e logs de agentes.
- `PATCH /api/projects/[id]`: Atualiza o `AI_CONTEXT.md` no banco e no disco.
- `GET /api/projects/[id]/files`: Navega recursivamente pelos arquivos e lê conteúdo para o explorador de arquivos (árvore e viewer).
- `POST /api/tasks`: Cria nova sprint e aciona a Squad.
- `GET /api/tasks/[taskId]/stream`: Stream Server-Sent Events (SSE) atualizando logs e nós dos agentes em tempo real.

---

# 6. INSTRUÇÕES DE EXECUÇÃO IMEDIATA
Execute todos os passos sem parar para pedir aprovação intermediária:
1. Monte o projeto Next.js 14+ com Tailwind CSS e TypeScript.
2. Instale: `@prisma/client`, `prisma`, `simple-git`, `@octokit/rest`, `lucide-react`, `ai`, `@ai-sdk/anthropic`, `@ai-sdk/openai`, `zod`, `clsx`, `tailwind-merge`.
3. Configure o schema Prisma, execute `npx prisma db push` e crie o seed com um repositório real em `./workspaces/lista-tarefas`.
4. Implemente as bibliotecas, orquestrador com suporte a Claude CLI e Devin CLI, e todas as rotas da API.
5. Construa a tela do Dashboard e a tela do Workspace exatamente no padrão visual das imagens fornecidas (Layout com os 5 menus horizontais: Histórico, Comando, Agentes, Memória e Arquivos & Diff com árvore hierárquica e visualizador).
6. Rode a verificação de compilação com `npx tsc --noEmit` e inicie o servidor dev.