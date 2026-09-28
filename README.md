# FUTBOTS Site

Webapp educacional para controlar um carrinho **Arduino Nano** via **HM-10** usando **Web Bluetooth** (Microsoft Edge / Chrome), com calibração PWM e programação por blocos.

Site estático preparado para **GitHub Pages**.

---

## URL esperada após o deploy

```
https://beast-robotics-br.github.io/FUTBOTS_Site/
```

(Ajuste o nome da org/usuário se for diferente.)

---

## 1. Pré-requisitos

- Node.js **22+**
- [pnpm](https://pnpm.io/) **10+**
  ```bash
  npm install -g pnpm
  ```
- Conta GitHub com permissão no repositório
- Navegador com Web Bluetooth (Edge ou Chrome) em **HTTPS** (GitHub Pages já entrega HTTPS)

---

## 2. Desenvolvimento local

```bash
# Clonar
git clone https://github.com/Beast-Robotics-BR/FUTBOTS_Site.git
cd FUTBOTS_Site

# Instalar
pnpm install

# Rodar em http://localhost:3000
pnpm dev
```

Build de produção local (simula o Pages):

```bash
VITE_BASE=/FUTBOTS_Site/ pnpm build
pnpm preview
```

Os arquivos estáticos ficam em `dist/public/`.

---

## 3. O que foi limpo para o GitHub Pages

| Removido / ajustado | Motivo |
|---|---|
| Plugins Manus no Vite | Só serviam o ambiente Manus (debug, storage) |
| `server/` (Express) | Pages só hospeda estático; BLE é no browser |
| `vite-plugin-manus-runtime`, jsx-loc, etc. | Dependências desnecessárias no build estático |
| Patch `wouter` quebrado | Pasta `patches/` não existia → `pnpm install` falhava |
| Analytics com placeholders | URLs inválidas no HTML |
| Pasta `__manus__/` pública | Artefato do runtime Manus |

| Mantido | Motivo |
|---|---|
| `client/` (React + Vite) | O app completo |
| `shared/const.ts` | Alias `@shared` usado pelo código |
| Web Bluetooth / programação por blocos | Funcionalidade principal (roda no cliente) |

---

## 4. Deploy no GitHub Pages (passo a passo)

### 4.1 Subir o código

Se você está usando este pacote limpo:

```bash
cd FUTBOTS_Site-pages   # ou o nome da pasta
git init
git add .
git commit -m "Prepare static site for GitHub Pages"
git branch -M main
git remote add origin https://github.com/Beast-Robotics-BR/FUTBOTS_Site.git
git push -u origin main
```

> Se o repositório já existir com histórico, faça merge/PR em vez de forçar push.

### 4.2 Ativar Pages com GitHub Actions

1. Abra o repositório no GitHub  
2. **Settings** → **Pages**  
3. Em **Build and deployment** → **Source**, escolha **GitHub Actions**  
4. Salve

### 4.3 Workflow automático

O arquivo `.github/workflows/deploy-pages.yml` já está incluído. Ele:

1. Instala dependências com pnpm  
2. Faz `vite build` com `VITE_BASE=/FUTBOTS_Site/`  
3. Copia `index.html` → `404.html` (fallback de SPA)  
4. Publica em GitHub Pages  

Dispara em:

- push na branch `main`
- execução manual em **Actions** → **Deploy FUTBOTS to GitHub Pages** → **Run workflow**

### 4.4 Gerar o lockfile (primeira vez)

Como o `package.json` foi limpo, rode **uma vez** localmente e commite o lock:

```bash
pnpm install
git add pnpm-lock.yaml
git commit -m "Add pnpm-lock.yaml"
git push
```

Se o workflow falhar em `--frozen-lockfile` sem o lock, ou rode o comando acima, ou altere temporariamente a linha do workflow para `pnpm install`.

### 4.5 Conferir o site

Após o workflow ficar verde:

```
https://beast-robotics-br.github.io/FUTBOTS_Site/
```

Pode levar 1–2 minutos após o deploy.

---

## 5. Domínio customizado (opcional)

1. Em **Settings → Pages → Custom domain**, coloque o domínio (ex.: `futbots.seudominio.com`)  
2. No DNS, crie um registro **CNAME** apontando para `beast-robotics-br.github.io`  
3. Se usar domínio próprio na **raiz** (user/org site), mude no workflow:

```yaml
VITE_BASE: /
```

---

## 6. Web Bluetooth e HTTPS

- Web Bluetooth **exige contexto seguro** (HTTPS ou `localhost`)  
- GitHub Pages já usa HTTPS → funciona no Edge/Chrome  
- No celular, use Chrome/Edge atualizado e permita Bluetooth quando o navegador pedir  

Fluxo típico (RoverLab):

1. Ligue o carrinho  
2. Abra o site no Edge/Chrome  
3. **Controle manual** → conectar BLE  
4. Teste **Parada** em potência baixa (L1)  
5. Calibre e só depois aumente PWM  

---

## 7. Estrutura do projeto

```
FUTBOTS_Site/
├── .github/workflows/deploy-pages.yml   # Deploy automático
├── client/
│   ├── index.html
│   ├── public/
│   └── src/                             # React app
│       ├── App.tsx
│       ├── pages/Home.tsx               # UI principal (BLE, blocos, CLI)
│       └── ...
├── shared/const.ts
├── package.json
├── vite.config.ts                       # base = VITE_BASE
├── tsconfig.json
└── README.md
```

---

## 8. Scripts

| Comando | Função |
|---|---|
| `pnpm dev` | Dev server (porta 3000) |
| `pnpm build` | Build estático → `dist/public` |
| `pnpm preview` | Preview do build |
| `pnpm check` | Typecheck TypeScript |
| `pnpm format` | Prettier |

---

## 9. Problemas comuns

**Página em branco / assets 404**  
→ `VITE_BASE` deve ser `/NomeDoRepo/` (com barras). O workflow já define isso.

**Refresh em rota dá 404**  
→ O workflow gera `404.html` igual ao `index.html`. Confirme que o step rodou.

**`pnpm install` falha no CI**  
→ Commit do `pnpm-lock.yaml` após `pnpm install` local, ou troque `--frozen-lockfile` por `pnpm install`.

**Bluetooth não aparece**  
→ Use Edge/Chrome em HTTPS; permita permissão; módulo HM-10 ligado e próximo.

**Workflow sem permissão**  
→ Settings → Actions → General → Workflow permissions: **Read and write**.  
Pages Source deve ser **GitHub Actions**.

---

## 10. Licença

MIT — veja o campo `license` em `package.json`.
