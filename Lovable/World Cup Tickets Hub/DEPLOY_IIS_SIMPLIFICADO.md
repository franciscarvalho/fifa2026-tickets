# 🏆 Deploy Simplificado - FIFA 2026 Tickets
## Windows Server + IIS + Node.js

---

## 📋 PARTE 1: Instalação de Pré-requisitos

### 1.1 Instalar IIS com todos os recursos necessários

Abra o **PowerShell como Administrador** e execute:

```powershell
# Instalar IIS
Write-Host "Instalando IIS..." -ForegroundColor Green
Install-WindowsFeature -Name Web-Server -IncludeManagementTools

# Instalar recursos adicionais do IIS
Write-Host "Instalando recursos adicionais do IIS..." -ForegroundColor Green
Install-WindowsFeature -Name Web-WebSockets
Install-WindowsFeature -Name Web-Stat-Compression
Install-WindowsFeature -Name Web-Dyn-Compression

Write-Host "✅ IIS instalado com sucesso!" -ForegroundColor Green
```

### 1.2 Instalar Node.js

Acesse: https://nodejs.org/en/download

Baixe e instale a versão **LTS** recomendada.

### 1.3 Instalar iisnode

Acesse: https://github.com/Azure/iisnode/releases

**⚠️ IMPORTANTE:** Instalar a versão **Full**

### 1.4 Instalar URL Rewrite Module

Acesse: https://www.iis.net/downloads/microsoft/url-rewrite

### 1.5 Instalar ARR (Application Request Routing)

Acesse: https://www.iis.net/downloads/microsoft/application-request-routing

---

## 📋 PARTE 2: Configuração da Aplicação

### 2.1 Baixar a aplicação (já compilada)

Baixe os **dois ZIPs prontos** (backend + frontend) e extraia ambos para `C:\inetpub\wwwroot`:

- Backend: https://stotfteccopaazure.blob.core.windows.net/copa2026/fifa2026-api.zip
- Frontend: https://stotfteccopaazure.blob.core.windows.net/copa2026/fifa2026-web.zip

Ao extrair os dois, você terá `C:\inetpub\wwwroot\fifa2026-api\` e `C:\inetpub\wwwroot\fifa2026-web\`.

> 💡 **Já vêm prontos:** o `fifa2026-api.zip` inclui a pasta `node_modules/` (você **não** precisa rodar `npm install`) e o `fifa2026-web.zip` já está buildado. Você não compila nada.

O arquivo "FIFA2026Tickets.bacpac" não será usado no servidor de aplicação, somente no servidor de Banco de Dados.

### 2.2 Configurar permissões na pasta da aplicação

No **PowerShell como Administrador**, execute:

```powershell
# Dar permissão ao IIS para acessar a pasta
$acl = Get-Acl "C:\inetpub\wwwroot\fifa2026-api"
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule("IIS_IUSRS", "FullControl", "ContainerInherit,ObjectInherit", "None", "Allow")
$acl.SetAccessRule($rule)
Set-Acl "C:\inetpub\wwwroot\fifa2026-api" $acl

$rule2 = New-Object System.Security.AccessControl.FileSystemAccessRule("IUSR", "FullControl", "ContainerInherit,ObjectInherit", "None", "Allow")
$acl.SetAccessRule($rule2)
Set-Acl "C:\inetpub\wwwroot\fifa2026-api" $acl

Write-Host "✅ Permissões configuradas!" -ForegroundColor Green
```

### 2.3 Apontar o frontend para o backend

O `web.config` do frontend vem com o placeholder `__BACKEND_URL__`. Como o backend roda **nesta mesma máquina** (porta 3001), substitua pelo `localhost`. PowerShell **como Administrador**:

```powershell
cd C:\inetpub\wwwroot\fifa2026-web
(Get-Content web.config) -replace '__BACKEND_URL__','http://localhost:3001' | Set-Content web.config
```

Confirme que não sobrou o placeholder:

```powershell
Select-String -Path web.config -Pattern '__BACKEND_URL__'   # NÃO deve retornar nada
```

---

## 📋 PARTE 3: Configuração do IIS

### 3.1 Habilitar Proxy no ARR

1. Abra o **Gerenciador do IIS**
2. Clique no nome do servidor (primeiro item da árvore)
3. No painel central, dê duplo clique em **"Application Request Routing Cache"**
4. No painel direito, clique em **"Server Proxy Settings..."**
5. Marque a opção **"Enable proxy"**
6. Clique em **"Apply"** no painel direito

### 3.2 Ajustar o Feature Delegation

1. Abra o **IIS Manager**
2. Clique no nome do servidor (nível raiz)
3. Clique em **Feature Delegation** (Delegação de Recursos)
4. Encontre **Handler Mappings**
5. Clique com botão direito → **Read/Write**

---

## 📋 PARTE 4: Criar Site no IIS

### 4.1 Abrir o Gerenciador do IIS

1. Clique no ícone de pesquisa
2. Digite: `IIS`
3. Clique em **"Gerenciador do Serviços de Informações da Internet (IIS)"**

### 4.2 Criar o site

1. No painel esquerdo, expanda o nome do servidor
2. Clique com o **botão direito** em **"Sites"**
3. Clique em **"Adicionar Site..."** (Add Website)

### 4.3 Configurar o site

Preencha os campos:

| Campo | Valor |
|-------|-------|
| Nome do site | `FIFA2026-API` |
| Caminho físico | Clique em `...` → Navegue até `C:\inetpub\wwwroot\fifa2026-api` → OK |
| Tipo | `http` |
| Endereço IP | Todos não atribuídos |
| Porta | `3001` |
| Nome do host | (deixe em branco) |

4. Clique em **"OK"**

### 4.4 Configurar o Application Pool

1. No painel esquerdo, clique em **"Pools de Aplicativos"**
2. Encontre **"FIFA2026-API"** na lista
3. Clique com o **botão direito** → **"Configurações Avançadas..."**
4. Encontre **"Versão do .NET CLR"**
5. Mude para: **"Sem Código Gerenciado"** (No Managed Code)
6. Clique em **"OK"**

### 4.5 Iniciar o site

1. Volte para **"Sites"** no painel esquerdo
2. Clique em **"FIFA2026-API"**
3. No painel direito, clique em **"Iniciar"** (se não estiver iniciado)

---

## 📋 PARTE 5: Testes

### 4.1 Testar se a API está respondendo

```powershell
# Testar health da API
Invoke-RestMethod -Uri "http://localhost:3001/api/health"

# Testar listagem de times
Invoke-RestMethod -Uri "http://localhost:3001/api/teams"

# Testar listagem de jogos
Invoke-RestMethod -Uri "http://localhost:3001/api/matches"
```

### 4.2 Testar conectividade do banco de dados

```powershell
Invoke-RestMethod -Uri "http://localhost:3001/api/health/db"
```

---

## 📋 PARTE 6: Criar Site do Frontend no IIS

### 6.1 Abrir o Gerenciador do IIS

1. Clique no ícone de pesquisa
2. Digite: `IIS`
3. Clique em **"Gerenciador do Serviços de Informações da Internet (IIS)"**

### 6.2 Criar o site

1. No painel esquerdo, expanda o nome do servidor
2. Clique com o **botão direito** em **"Sites"**
3. Clique em **"Adicionar Site..."** (Add Website)

### 6.3 Configurar o site

Preencha os campos:

| Campo | Valor |
|-------|-------|
| Nome do site | `FIFA2026-Web` |
| Caminho físico | Clique em `...` → Navegue até `C:\inetpub\wwwroot\fifa2026-web` → OK |
| Tipo | `http` |
| Endereço IP | Todos não atribuídos |
| Porta | `80` |
| Nome do host | (deixe em branco) |

4. Clique em **"OK"**

### 6.4 Configurar o Application Pool

1. No painel esquerdo, clique em **"Pools de Aplicativos"**
2. Encontre **"FIFA2026-Web"** na lista
3. Clique com o **botão direito** → **"Configurações Avançadas..."**
4. Encontre **"Versão do .NET CLR"**
5. Mude para: **"Sem Código Gerenciado"** (No Managed Code)
6. Clique em **"OK"**

### 6.5 Iniciar o site

1. Volte para **"Sites"** no painel esquerdo
2. Clique em **"FIFA2026-Web"**
3. No painel direito, clique em **"Iniciar"** (se não estiver iniciado)

### 6.6 Testar o Frontend

1. Abra o navegador
2. Acesse: `http://localhost`
3. O site FIFA 2026 deve carregar

---

# 🔧 PARTE 7: SOLUÇÃO DE PROBLEMAS

## Erro: "Não consegue conectar ao SQL Server"

### Possíveis causas e soluções:

1. **TCP/IP não habilitado:**
   - Abra SQL Server Configuration Manager
   - Habilite TCP/IP
   - Reinicie SQL Server

2. **Firewall bloqueando:**
   - Verifique regra para porta 1433 no Windows Firewall
   - Verifique NSG no Azure

3. **IP errado no .env:**
   - Use o IP **privado** da VM SQL, não o público
   - Verifique no Azure Portal

## Erro: "500 Internal Server Error" na API

### Verificar logs:

1. Abra a pasta: `C:\inetpub\wwwroot\fifa2026-api\logs`
2. Abra o arquivo de log mais recente
3. Procure por mensagens de erro

### Causas comuns:

1. **Arquivo .env não existe ou está errado**
2. **`node_modules` não veio no zip** — confirme a pasta `C:\inetpub\wwwroot\fifa2026-api\node_modules`; se faltar, rebaixe e reextraia o `fifa2026-api.zip` (não precisa `npm install`)
3. **Caminho do Node.js errado** no web.config

## Erro: "404 Not Found" no Frontend

### Verificar:

1. URL Rewrite está instalado?
2. web.config existe na pasta do frontend?
3. O arquivo index.html existe na pasta?

## Erro: "Site não carrega"

### No IIS Manager:

1. Verifique se o Application Pool está iniciado
2. Verifique se o Site está iniciado
3. Clique com botão direito no site → "Procurar" para testar

### Verificar eventos do Windows:

1. Pressione **Win + R**
2. Digite: `eventvwr.msc`
3. Vá em: Logs do Windows → Aplicativo
4. Procure por erros relacionados ao IIS

## API funciona, mas Frontend não conecta

O frontend chama `/api` na mesma origem e o IIS faz o proxy via `web.config` (ARR). Se `/api/*` falhar:

1. **Placeholder não substituído:** confirme que `C:\inetpub\wwwroot\fifa2026-web\web.config` não contém mais `__BACKEND_URL__` e aponta para `http://localhost:3001` (passo 2.3).
2. **ARR proxy desabilitado:** IIS Manager → servidor → Application Request Routing Cache → Server Proxy Settings → ✅ Enable proxy (passo 3.1).
3. Como front e back ficam na mesma origem (proxy reverso), **CORS não é exercitado** — não precisa configurar `CORS_ORIGIN`.

---

# 📞 INFORMAÇÕES IMPORTANTES

## Credenciais Padrão

| Sistema | Usuário | Senha |
|---------|---------|-------|
| Aplicação (Admin) | admin@fifa2026.com | admin123 |
| SQL Server (App) | fifa2026_db | F1f@2026App |

## Portas Utilizadas

| Serviço | Porta |
|---------|-------|
| Site (Frontend) | 80 |
| API (Backend) | 3001 |
| SQL Server | 1433 |
| Área de Trabalho Remota | 3389 |

## Caminhos Importantes

| Item | Caminho |
|------|---------|
| Backend | C:\inetpub\wwwroot\fifa2026-api |
| Frontend | C:\inetpub\wwwroot\fifa2026-web |
| Logs do Backend | C:\inetpub\wwwroot\fifa2026-api\logs |
| Logs do IIS | C:\inetpub\logs\LogFiles |

---

**🎉 Parabéns! Se você chegou até aqui e tudo está funcionando, seu sistema FIFA 2026 Tickets está no ar!**
