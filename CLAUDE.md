# CLAUDE.md

Bu fayl Claude Code uchun loyiha haqidagi asosiy ma'lumotlarni saqlaydi.

## Loyiha haqida

Islambek Shamuratovning **shaxsiy portfolio sayti** — front-end dasturchi, pedagog va
moliyaviy mutaxassis sifatida o'zini tanishtiruvchi bir sahifali (single-page) sayt.
Sayt ikki tilli: **o'zbek (default)** va **rus** (`RU` tugmasi orqali almashtiriladi).

## Texnologiyalar

Sof statik sayt — build qadami yoki npm yo'q. Hammasi 3 ta fayl:

- **HTML5** — semantik tuzilish, `lang="uz"`
- **CSS3** — CSS o'zgaruvchilari (`:root` da `--primary`, `--gradient-main`, `--border`,
  `--ease` va h.k.), responsive (mobil breakpoint ~768px), wave SVG bo'limlar orasida
- **Vanilla JavaScript** — framework yo'q
- **Tashqi kutubxonalar (CDN orqali):**
  - Font Awesome 6.5.0 — ikonkalar (`fab fa-telegram`, `fas fa-*` va h.k.)
  - GSAP 3.12.5 + ScrollTrigger — scroll animatsiyalari (`gs-fade-up`, `gs-stagger`,
    `gs-slide-right`, `gs-slide-left` klasslari)
  - Typed.js 2.1.0 — hero bo'limidagi yozuv effekti (`#typed`)
  - Google Fonts — Inter shrifti

## Fayl tuzilishi

```
index.html              # Yagona sahifa: navbar, hero, about, skills, projects, services, contact, footer
style.css               # Barcha uslublar (CSS o'zgaruvchilari :root da)
script.js               # Til almashtirish, burger menyu, GSAP animatsiyalar, Typed.js, statistika hisoblagich
50x.html                # Caddy server xatolik sahifasi (5xx uchun, ikki tilli)
assets/images/profile.jpg   # Profil rasmi
.github/workflows/deploy.yml # GitHub Actions: main'ga push'da VPS'ga rsync (avto-deploy)
Dockerfile              # Eski (nginx:alpine) — endi ishlatilmaydi, rollback uchun qolgan
.gitignore              # .claude/settings.local.json va OS fayllarini chiqaradi
```

## Asosiy bo'limlar (index.html)

- **navbar** (`#navbar`) — logo, nav-links, nav-right (Telegram ikonkasi + `RU` til tugmasi + burger)
- **hero** (`#hero`) — ism, Typed.js yozuvi, bio, ijtimoiy havolalar, aylanuvchi SVG orbital vizual
- **about** (`#about`) — 3 ta karta (Front-end, Buxgalteriya, Pedagogika)
- **skills** (`#skills`) — progress-bar ko'nikmalar + texnologiya teglari + statistika
- **projects** (`#projects`) — 3 ta loyiha kartasi (hozircha placeholder havolalar `href="#"`)
- **services** (`#services`) — 4 ta xizmat kartasi
- **contact** (`#contact`) — email, Telegram, joylashuv + ijtimoiy kartalar
- **footer** — copyright + ijtimoiy havolalar

## Ko'p tillilik (i18n)

Tarjima qilinadigan matnlar `data-i18n="kalit"` atributi bilan belgilanadi. Tarjimalar
**`script.js`** ichidagi obyektda saqlanadi. Yangi matn qo'shganda: HTML'ga `data-i18n`
qo'shing **va** `script.js` dagi ham `uz`, ham `ru` obyektiga kalitni qo'shing.

## Aloqa / ijtimoiy havolalar

- Email: `boomboxuz@gmail.com`
- Telegram: `https://t.me/boomboxuz`
- Instagram: `https://instagram.com/boomboxuz`
- YouTube: `https://youtube.com/@boomboxuz`

## Deploy (VPS — asosiy, jonli)

Hetzner VPS jonli muhit (`remote-control.uz`). Arxitektura **Caddy** ga o'tkazildi (Docker + Nginx olib tashlandi):
- Domain: `remote-control.uz`, Cloudflare DNS orqali (A record, **Proxied** — origin IP shu sababli ataylab oshkor qilinmaydi, kerak bo'lsa mahalliy xotiradan/foydalanuvchidan so'rang)
- Server: VPS'da **Caddy** (`/etc/caddy/Caddyfile`) statik fayllarni to'g'ridan-to'g'ri serve qiladi: `root /var/www/personal-website`, `:80`→`:443` redirect, HTTP/2+HTTP/3
- TLS: **Cloudflare Origin CA** sertifikat `/etc/ssl/cloudflare/{cert.pem,key.pem}` (2041-gacha, `root:caddy`, kalit `640`). Cloudflare SSL/TLS rejimi **Full (strict)**
- SSH: shaxsiy foydalanuvchi `boomboxuz` (sudo bor); deploy uchun alohida **sudo'siz `deploy`** foydalanuvchi (faqat `/var/www/personal-website` egasi). Root va parol bilan kirish o'chirilgan — faqat SSH key
- Eski Docker konteyner (`personal-website`) to'xtatilgan (`--restart=no`), Nginx `disable` qilingan — rollback uchun qolgan, keraksiz bo'lsa tozalash mumkin

## Avto-deploy (GitHub Actions → VPS)

`main` branch'ga push qilinganda (sayt fayllari o'zgarsa) `.github/workflows/deploy.yml` ishga tushadi va `rsync --delete` orqali fayllarni VPS'dagi `/var/www/personal-website` ga yetkazadi.
- Ulanish: `deploy` foydalanuvchi + maxsus SSH deploy kaliti
- GitHub repo sekretlari: `DEPLOY_SSH_KEY` (private key), `DEPLOY_HOST`, `DEPLOY_USER`, `DEPLOY_KNOWN_HOSTS`
- rsync `--exclude`: `.git`, `.github`, `CLAUDE.md`, `Dockerfile`, `.gitignore`, `README.md` (web bo'lmagan fayllar webroot'ga tushmaydi)
- Qo'lda ishga tushirish: `gh workflow run "Deploy to VPS"` yoki Actions tab'dan; holat: `gh run list`

## Deploy (GitHub Pages)

- **Repo:** https://github.com/boomboxuz-hash/personal-website (public)
- **Jonli sayt:** https://boomboxuz-hash.github.io/personal-website/
- **Manba:** `main` branch, ildiz (`/`) papka — legacy Pages build, HTTPS majburiy
- Push qilingach Pages avtomatik qayta build qiladi (~1 daqiqa). Brauzerda ko'rish uchun
  **Ctrl+F5** (kesh tozalash).

## Ish jarayoni (workflow)

O'zgartirishdan keyin:

```powershell
git add .
git commit -m "izoh"
git push
```

## Muhit eslatmalari (bu mashina)

- OS: Windows 10, shell PowerShell 7. Claude Code dagi `!` prefiksi esa **bash** (git-bash) da ishlaydi.
- `gh` (GitHub CLI) o'rnatilgan: `C:\Program Files\GitHub CLI\gh.exe` (PATH ga qo'shilgan, oddiy
  `gh` buyrug'i ishlaydi). Push uchun kredensial: `gh auth setup-git` orqali Git Credential
  Manager o'rniga `gh` autentifikatsiyasidan foydalanish sozlangan (global git config).
- Git identifikatori: `boomboxuz-hash` / `boomboxuz@gmail.com`. GitHub login: `boomboxuz-hash` (HTTPS).
- Fayllarda LF→CRRF ogohlantirishi normal (Windows), e'tibor bermaslik mumkin.
