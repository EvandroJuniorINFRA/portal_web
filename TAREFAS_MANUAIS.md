# Tarefas manuais (atualizado)

## Decisao de arquitetura (atual)

- Dados dos titulos: **somente Oracle** (sem API TOTVS).
- **VPS com IP fixo** roda portal + API e consulta o Oracle remoto.
- **Dominio** aponta para a VPS (HTTPS com Nginx).
- **Nao instale Oracle Database na VPS** — use o Oracle que o Winthor ja utiliza; libere o **IP da VPS** no ACL.

Guia principal: **[INSTALACAO_VPS_DOMINIO.md](./INSTALACAO_VPS_DOMINIO.md)**  
Alternativa rede interna: **[INSTALACAO_SERVIDOR_192.168.1.210.md](./INSTALACAO_SERVIDOR_192.168.1.210.md)**

---

## Voce precisa fazer (resumo)

### 1. Escolher onde rodar

| Opcao | Quando usar |
|-------|-------------|
| **Servidor Windows na empresa** | Ja acessa Oracle e pasta `\\10.0.1.3` |
| **VPS Linux** | IP fixo na nuvem; PDF na rede interna nao funciona na VPS |

### 2. Oracle — liberar IP

1. Descubra o IP publico: https://ifconfig.me (no servidor ou na rede do servidor)
2. Oracle Cloud → banco → **ACL** → adicione `SEU_IP/32`
3. Teste: `http://servidor:3333/api/health/oracle` → `"status":"ok"`

### 3. Instalar o sistema

**Windows:** duplo clique em `iniciar-producao-windows.bat`  
(antes: copie `.env` que ja funciona no seu PC)

**Linux VPS:** veja [DEPLOY_VPS.md](./DEPLOY_VPS.md) caminho B

### 4. Manter ligado

Windows: `pm2 start ecosystem.config.cjs` + `pm2 save`  
Linux: igual

### 5. Internet (opcional)

- Dominio apontando para o servidor
- Nginx + HTTPS (exemplo em `nginx/ciadosilk-portal.conf.example`)
- Ou Cloudflare Tunnel no servidor local

### 6. Desligar Vercel/Railway (opcional)

Nao sao mais necessarios se tudo rodar no servidor.

---

## Variaveis `.env` no servidor (producao)

```env
NODE_ENV=production
HOST=0.0.0.0
PORT=3333
SERVE_WEB=true
USE_MOCK_DATA=false
ORACLE_USER=...
ORACLE_PASSWORD=...
ORACLE_CONNECT_STRING=...
WINTHOR_DOCUMENT_COLUMN=CGCENT
BOLETO_FILE_BASE_PATH=\\10.0.1.3\winthor-share-file\Winthor
VITE_WHATSAPP_NUMBER=5511...
```

Build com WhatsApp: defina `VITE_WHATSAPP_NUMBER` **antes** de `npm run build:production`.
