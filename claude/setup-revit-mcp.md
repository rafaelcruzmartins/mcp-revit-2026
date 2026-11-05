# Tarefa: conectar o Claude ao Revit (pyRevit + revit-mcp-server)

Você está rodando no Claude Code, num PC Windows. O usuário já instalou e configurou o pyRevit (carregador beta desligado e Servidor de Rotas ligado). Sua tarefa é fazer todo o resto e deixar o MCP do Revit funcionando no Claude Code e no chat do Claude Desktop.

## Regras de execução

- Trabalhe de forma autônoma. Não peça confirmação a cada etapa; execute, verifique e corrija sozinho.
- Use PowerShell. Não exija privilégios de administrador; tudo é instalado no perfil do usuário.
- Não presuma nome de usuário, letra de disco nem caminhos: descubra tudo com `$env:USERPROFILE`, `$env:APPDATA`, `$env:LOCALAPPDATA`, `where.exe`, `Get-Command` etc.
- Se algo falhar, diagnostique a causa e tente outra rota antes de desistir. Só pare para falar com o usuário se for algo que ele precisa fazer com as próprias mãos (ex.: abrir o Revit).
- Antes de editar qualquer arquivo de configuração existente, faça um backup com sufixo `.bak-AAAAMMDD-HHMMSS` ao lado do original.
- Não feche nem reinicie o Claude Desktop: você pode estar rodando dentro dele. Ao final, diga ao usuário para reiniciar.
- Não libere nada no Firewall do Windows. A conexão é local (localhost).
- Nunca acione o "Recarregar" do pyRevit nem chame rotas de reload: com o Servidor de Rotas ativo isso derruba o Revit.

## Etapas

### 1. Diagnóstico inicial
Verifique e anote:
- Versão do Windows e do PowerShell.
- Se o pyRevit está instalado (`pyrevit --version` ou pasta `%APPDATA%\pyRevit-Master` / `%PROGRAMDATA%\pyRevit-Master`). Se não estiver, avise o usuário e pare: ele precisa fazer os passos 1 e 2 do tutorial (instalador testado: https://github.com/pyrevitlabs/pyRevit/releases/download/v6.5.5.26237%2B2044/pyRevit_6.5.5.26237_signed.exe).
- Se o Revit está em execução (`Get-Process Revit -ErrorAction SilentlyContinue`).
- Se `git` e `uv` já existem no PATH.

### 2. Conferir a configuração do pyRevit
Leia `%APPDATA%\pyRevit\pyRevit_config.ini`.
- Confirme que o Servidor de Rotas está habilitado (seção `[routes]`, chave tipo `enabled = true`) e anote a porta (padrão 48884).
- Confirme que o novo carregador beta está desligado (chave de "new loader" na seção `[core]`, se existir).
- Se algum dos dois estiver errado e o Revit estiver FECHADO, corrija no ini (com backup). Se o Revit estiver aberto, não edite: anote para avisar o usuário no final.
- Se não conseguir identificar as chaves com segurança, não invente: apenas registre no relatório final.

### 3. Instalar o uv
Se `uv` não existir:
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```
Depois atualize o PATH da sessão atual (o instalador põe o uv em `$env:USERPROFILE\.local\bin`) e confirme com `uv --version`. Guarde o caminho absoluto do `uv.exe` (`(Get-Command uv).Source`); ele será usado nas configurações.

### 4. Baixar a extensão revit-mcp-server
Pasta de extensões do pyRevit: use `pyrevit extensions paths` se disponível; senão `%APPDATA%\pyRevit\Extensions`. Crie a pasta se não existir.

Destino: `<Extensions>\mcp-server-for-revit-python.extension`
- Se a pasta já existir e for um repositório git, rode `git pull`.
- Se existir sem ser git ou estiver corrompida, renomeie para `.old-<data>` e baixe de novo.
- Com git: `git clone https://github.com/Demolinator/revit-mcp-server.git "<destino>"`
- Sem git: baixe `https://github.com/Demolinator/revit-mcp-server/archive/refs/heads/master.zip`, extraia e mova o conteúdo para o destino (o nome da pasta final tem que ser exatamente `mcp-server-for-revit-python.extension`).

Se a pasta usada não for a padrão do pyRevit, registre-a com `pyrevit extensions paths add "<pasta>"` e confirme com `pyrevit extensions paths`.

### 5. Preparar o ambiente Python
Dentro da pasta da extensão:
```powershell
uv sync
```
Leia o README e o `main.py` do repositório e confirme que as dependências importam sem erro (ex.: `uv --directory "<destino>" run python -c "import mcp"`). Não deixe o servidor rodando em primeiro plano: ele usa stdio e fica esperando entrada. Corrija dependências se falhar.

### 6. Registrar no Claude Code
```powershell
claude mcp remove revit --scope user 2>$null
claude mcp add --scope user revit -- "<caminho completo do uv.exe>" --directory "<destino>" run main.py
claude mcp list
```
Confirme que `revit` aparece.

### 7. Registrar no chat do Claude Desktop
Localize o `claude_desktop_config.json`:
- Instalação padrão: `%APPDATA%\Claude\claude_desktop_config.json`
- Instalação pela Microsoft Store/MSIX: procure em `%LOCALAPPDATA%\Packages\Claude_*\LocalCache\Roaming\Claude\claude_desktop_config.json`
- Se existirem os dois, atualize os dois. Se nenhum existir, crie em `%APPDATA%\Claude\`.

Faça backup, leia como JSON (não edite como texto), e faça um merge: crie `mcpServers` se não existir e adicione/substitua só a chave `revit`, preservando todo o resto:
```json
"revit": {
  "command": "<caminho completo do uv.exe com barras normais>",
  "args": ["--directory", "<destino com barras normais>", "run", "main.py"]
}
```
Grave em UTF-8 sem BOM e valide relendo o arquivo como JSON. Se o arquivo original estiver inválido (ex.: dois blocos `{ }` seguidos), conserte unindo tudo num único objeto.

### 8. Teste com o Revit
- Se o Revit estiver aberto com um projeto: faça `Invoke-RestMethod http://localhost:<porta>/revit_mcp/status/`. O esperado é `health: healthy` e `revit_available: true`.
- Se o Revit estava aberto durante a instalação da extensão, ela ainda não foi carregada: a rota vai responder "Route does not exist". Isso é normal; peça ao usuário para fechar e abrir o Revit e abrir um projeto, e depois repita o teste.
- Se o Revit estiver fechado: peça ao usuário para abrir o Revit e um projeto, espere ele avisar e repita o teste.
- Se der "connection refused": o Servidor de Rotas está desligado ou não há projeto aberto.
- Com o status OK, use a ferramenta MCP do revit (ex.: status e listar níveis) para confirmar ponta a ponta. Se as ferramentas MCP ainda não aparecerem nesta sessão, diga ao usuário para abrir uma nova sessão no Code do Claude Desktop e pedir o teste lá.

### 9. Relatório final ao usuário
Em poucas linhas:
- O que foi instalado/alterado e onde (caminhos), incluindo os backups criados.
- Resultado do teste.
- O que ele ainda precisa fazer, se houver: reiniciar o Claude Desktop pela bandeja do sistema (botão direito > Sair, e abrir de novo), reiniciar o Revit, corrigir algo no pyRevit.
- Lembretes: nunca usar "Recarregar" no pyRevit com as Rotas ligadas; nos primeiros usos trabalhar numa cópia do modelo.
