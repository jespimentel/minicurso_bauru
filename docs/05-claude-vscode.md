---
title: "5. Claude no VS Code"
---

## Visão geral do tema

**Subtópicos:**

-   O que é: extensão oficial que traz o Claude Code diretamente para dentro da IDE, sem precisar alternar para o terminal

-   Interface: painel lateral com visualização de alterações em tempo real e diff inline (lado a lado) para aceitar ou rejeitar cada mudança

-   Modos de operação: *Normal* (pede confirmação a cada edição) e *Plan* (apresenta o plano completo em Markdown antes de agir)

-   O que pode fazer: ler e editar arquivos do projeto, gerar scripts e executar comandos

-   MCP e navegação

## Cowork X Claude Code

| | Cowork | Claude Code |
|---|---|---|
| **Interface** | Desktop (GUI) | Terminal / VS Code |
| **Executa código** | Sim, em VM sandboxed | Sim, no terminal local |
| **Público-alvo** | Conhecimento geral | Desenvolvedor / Usuário avançado |
| **Acesso a arquivos** | Pasta designada pelo usuário | Direto no filesystem |
| **MCP** | Sim | Sim |

## Referências

-   [Extensão no Marketplace](https://marketplace.visualstudio.com/items?itemName=Anthropic.claude-code)

-   [Anthropic Docs. Claude Code](https://docs.anthropic.com/en/docs/claude-code)