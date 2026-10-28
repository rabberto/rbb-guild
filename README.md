# Rbb Guild — canal público

O **Guild** é a plataforma de engenharia da Rbb Soluções: skills que o Claude Code
executa, agentes por camada da arquitetura, o CLI `rbb` e as guardas que
bloqueiam escrita fora da fronteira de cada agente.

Este repositório **não tem o código da plataforma**. Ele existe para duas coisas:
publicar o catálogo e o guia de adoção, e receber o que você tem a dizer sobre o
Guild.

## Documentação

| | |
|---|---|
| [Catálogo](https://rabberto.github.io/rbb-guild/) | cada skill, comando e agente da versão mais nova, com o que recebe e o que gera |
| [Guia prático de adoção](https://github.com/rabberto/rbb-guild/blob/main/docs/adoption-guide.md) | do zero até um repositório com o Guild funcionando, em português |

Os dois são publicados automaticamente a cada versão do CLI. Não edite `docs/` aqui.

## Abrir uma issue

Aba **Issues → New issue** e escolha o formulário:

| Formulário | Use quando |
|---|---|
| **Problema** | algo do Guild não funciona como deveria — skill, comando `rbb`, agente, guarda, instalação |
| **Sugestão** | uma melhoria ou um recurso novo |
| **Dúvida** | você não sabe como fazer algo com o Guild |

Antes de abrir uma dúvida, vale tentar `/guild-help` no Claude Code e
`rbb catalog` no terminal — eles mostram o que cada skill e comando faz.

## Esta issue é pública

Qualquer pessoa na internet lê o que você escrever aqui. **Não cole:**

- código ou nome de cliente
- tokens, PATs, senhas, chaves
- URLs internas (Azure DevOps, feeds, ambientes)
- conteúdo de `.rbb/` sem revisar

Descreva o comportamento com um exemplo mínimo, no lugar do caso real.

## O que acontece depois

Toda issue nasce com o label `triagem`. Cada uma é lida e termina em um destes:

| Label | Significa |
|---|---|
| `aceita` | virou card no board da plataforma — o comentário traz o número |
| `recusada` | não será feita — o comentário diz por quê |
| `duplicada` | já existe outra issue sobre o mesmo assunto |

A implementação acontece no repositório da plataforma; a issue é fechada quando
a correção é publicada no CLI (`rbb update`).

## Pull requests

Não são aceitos — o código não mora aqui. Se você tem uma proposta de mudança,
abra uma **Sugestão** descrevendo-a.

---

© Rbb Soluções. Todos os direitos reservados.
