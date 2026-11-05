# Revit 2026 com ChatGPT e Codex

Este guia reaproveita o `revit-mcp-server` instalado no pyRevit. Se o Revit já responde pelo Claude, use a mesma pasta da extensão e configure apenas o cliente da OpenAI.

| Onde você quer usar | Como conectar |
| --- | --- |
| Codex local no Windows | Cadastre o servidor local como STDIO. |
| ChatGPT no aplicativo para Windows | O aplicativo permite cadastrar MCP local nas configurações do host. A disponibilidade das ferramentas precisa ser confirmada no modo de conversa que você quer usar. |
| ChatGPT no navegador (`chatgpt.com`) | Requer uma integração separada com um plugin que acesse um servidor MCP remoto. |

## 1. Confira os pré-requisitos

Abra o Revit com um projeto. A extensão do MCP deve estar instalada e o Servidor de Rotas do pyRevit deve estar ativo. Para uma instalação nova, consulte a seção de pyRevit no [README principal](../README.md) e as etapas de instalação do `uv` e da extensão no [guia de instalação](../claude/setup-revit-mcp.md). O cadastro no Claude é opcional para este guia.

No PowerShell, descubra o caminho do executável e confira a pasta padrão da extensão:

```powershell
$uv = (Get-Command uv -ErrorAction Stop).Source
$ext = Join-Path $env:APPDATA 'pyRevit\Extensions\mcp-server-for-revit-python.extension'
$uv
$ext
Test-Path (Join-Path $ext 'main.py')
```

Se o último comando retornar `False`, ajuste `$ext` para a pasta da extensão que contém `main.py`. Se `uv` não estiver no PATH do PowerShell, verifique `%USERPROFILE%\.local\bin\uv.exe`.

## 2. Cadastre o MCP local

No aplicativo para Windows, abra **Configurações → Servidores MCP**. Se `revit` já aparecer como habilitado, siga para o teste. Caso contrário, escolha **Adicionar servidor** e use estes campos:

| Campo | Valor |
| --- | --- |
| Nome | `revit` |
| Tipo | `STDIO` |
| Comando | Caminho completo de `uv.exe`, obtido no passo anterior |
| Argumentos | `--directory`, caminho completo da extensão, `run`, `main.py` |

Informe cada argumento separadamente, mantendo o caminho da extensão como um único argumento. Salve e use a opção de reiniciar do aplicativo.

Se estiver usando a CLI do Codex, o cadastro equivalente é:

```powershell
codex mcp list
# Execute a linha abaixo apenas se o servidor revit ainda não estiver cadastrado.
codex mcp add revit -- $uv --directory $ext run main.py
```

O aplicativo e a CLI podem compartilhar a configuração quando usam o mesmo host local. Consulte a [documentação da OpenAI sobre MCP](https://learn.chatgpt.com/docs/extend/mcp#connect-an-existing-mcp-server).

## 3. Teste no modo que pretende usar

Em uma nova conversa local, peça: **“Use `get_revit_status` para verificar a conexão com o Revit e informar o nome do projeto aberto.”** Confirme que a conversa chamou a ferramenta e que o nome coincide com o projeto aberto. Em seguida, peça para consultar a vista atual com `get_current_view_info`. Se a consulta expirar, verifique se o Revit está ocupado ou com uma janela de diálogo aberta e tente novamente quando estiver livre. Evite usar **Recarregar** no pyRevit com o Servidor de Rotas ativo.

No teste realizado no **Codex local**, `get_revit_status` respondeu que o Revit estava ativo e retornou o nome do projeto. A consulta da vista atual excedeu o tempo limite. O modo **Chat** do aplicativo e o navegador não foram validados nesse teste.

## ChatGPT pelo navegador

O `chatgpt.com` usa ferramentas MCP remotas por meio de plugins. Para reaproveitar este servidor STDIO, seria necessário criar uma integração separada que ofereça as ferramentas via MCP por HTTP e conectar o plugin ao ChatGPT. O cadastro local acima não habilita o Revit no navegador. A disponibilidade depende do plano e das permissões do workspace. Veja o [guia oficial para conectar e testar um plugin](https://developers.openai.com/plugins/deploy/connect-and-test).
