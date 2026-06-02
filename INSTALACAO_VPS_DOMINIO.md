# VPS + dominio (cenario oficial)

## Entendimento correto

| O que | Onde fica |
|-------|-----------|
| **Banco Winthor (Oracle)** | Ja existe (Oracle Cloud / servidor TOTVS) — **nao migre o Winthor para a VPS** |
| **Portal + API (este projeto)** | **VPS** com IP fixo |
| **Liberacao de acesso** | IP publico da **VPS** cadastrado no ACL do Oracle |
| **Dominio** | Aponta para a VPS (ex.: `boletos.ciadoscilk.com.br`) |

Na VPS voce instala **Node.js**, nao o servidor Oracle Database completo.  
O pacote `oracledb` do Node conecta no Oracle remoto (mesmo `ORACLE_CONNECT_STRING` do `.env`).

---

## 1. Contratar VPS

Requisitos minimos:

- **SO:** Ubuntu 22.04 LTS
- **RAM:** 1 GB (2 GB recomendado)
- **IP fixo** (padrao em VPS)
- Provedores: Hostinger, Contabo, Oracle Cloud VM, DigitalOcean, etc.

Anote o **IP publico da VPS** (ex.: `200.150.x.x`).

---

## 2. Liberar VPS no Oracle Cloud

1. Console Oracle → banco Autonomous (o mesmo do Winthor)
2. **Network** → **Access Control List**
3. Adicione: `IP_DA_VPS/32` (ex.: `200.150.10.20/32`)
4. Salve e aguarde alguns minutos

Teste depois do deploy: `https://boletos.seudominio.com.br/api/health/oracle`

---

## 3. Instalar software na VPS

Conecte via SSH:

```bash
ssh root@IP_DA_VPS
```

### Atualizar e instalar pacotes

```bash
apt update && apt upgrade -y
apt install -y git nginx certbot python3-certbot-nginx ufw curl
```

### Node.js 22

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs
node -v
npm -v
```

### Firewall

```bash
ufw allow OpenSSH
ufw allow 'Nginx Full'
ufw enable
```

---

## 4. Clonar o projeto

```bash
mkdir -p /var/www
cd /var/www
git clone https://github.com/evandromxjunior/portal_web.git
cd portal_web
npm install
```

---

## 5. Arquivo `.env` na VPS

```bash
nano /var/www/portal_web/.env
```

Conteudo (ajuste valores reais):

```env
NODE_ENV=production
HOST=0.0.0.0
PORT=3333
SERVE_WEB=true
USE_MOCK_DATA=false

ORACLE_USER=CIASILK
ORACLE_PASSWORD=sua_senha
ORACLE_CONNECT_STRING=201.157.219.14:1521/CG6V4Y_195977_W_high.paas.oracle.com

WINTHOR_DOCUMENT_COLUMN=CGCENT
RECEIVABLES_CACHE_TTL_MS=300000

VITE_WHATSAPP_NUMBER=5511999999999
CORS_ORIGIN=*
```

**PDF na rede `\\10.0.1.3`:** nao funciona na VPS Linux (caminho Windows).  
Linha digitavel e listagem de titulos funcionam. PDF exige servidor Windows na rede da empresa.

Build (WhatsApp entra no build):

```bash
cd /var/www/portal_web
npm run build:production
```

---

## 6. PM2 (API + portal sempre ligados)

```bash
npm install -g pm2
cd /var/www/portal_web
pm2 start ecosystem.config.cjs
pm2 save
pm2 startup
# execute o comando que o PM2 mostrar
```

Teste local na VPS:

```bash
curl http://127.0.0.1:3333/api/health
curl http://127.0.0.1:3333/api/health/oracle
```

---

## 7. Dominio + HTTPS (Nginx)

### DNS

No registro do dominio (Registro.br, Cloudflare, etc.):

| Tipo | Nome | Valor |
|------|------|--------|
| **A** | `boletos` | `IP_DA_VPS` |

Resultado: `boletos.seudominio.com.br` → VPS

### Nginx

```bash
nano /etc/nginx/sites-available/ciadosilk
```

```nginx
server {
    listen 80;
    server_name boletos.seudominio.com.br;

    location / {
        proxy_pass http://127.0.0.1:3333;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
ln -s /etc/nginx/sites-available/ciadosilk /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx
```

### Certificado SSL (HTTPS)

```bash
certbot --nginx -d boletos.seudominio.com.br
```

Acesso final: **https://boletos.seudominio.com.br**

---

## 8. Vercel e Railway

Neste cenario **nao sao necessarios**:

- Tudo (portal + API + consulta Oracle) fica na VPS.
- Desligue projetos na Vercel/Railway para evitar confusao.

---

## 9. Checklist

- [ ] VPS com IP fixo
- [ ] IP da VPS no Oracle ACL
- [ ] Node + Git + Nginx + PM2 instalados
- [ ] `.env` com Oracle correto
- [ ] `npm run build:production` OK
- [ ] `pm2 status` online
- [ ] `/api/health/oracle` → `"status":"ok"`
- [ ] DNS apontando para VPS
- [ ] HTTPS com certbot
- [ ] Portal abre no dominio e busca CPF funciona

---

## 10. Atualizar versao

```bash
cd /var/www/portal_web
git pull
npm install
npm run build:production
pm2 restart ciadosilk-portal
```

---

## Quando instalar Oracle Database NA VPS?

**Somente se** a TOTVS/empresa decidir **migrar o banco Winthor inteiro** para essa VPS.  
Isso e projeto de infraestrutura/DBA (licenca, backup, Winthor apontando para novo servidor), **nao** e necessario para este portal de boletos.

Para este projeto basta: **VPS + Node + conexao ao Oracle existente**.
