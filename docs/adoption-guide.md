# Guia prático — adotando o Guild no seu time

Este guia leva você do zero até um repositório com o Guild funcionando: CLI
instalado, skills disponíveis no Claude Code, agentes derivados e o board
conectado.

O Guild roda no **Claude Code**. É lá que as guardas existem: a mesma skill
aberta em outra ferramenta roda sem nenhuma delas.

Quase tudo é **guiado por uma skill** dentro do Claude Code. O que você faz à mão
é só a preparação da máquina, uma vez:

| Etapa | Quem faz | Quando |
|---|---|---|
| **1. Preparar a máquina** | você, no terminal (~10 min) | uma vez por máquina |
| **2. Configurar o repositório** | a skill `/guild-init` | uma vez por repositório |
| **Depois** | a skill `/guild-upgrade` | a cada versão nova da plataforma |

Passos marcados com **[Azure Boards]** só valem para o time que controla o
trabalho num board do Azure DevOps. Com o board em modo local, pule-os.

---

## O que é o Guild, em 30 segundos

| Peça | O que é |
|---|---|
| **Skills** | Processos de engenharia que a IA executa (`/guild-init`, `/codebase-analysis`, …) |
| **Agentes** | Um agente dev por camada da arquitetura, cada um com sua fronteira de escrita |
| **CLI `rbb`** | A parte mecânica: board, sincronização, perfil, verificação. As skills o chamam por você |
| **Guardas** | Hooks do Claude Code que **bloqueiam de verdade** escrita fora da fronteira do agente |

O catálogo completo — cada skill, comando e agente, com o que recebe e o que gera
— está em <https://rabberto.github.io/rbb-guild/>. Ele mostra a versão mais nova
da plataforma; a instalada na sua máquina responde em `rbb version`. Sentiu falta
de algo do catálogo? Sua versão pode ser mais antiga: rode `/guild-upgrade`.

---

## Etapa 1 — Preparar a máquina

Nada disto pode ser feito por skill: ou é o que faz as skills existirem (o
CLI), ou é segredo (tokens), que as skills nunca pedem nem gravam.

### 1.1 Pré-requisitos

| Ferramenta | Onde instalar |
|---|---|
| **Node.js** (LTS), que traz o **npm** | <https://nodejs.org/en/download> |
| **Git** (no Windows, já traz o Git Bash) | <https://git-scm.com/downloads> |
| **Claude Code** (CLI ou extensão do VS Code) | <https://code.claude.com/docs/en/setup> |

**Windows:** as três pelo `winget`, no **PowerShell** (o Git Bash só existe
depois que o Git for instalado):

```powershell
winget install --id OpenJS.NodeJS.LTS -e
winget install --id Git.Git -e
winget install --id Anthropic.ClaudeCode -e
```

Prefere o Claude Code dentro do VS Code? Instale a extensão **Claude Code** pelo
marketplace do VS Code.

E o acesso ao **feed** do CLI — é o único acesso necessário: skills, agentes e
templates vêm dentro do pacote. Quem distribui o Guild para o seu time libera o
acesso e passa a URL do feed.

Depois de instalar, **feche e abra o terminal**: só um terminal novo enxerga o
`node`, o `npm`, o `git` e o `claude` recém-instalados.

> **Windows: daqui em diante, use o Git Bash.** No PowerShell, o comando `bash` é
> o do WSL — outro sistema, sem o seu Node. Comandos e gates do Guild rodados por
> ali falham com mensagens enganosas.

### 1.2 Autenticar o npm no feed do Guild

O CLI é distribuído por um feed npm do Azure Artifacts. Grave a URL do feed, que
vem de quem liberou o seu acesso, trocando `<url-do-feed>`:

```bash
npm config set @rbbsolucoes:registry <url-do-feed>
```

O `npm config set` grava no arquivo de configuração que o npm de fato lê
(`npm config get userconfig` mostra qual), seja qual for o seu terminal.

Agora a credencial. **Windows:**

```bash
npm install -g vsts-npm-auth
vsts-npm-auth -config "$(npm config get userconfig)"
```

**Linux / macOS / WSL:** gere um PAT em *User settings → Personal Access
Tokens* com escopo **Packaging: Read** e adicione-o ao arquivo que
`npm config get userconfig` mostra, no formato de *Artifacts → Connect to feed →
npm → Other*.

Teste:

```bash
npm view @rbbsolucoes/rbb-engineering-cli version
```

Respondeu uma versão? Autenticação OK.

### 1.3 Instalar o CLI

```bash
npm install -g @rbbsolucoes/rbb-engineering-cli
rbb version
rbb configure
```

O `rbb configure`, rodado em qualquer pasta, é o passo da máquina: mostra o feed que
o CLI encontrou no seu npm e instala as skills de entrada (`/guild-init`,
`/guild-help` e `/guild-upgrade`) em `~/.claude/skills`. Daqui em diante, todo
repositório aberto no Claude Code já tem as três. Com uma sessão do Claude Code já
aberta, rode `/reload-skills` (ou reabra a sessão).

### 1.4 PAT do board **[Azure Boards]**

O Guild lê o PAT de uma **variável de ambiente** — ele nunca é gravado em
arquivo. Gere um PAT com escopo **Work Items: Read & Write** e defina a
variável. O nome é escolha do time: o `/guild-init` pergunta qual é e grava só
o **nome** em `.rbb/boards.json`. Na Rbb, `AZURE_TEAM_ADM_EXT_PAT`:

```bash
# Windows (permanente — abra um terminal novo depois)
setx AZURE_TEAM_ADM_EXT_PAT "<seu-pat>"

# bash (Linux/macOS) — coloque no ~/.bashrc ou ~/.zshrc
export AZURE_TEAM_ADM_EXT_PAT=<seu-pat>
```

> Se você já usa o `az` CLI, `AZURE_DEVOPS_EXT_PAT` também é aceita.

### 1.5 Ajuste de git — se o time integra numa `develop`

Cada time tem o seu fluxo de branch. Este ajuste vale para quem cria as
branches de trabalho a partir de `origin/develop`:

```bash
git config --global branch.autoSetupMerge simple
```

Sem isso, criar uma branch a partir de `origin/develop` grava o upstream como
`origin/develop`, e o `git push` passa a sugerir empurrar direto para a
`develop`. Com `simple`, use `git push -u origin HEAD` na primeira vez.

---

## Etapa 2 — Configurar o repositório

Nada aqui é à mão. Abra o Claude Code na raiz do repositório e execute:

```
/guild-init
```

A skill lê o que já existe, mostra um **plano** antes de agir e conduz o resto.
Você responde às perguntas; ela explica cada escolha.

| # | O que acontece |
|---|---|
| 1 | Confere a configuração e traz skills e agentes para o repositório |
| 2 | Conecta o board — modo (Azure Boards ou local); em Azure Boards, também organização, projeto, nome da variável do PAT e mapeamento de estados, e testa a conexão |
| 3 | Gera o perfil do projeto (`.rbb/project.yaml`) com `codebase-analysis`, **se não existir** |
| 4 | Atualiza no perfil o que é verificável (dependências) |
| 5 | Deriva um agente por camada **e instala os hooks de guarda** |
| 6 | Verifica a máquina e o projeto, inclusive se a guarda está viva, e não só registrada — vermelho deixa a adoção como pendente |

> **Mapeamento de estados do board [Azure Boards]:** responda com os estados
> **reais** do processo do seu board, não com o preset "mais parecido". Estado
> que não existe no board faz a guarda recusar transições legítimas depois.

Parou no meio? Rode `/guild-init` de novo — ela confere o estado antes de cada
passo e não destrói configuração existente.

### Trabalho em aberto já existente **[Azure Boards]**

Só se o time está passando do board **local** para o **Azure Boards** e o
repositório já tem backlog, planos ou documentos de trabalho em `docs/`. Rode
**uma vez**:

```
/board-adoption
```

Ela leva esse trabalho para o board, para não existirem dois controles em
paralelo. Com o board em modo local, não há o que fazer aqui.

### Precisa trabalhar sem a guarda?

É o único comando do dia a dia que nenhuma skill faz por você:

```bash
rbb guard off     # desliga
rbb guard on      # religa
```

O `rbb guard off` desliga **só a guarda de escrita**, **só para quem roda** e
**só naquela máquina**. Os hooks continuam registrados no `.claude/settings.json`
do time, e a guarda segue valendo para todos os outros. O desligamento fica
registrado no ledger, e o `rbb guard status` diz desde quando está desligada.

### Versionar

Commite o que a adoção gerou (`.claude/`, `.gitignore`, `.rbb/project.yaml`,
`.rbb/boards.json` e, se existir, `.rbb/config.json`) **numa branch** e abra PR,
pelo fluxo de branch do seu time. O `.gitignore` já vem com as regras certas: o
resto de `.rbb/` é estado local e fica fora do git.

---

## Depois — mantendo em dia

Saiu versão nova da plataforma, ou algo do Guild falhou por configuração? Abra o
Claude Code no repositório e rode:

```
/guild-upgrade
```

Ela faz o upgrade inteiro, na ordem certa: atualiza o CLI, as skills e os agentes
do repositório e as skills de entrada da máquina, o perfil, o time e os hooks, e
termina verificando tudo. O que faltar configurar ela pergunta no chat em vez de
deixar o comando falhar.

Ela não pede tokens: se faltar autenticação no feed, ela mostra o que fazer e
você refaz o passo 1.2.

---

## Checklist final

- [ ] `rbb version` responde
- [ ] `/guild-init` terminou sem pendência, com a verificação final verde
- [ ] `.rbb/project.yaml` existe e descreve as camadas reais do código
- [ ] Agentes em `.claude/agents/`, um por camada
- [ ] O que a adoção gerou está numa branch, com PR aberto

---

## Dia a dia

| Quero… | No Claude Code |
|---|---|
| Ver todas as skills, comandos e agentes | <https://rabberto.github.io/rbb-guild/> |
| Saber qual skill serve para o meu caso, ou o estado do Guild neste repositório | `/guild-help` |
| Atualizar a plataforma inteira (CLI, skills, agentes, hooks) | `/guild-upgrade` |
| Registrar um problema ou uma sugestão | `/feedback <o que aconteceu>` |

---

## Problemas comuns

| Sintoma | Causa provável | O que fazer |
|---|---|---|
| `npm` responde **401/403** ao instalar | npm não autenticado no feed | Refaça o passo 1.2 |
| `npm` responde **404** | URL do feed errada | Confira a URL com quem liberou o acesso e refaça o passo 1.2 |
| O npm avisa `Unknown user config "always-auth"` | Configuração antiga no seu arquivo do npm | Apague a linha `always-auth=true` do arquivo que `npm config get userconfig` mostra |
| `/guild-init` ou `/guild-upgrade` não aparecem no Claude Code | As skills de entrada não estão nesta máquina, ou a sessão abriu antes delas | Refaça o passo 1.3 e rode `/reload-skills` |
| O Guild diz que não há feed do CLI | O npm desta máquina não tem o escopo `@rbbsolucoes` | Refaça o passo 1.2 |
| Board responde **401** / `TF400813` **[Azure Boards]** | PAT ausente, expirado ou sem escopo | Confira a variável do passo 1.4 e abra um terminal novo |
| Algo falha com "sem node" no Windows | Rodou `bash` pelo PowerShell (WSL) | Use o Git Bash |

Para tirar as skills de entrada da máquina, apague as pastas `guild-init`,
`guild-help` e `guild-upgrade` de `~/.claude/skills/` e o arquivo
`~/.rbb/skills-de-entrada.json`: o `npm uninstall -g` não as remove.

Dúvidas ou feedback: `/feedback <contexto do problema>`. Sem o board da
plataforma, o envio vira uma issue pública no
[rabberto/rbb-guild](https://github.com/rabberto/rbb-guild/issues).
