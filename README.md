tarea
# safari-marketplace

[![CI](https://github.com/rcuello/safari-marketplace/actions/workflows/ci.yml/badge.svg)](https://github.com/rcuello/safari-marketplace/actions/workflows/ci.yml)

Monorepo didáctico para enseñar **sistemas distribuidos** con una aplicación
real: una tienda de tecnología con su panel de administración, una API, una
base de datos compartida y un scraper que trae productos de seis tiendas
colombianas. Todo se empaqueta en imágenes de Docker, se prueba en cada push
con GitHub Actions y se despliega en un VPS con Dokploy.

> **Empieza aquí:** [`docs/README.md`](docs/README.md) es la guía paso a paso
> para desplegar **tu fork** en tu propio dominio, con HTTPS.

---

## Qué hay adentro

```text
            navegador
                │ https
                ▼
   Traefik (Dokploy) ──▶ proxy (Caddy) ──┬──▶ tienda   (Next.js)
                                         ├──▶ admin    (Next.js)
                                         └─ /backend ─▶ api-lb (Caddy) ──▶ api × N (NestJS)
                                                                              │
   scraper (Scrapy + Playwright) ──── upsert de productos ───▶ PostgreSQL ◀───┘
```

| Pieza | Qué hace | Carpeta |
|---|---|---|
| Tienda | El sitio público: catálogo, carrito, checkout, cuentas | [`apps/shop`](apps/shop) |
| Admin | Panel para administradores, dueños de tienda y su staff | [`apps/admin/rest`](apps/admin/rest) |
| API | REST con autenticación JWT y permisos por rol; escalable detrás de un balanceador | [`apps/api/rest`](apps/api/rest) |
| Capa de datos | Prisma sobre PostgreSQL, con un repositorio por agregado | [`packages/db`](packages/db) |
| Esquema y datos iniciales | El DDL comentado y el seed del catálogo | [`db`](db) |
| Scraper | Seis spiders (Alkosto, Compulago, Compuworking, Éxito, Falabella, Tauret) que escriben en la misma tabla que lee la tienda | [`services/scraper-worker`](services/scraper-worker) |
| Proxy | Caddy: reparte el tráfico entre tienda, admin y API, y balancea la API | [`docker`](docker) |

La tienda y el admin parten de la plantilla Pickbazar, adaptada para leer de
PostgreSQL en lugar de datos de prueba.

## Por dónde empezar

**Quiero desplegarlo** en internet, con mi dominio → [`docs/README.md`](docs/README.md).
Fork, CI propio, dominio en Cloudflare, VM en Azure for Students y Dokploy, en
seis fases con capturas.

**Quiero correrlo en mi equipo** para programar → [`apps/README.md`](apps/README.md).
Todo se orquesta con [`just`](https://github.com/casey/just) desde la raíz:

```bash
just doctor        # comprueba Node, yarn, Python, los .env y las dependencias
just setup         # crea los .env desde plantillas e instala dependencias (~15 min)
just db-up         # PostgreSQL en Docker, con el esquema y el seed
just db-build      # construye la capa de datos (packages/db)

just api-dev       # API      → http://localhost:9001/api  (Swagger en /docs)
just shop-dev      # tienda   → http://localhost:3003
just admin-dev     # admin    → http://localhost:3002
just verify        # comprueba que los tres responden con contenido real
```

Arranca la API primero: la tienda y el admin la consultan al renderizar en el
servidor. Para levantar todo en contenedores, como en producción:
`just stack-up`. `just` a secas lista todas las tareas.

## Stack

| Capa | Tecnología |
|---|---|
| Frontend | Next.js 16 (pages router), React 19, Tailwind CSS 4 |
| API | NestJS 12 (ESM), TypeScript 6, Vitest |
| Datos | PostgreSQL 16, Prisma 7 con `@prisma/adapter-pg` |
| Scraper | Python 3.11+, Scrapy, Playwright, psycopg 3 |
| Contenedores | Docker, Docker Compose, Caddy 2 |
| CI/CD | GitHub Actions, Docker Hub, Dokploy (Traefik + Let's Encrypt) |
| Herramientas | Node 22.22.3 (`.nvmrc`), yarn, `just` |

## Calidad y CI

Cada push a `main` y cada pull request corren en GitHub Actions
([`.github/workflows/ci.yml`](.github/workflows/ci.yml)):

1. **Calidad:** lint, typecheck y tests de todo el monorepo, y los builds de la
   API, la tienda y el admin. Cada paso es un step propio, con una tabla de
   resumen al final.
2. **Imágenes:** construye las cinco imágenes de Docker (API, tienda, admin,
   `db-init` y `proxy`). Desde `main` las publica en Docker Hub con la etiqueta
   `sha-<commit>`; un fork las publica en **su** Docker Hub si lo configuró
   ([`docs/github/`](docs/github/README.md)).

En local es la misma lista: `just check` corre los pasos de calidad,
`just check <id>` uno solo y `just check-full` agrega los builds.

> La insignia de arriba muestra el CI de este repositorio. En tu fork, el
> estado está en la pestaña **Actions**.

## Documentación

| Dónde | Qué |
|---|---|
| [`docs/README.md`](docs/README.md) | Guía del estudiante: del fork al despliegue |
| [`apps/README.md`](apps/README.md) | Arranque local, puertos, variables, imágenes y detalles de cada app |
| [`db/README.md`](db/README.md) | El modelo de datos del catálogo |
| [`packages/db/README.md`](packages/db/README.md) | La capa de datos con Prisma |
| [`services/scraper-worker/README.md`](services/scraper-worker/README.md) | El scraper y sus spiders |
| [`docs/product/`](docs/product/README.md) | Las épicas e historias de usuario con las que se construyó el proyecto (flujo spec-driven) |

## Licencia

[MIT](LICENSE).
