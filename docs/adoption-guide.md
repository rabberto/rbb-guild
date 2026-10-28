# Guia prático — adotando o Guild no seu time

Este guia leva você do zero até um repositório com o Guild funcionando: CLI
instalado, skills disponíveis no Claude Code, agentes derivados e o board
conectado.

O Guild roda no **Claude Code**. É lá que as guardas existem: a mesma skill
aberta em outra ferramenta roda sem nenhuma delas.

A maior parte do trabalho é **guiada por uma skill** dentro do Claude Code. O
que você faz à mão é só o que precisa existir antes de qualquer skill rodar:

| Etapa | Quem faz | Quando |
|---|---|---|
| **1. Preparar a máquina** | você, à mão (~10 min) | uma vez por máquina |
| **2. Configurar o repositório** | você, à mão (`rbb configure`) | uma vez por repositório |
| **3. Adoção guiada** | a skill `/guild-init` | uma vez por repositório |
| **Depois** | a skill `/guild-upgrade` | a cada versão nova da plataforma |

Passos marcados com **[Azure Boards]** só valem para o time que controla o
trabalho num board do Azure DevOps. Com o board em modo local, pule-os.

---

## O que é o Guild, em 30 segundos

| Peça | O que é |
|---|---|
| **Skills** | Processos de engenharia que a IA executa (`/guild-init`, `/codebase-analysis`, …) |
| **Agentes** | Um agente dev por camada da arquitetura, cada um com sua fronteira de escrita |
| **CLI `rbb`** | A parte mecânica: board, sincronização, perfil, verificação |
| **Guardas** | Hooks do Claude Code que **bloqueiam de verdade** escrita fora da fronteira do agente |

O catálogo completo — cada skill, comando e agente, com o que recebe e o que
gera — abre com:

```bash
rbb catalog
```

Ele mostra a versão instalada na sua máquina, não a mais nova da plataforma. A
mais nova está publicada em <https://rabberto.github.io/rbb-guild/>.

---

## Etapa 1 — Preparar a máquina

Nada disto pode ser feito por skill: ou é o que faz as skills existirem (o
CLI), ou é segredo (tokens), que as skills nunca pedem nem gravam.

### 1.1 Pré-requisitos

- **Node.js** (LTS) e **npm**
- **Git**
- **Claude Code** (CLI ou extensão do VS Code)
- Acesso ao **feed** do CLI — é o único acesso necessário: skills, agentes e
  templates vêm dentro do pacote. Quem distribui o Guild para o seu time libera
  o acesso ao feed

> **Windows: use o Git Bash.** No PowerShell, o comando `bash` é o do WSL —
> outro sistema, sem o seu Node. Comandos e gates do Guild rodados por ali
> falham com mensagens enganosas.

### 1.2 Autenticar o npm no feed do Guild

O CLI é distribuído por um feed npm do Azure Artifacts. Crie (ou complemente)
o arquivo `~/.npmrc` com a URL do feed, que vem de quem liberou o seu acesso:

```ini
@rbbsolucoes:registry=<url-do-feed>
always-auth=true
```

**Windows:**

```bash
npm install -g vsts-npm-auth
vsts-npm-auth -config ~/.npmrc
```

**Linux / macOS / WSL:** gere um PAT em *User settings → Personal Access
Tokens* com escopo **Packaging: Read** e adicione-o ao `~/.npmrc` no formato
mostrado em *Artifacts → Connect to feed → npm → Other*.

Teste:

```bash
npm view @rbbsolucoes/rbb-engineering-cli version
```

Respondeu uma versão? Autenticação OK.

### 1.3 Instalar o CLI

```bash
npm install -g @rbbsolucoes/rbb-engineering-cli
rbb version
```

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

As skills vêm no pacote do CLI que você acabou de instalar: o `rbb sync` as
copia dele para o projeto, sem clonar nada. O que o repositório precisa guardar
é só o feed, para o `rbb update` saber de onde instalar as versões novas. Na
raiz do repositório do seu projeto:

```bash
rbb configure
```

Ele pergunta **uma URL**: o feed npm do CLI. O `rbb configure` grava
`.rbb/config.json` **dentro do repositório**. O arquivo é versionado e não tem
segredo: quem clonar depois já vem configurado, e nesta etapa só confere com
`rbb configure --show`.

Depois, traga as skills:

```bash
rbb sync all
```

Ele instala skills e agentes em `.claude/` e diz de onde vieram ("Skills da
versão X do CLI"). Correção de skill chega com o `rbb update`, e não no próximo
sync: CLI e skills andam sempre na mesma versão. Cada chave da configuração, e
de onde ela veio, aparece em `rbb configure --show`.

---

## Etapa 3 — Adoção guiada

Abra o Claude Code na raiz do repositório e execute:

```
/guild-init
```

A skill lê o que já existe, mostra um **plano** antes de agir e conduz o resto.
Você responde às perguntas; ela explica cada escolha.

| # | O que acontece |
|---|---|
| 1 | Ressincroniza a plataforma |
| 2 | Conecta o board — modo (Azure Boards ou local); em Azure Boards, também organização, projeto, nome da variável do PAT e mapeamento de estados, e testa a conexão |
| 3 | Gera o perfil do projeto (`.rbb/project.yaml`) com `codebase-analysis`, **se não existir** |
| 4 | Atualiza no perfil o que é verificável (dependências) |
| 5 | Deriva um agente por camada **e instala os hooks de guarda** |
| 6 | Roda o `rbb doctor` — vermelho deixa a adoção como pendente |

> **Mapeamento de estados do board [Azure Boards]:** responda com os estados
> **reais** do processo do seu board, não com o preset "mais parecido". Estado
> que não existe no board faz a guarda recusar transições legítimas depois.

Parou no meio? Rode `/guild-init` de novo — ela confere o estado antes de cada
passo e não destrói configuração existente.

### Trabalho em aberto já existente

Se o repositório já tem backlog, planos ou documentos de trabalho em `docs/`,
rode **uma vez**:

```
/board-adoption
```

Ela leva esse trabalho para o board, para não existirem dois controles em
paralelo.

### Confirmar que a guarda funciona de verdade

"O comando rodou sem erro" não prova que o hook dispara. Confira:

```bash
rbb guard status                           # hooks registrados e fronteira de cada camada
rbb guard check <caminho-fora-da-camada>   # deve BLOQUEAR (exit 2)
```

### Precisa trabalhar sem a guarda?

```bash
rbb guard off     # desliga
rbb guard on      # religa
```

O `rbb guard off` desliga **só a guarda de escrita**, **só para quem roda** e
**só naquela máquina**. Os hooks continuam registrados no `.claude/settings.json`
do time, e a guarda segue valendo para todos os outros. O desligamento fica
registrado no ledger, e o `rbb guard status` diz desde quando está desligada.

### Versionar

Commite o que a adoção gerou (`.claude/`, `.gitignore`, `.rbb/config.json`,
`.rbb/project.yaml`, `.rbb/boards.json`) **numa branch** e abra PR, pelo fluxo
de branch do seu time. O `.gitignore` já vem com as regras certas: o resto de
`.rbb/` é estado local e fica fora do git.

---

## Depois — mantendo em dia

Saiu versão nova da plataforma, ou um comando do `rbb` falhou por configuração
(feed, repositório)? Abra o Claude Code no repositório e rode:

```
/guild-upgrade
```

Ela faz o upgrade inteiro, na ordem certa, e pergunta no chat o que faltar
configurar em vez de deixar o comando falhar:

| O que ela cuida | No lugar de |
|---|---|
| Configuração: o feed faltando, ou URLs antigas | `rbb configure` |
| Atualizar o CLI | `rbb update` |
| Ressincronizar skills e agentes | `rbb sync all` |
| Atualizar o perfil | `rbb profile refresh` |
| Re-derivar o time e reinstalar os hooks | `rbb agents derive` |
| Verificar | `rbb doctor` |

Ela não pede tokens: se faltar autenticação no feed, ela mostra o que fazer e
você refaz o passo 1.2.

---

## Checklist final

- [ ] `rbb version` responde
- [ ] `rbb configure --show` → `"origem": "projeto"`, `"faltando": []`, `"legado": []`
- [ ] `rbb doctor` verde
- [ ] `.rbb/project.yaml` existe e descreve as camadas reais do código
- [ ] Agentes em `.claude/agents/`, um por camada
- [ ] `rbb guard check` bloqueia escrita fora da camada
- [ ] Nenhum arquivo em `.claude/commands/` contém `{{include:` (se contiver, o CLI está desatualizado — rode `/guild-upgrade`)

---

## Dia a dia

| Quero… | Comando |
|---|---|
| Ver todas as skills, comandos e agentes da versão instalada | `rbb catalog` |
| Saber qual skill serve para o meu caso | `/guild-help` |
| Atualizar a plataforma inteira (CLI, skills, agentes, hooks) | `/guild-upgrade` |
| Ver o estado do Guild neste repositório | `rbb guild` |
| Ver um card do board (com as anotações do Discussion) | `rbb boards get <id>` |
| Diagnosticar a instalação | `rbb doctor` |
| Trabalhar sem a guarda, só nesta máquina | `rbb guard off` (e `rbb guard on` para religar) |

---

## Problemas comuns

| Sintoma | Causa provável | O que fazer |
|---|---|---|
| `npm` responde **401/403** ao instalar | npm não autenticado no feed | Refaça o passo 1.2 (`vsts-npm-auth`) |
| `npm` responde **404** | URL do feed errada no `.npmrc` | Copie a URL de *Artifacts → Connect to feed* |
| `/guild-init` ou `/guild-upgrade` não aparecem no Claude Code | Etapa 2 não foi feita neste repositório | Rode a etapa 2 e reabra a sessão |
| `rbb sync all` falha pedindo repositório | Repositório sem `.rbb/config.json` | Etapa 2 |
| Aviso "Usando ~/.rbb/config.json (global)" | Config antiga, da época em que era por máquina | Etapa 2 — grava a config no repositório |
| `rbb update` diz que não há feed configurado | Rodou fora de um repositório configurado | Rode dentro do repositório, ou defina `RBB_NPM_REGISTRY` com a URL do feed |
| Board responde **401** / `TF400813` **[Azure Boards]** | PAT ausente, expirado ou sem escopo | Confira a variável do passo 1.4 e abra um terminal novo |
| Gate ou script falha com "sem node" no Windows | Rodou `bash` pelo PowerShell (WSL) | Use o Git Bash |

Dúvidas ou feedback: `rbb feedback new` registra, e o `rbb feedback push` envia.
Sem o board da plataforma, o envio vira uma issue pública no
[rabberto/rbb-guild](https://github.com/rabberto/rbb-guild/issues), e o push pede a
confirmação: `rbb feedback push --publico`.
