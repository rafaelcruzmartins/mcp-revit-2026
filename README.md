# MCP Revit 2026

Tutoriais para conectar agentes de IA ao Revit via pyRevit + [revit-mcp-server](https://github.com/Demolinator/revit-mcp-server) (48 ferramentas). Testado no Revit 2026 com pyRevit 6.5.5.

## Escolha o agente

| Agente | Tutorial |
|---|---|
| **Claude** (Claude Code + Claude Desktop) | [claude/](claude/README.md) — passo a passo + arquivo [`setup-revit-mcp.md`](claude/setup-revit-mcp.md) para o Claude Code executar |
| **Google Antigravity** | [antigravity/](antigravity/README.md) — cole o arquivo numa conversa nova do Antigravity |

## pyRevit (pré-requisito dos dois)

Versão testada: **[⬇ pyRevit_6.5.5.26237_signed.exe](https://github.com/pyrevitlabs/pyRevit/releases/download/v6.5.5.26237%2B2044/pyRevit_6.5.5.26237_signed.exe)** (download direto). Instale com o Revit fechado.

Versões novas: [github.com/pyrevitlabs/pyRevit/releases](https://github.com/pyrevitlabs/pyRevit/releases)

## Avisos

- Nunca use "Recarregar" nem "Salvar configurações e recarregar" do pyRevit com o Servidor de Rotas ligado: derruba o Revit.
- Nos primeiros usos, trabalhe numa cópia do modelo.
- Não use Claude e Antigravity no Revit ao mesmo tempo.
