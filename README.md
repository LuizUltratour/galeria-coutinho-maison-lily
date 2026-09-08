# Galeria — Coutinho Maison Lily

Galeria de imagens/plantas/vídeo para injeção via script no 3DVista, hospedada no AWS S3.

**Empreendimento:** Maison Lily — Coutinho
**Tema:** cinza-chumbo + dourado (`#646461` / `#2B2A27`)

---

## URLs de produção

| Arquivo | URL |
|---------|-----|
| Galeria | `https://skylineip.s3.sa-east-1.amazonaws.com/Tour+Virtual/COUTINHO/galeria-maison-lily/index.html` |
| Vídeo   | `https://skylineip.s3.sa-east-1.amazonaws.com/Tour+Virtual/COUTINHO/galeria-maison-lily/video-gallery.html` |
| Script  | `https://skylineip.s3.sa-east-1.amazonaws.com/Tour+Virtual/COUTINHO/galeria-maison-lily/inject.js` |

**S3 path:** `s3://skylineip/Tour Virtual/COUTINHO/galeria-maison-lily/`

---

## Estrutura de arquivos

```
galeria/  (coutinho / maison lily)
├── index.html              ← galeria de imagens + plantas (auto-suficiente)
├── video-gallery.html      ← player do vídeo de apresentação
├── inject.js               ← loader leve para injeção no 3DVista
├── deploy.ps1               ← deploy para o S3 (Windows; normaliza case + sync)
├── generate_thumbs.ps1      ← gerador de thumbnails (Windows/GDI+, sem deps)
└── assets/
    ├── imagens/
    │   ├── AREA COMUM/                  ← fachada, academia, barbearia, brinquedoteca,
    │   │                                   espaço pet, gourmet, hall social, home office,
    │   │                                   mini quadra, piscina, playground, reunião,
    │   │                                   salão de beleza, salão de festa
    │   ├── AP 165M - TERM 3/            ← unidade 165m² — terraço 3
    │   ├── AP 250M - TERM 1 E 2/        ← unidade 250m² — terraços 1 e 2
    │   └── RECORTES/                    ← closes/detalhes decorativos
    ├── plantas/                         ← subsolo, garagem, pilotis, mezaninos,
    │                                       pavimento tipo, lazer, coberturas
    └── thumbs/                          ← gerado automaticamente (espelha a árvore, .jpg)
```

> **Pastas = categorias.** Os nomes das pastas em `assets/` definem as categorias/subcategorias.
> Mantenha-os **minúsculos e sem espaços/acentos** (o S3 é case-sensitive) — nomes de
> arquivo com acento precisam estar em Unicode NFC (precomposto), não NFD.

---

## Categorias da galeria

### Modo `imagens`

| Categoria | Label | Sub-categorias | Pasta |
|-----------|-------|----------------|-------|
| `fachada` | Fachada | — | `assets/imagens/AREA COMUM/` (arquivos `1. FACHADA*`) |
| `apartamentos` | Apartamentos | AP 165m² — Terraço 3 · AP 250m² — Terraços 1 e 2 | `assets/imagens/AP 165M - TERM 3/`, `assets/imagens/AP 250M - TERM 1 E 2/` |
| `areas-comuns` | Áreas Comuns | Academia · Barbearia · Brinquedoteca · Espaço Pet · Gourmet · Hall Social · Home Office · Mini Quadra · Piscina · Playground · Sala de Reunião · Salão de Beleza · Salão de Festa | `assets/imagens/AREA COMUM/` |
| `detalhes` | Detalhes | — | `assets/imagens/RECORTES/` |

### Modo `plantas`

Categoria única `plantas`, sem sub-filtro — mostra todos os pavimentos em grade
(subsolo 1/2, garagem, pilotis, mezanino 01/02, pavimento tipo, pavimento tipo 3 quartos,
lazer, cobertura 1/2). Pasta: `assets/plantas/`.

> Há duas versões do "Pavimento Tipo 1" (Fev/26 e Mar/26) — ambas foram mantidas na
> galeria distinguidas por subtítulo. Remova a desatualizada se for o caso.

---

## Thumbnails

Todo `src` da grade usa um `thumb` leve (**900 px, JPEG q82**, achatado sobre branco).
Geração **sem dependências** no Windows via GDI+:

```powershell
./generate_thumbs.ps1          # gera apenas os que faltam / desatualizados
./generate_thumbs.ps1 -Force   # regenera todos
```

O script espelha `assets/` em `assets/thumbs/` (sempre `.jpg`, caminho minúsculo),
ignora `thumbs/`, `video/` e o PDF.

---

## Deploy AWS S3

> **Windows:** use o script pronto `./deploy.ps1`. Requer **AWS CLI** + `aws configure` (região `sa-east-1`).

```powershell
./deploy.ps1              # sync completo (sem o vídeo) + cache-control no HTML/JS
./deploy.ps1 -WithVideo   # inclui o vídeo de apresentação (~200 MB) — necessário na 1ª vez / ao trocá-lo
./deploy.ps1 -QuickHtml   # atualiza só index.html e inject.js (rápido)
```

O vídeo fica de fora do sync padrão (o `--exclude` também o protege do `--delete`),
então uma vez enviado com `-WithVideo` ele permanece no S3 nas próximas sincronizações.

> **`--cache-control "no-cache,no-store,must-revalidate"`** no HTML/JS — garante que o
> 3DVista nunca sirva uma versão cacheada da galeria ou do script.

---

## Integração 3DVista

### Passo 1 — Loader (JavaScript global do projeto)

```js
(function(){
  var s = document.createElement('script');
  s.src = 'https://skylineip.s3.sa-east-1.amazonaws.com/Tour+Virtual/COUTINHO/galeria-maison-lily/inject.js?v=' + Date.now();
  document.head.appendChild(s);
})();
```

> **`?v=` + `Date.now()`** — cache-busting: força o browser a baixar sempre a versão mais
> recente do script, evitando que o 3DVista sirva uma versão antiga em cache.

### Passo 2 — Acionar nos hotspots/botões

```js
GaleriaImagens(1);      // abre galeria de imagens   · GaleriaImagens(0) fecha
GaleriaPlantas(1);      // abre galeria de plantas    · GaleriaPlantas(0) fecha
AbrirGaleriaVideos(1);  // abre o vídeo de apresentação · AbrirGaleriaVideos(0) fecha
```

> `video-gallery.html` aponta para `assets/video/apresentacao.mp4`, que ainda não foi
> enviado neste projeto — envie o arquivo do vídeo teaser nessa pasta antes de usar
> `AbrirGaleriaVideos` / `-WithVideo`.

---

## Cores e tipografia

Tema **cinza-chumbo + dourado** da Coutinho Maison Lily:

| Token CSS | Valor | Papel |
|-----------|-------|-------|
| `--bg` (fundo) | `#646461` | Cinza-chumbo — fundo principal |
| `--surface` | `#706E6B` | Superfície de card (skeleton) |
| `--dark` (foreground) | `#F5F1EA` | Texto/ícones — quase-branco |
| `--accent` | `#2B2A27` | Chumbo quase-preto — destaque / estado ativo |
| Pílulas de filtro | `#F9D59D` | Dourado — fundo padrão dos botões de filtro |
| Lightbox | `#1c1c1a` | Quase-preto — palco da imagem/vídeo |
| Fonte títulos | Cormorant Garamond | — |
| Fonte UI | Inter | — |

> Cards de plantas mantêm **fundo branco** (legibilidade do desenho), com texto chumbo.
