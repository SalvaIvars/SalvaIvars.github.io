# Web personal de Salva — Plan de implementación

> **Para trabajadores agentic:** SUB-SKILL OBLIGATORIA: usa superpowers:subagent-driven-development (recomendada) o superpowers:executing-plans para implementar este plan tarea por tarea. Los pasos usan sintaxis de checkbox (`- [ ]`).

**Goal:** Crear la web personal minimalista de Salva con Hugo + tema Bear, con blog, páginas "Sobre mí" y "Proyectos", tags y RSS.

**Architecture:** Sitio estático generado por Hugo (v0.164.0) usando el tema `hugo-bearblog` como submódulo git. Contenido en Markdown bajo `content/`. Configuración centralizada en `hugo.toml`. El flujo de publicación es local: `hugo server` para desarrollo y `hugo` para generar el sitio en `public/`.

**Tech Stack:** Hugo (extended) instalado vía Homebrew, tema `hugo-bearblog`, Markdown, TOML.

## Global Constraints

- Hugo instalado con `brew install hugo` (binario global vía Homebrew; es el binario "local" que usa la web).
- Versión de Hugo: v0.164.0 (última estable).
- Idioma del sitio: español (`languageCode = "es-ES"`).
- Nombre a mostrar: **Salva**.
- Menú principal: `Blog · Proyectos · Sobre mí · RSS` (además del título "Salva" que enlaza al inicio).
- Tags predefinidos: `programación`, `inteligencia artificial`, `proyectos`.
- No deshabilitar taxonomías (`disableKinds` NO incluye `taxonomy`) para que los tags tengan páginas.
- URLs por defecto de Hugo (posts en `/blog/<slug>/`, tags en `/tags/<tag>/`).
- El directorio de trabajo es `personal-web/` y ya es un repositorio git (rama `main`, con el spec commiteado).
- Debe ignorarse `public/` y `.hugo_build.lock` en git.

---

### Task 1: Instalar Hugo (Homebrew)

**Files:**
- (ninguno — instalación de sistema)

**Interfaces:**
- Consumes: nada.
- Produces: binario `hugo` disponible en el PATH (versión v0.164.0).

- [ ] **Step 1: Instalar Hugo extended**

```bash
brew install hugo
```

- [ ] **Step 2: Verificar la instalación**

```bash
hugo version
```

Expected: la salida empieza por `hugo v0.164.0` (o versión igual o superior a 0.140).

- [ ] **Step 3: Commit (sin cambios en repo — omitir)**

No hay cambios que commitecar en el repositorio en esta tarea.

---

### Task 2: Crear el scaffolding del sitio y añadir el tema

**Files:**
- Create: `archetypes/default.md`, `assets/`, `content/`, `data/`, `i18n/`, `layouts/`, `static/`, `themes/` (creados por `hugo new site`).
- Create: `themes/hugo-bearblog/` (submódulo).

**Interfaces:**
- Consumes: binario `hugo` de la Task 1.
- Produces: estructura de directorios de Hugo lista para configurar.

- [ ] **Step 1: Inicializar el sitio Hugo en el directorio actual**

```bash
hugo new site . --force
```

Expected: se crean `hugo.toml`, `archetypes/`, `assets/`, `content/`, `data/`, `i18n/`, `layouts/`, `static/`, `themes/`. No debe borrarse `docs/` ni `.git`.

- [ ] **Step 2: Añadir el tema como submódulo git**

```bash
git submodule add https://github.com/janraasch/hugo-bearblog.git themes/hugo-bearblog
```

- [ ] **Step 3: Verificar que el tema está presente**

```bash
ls themes/hugo-bearblog/layouts
```

Expected: listado con al menos `_default/` y `partials/`.

- [ ] **Step 4: Commit**

```bash
git add .gitmodules themes/hugo-bearblog hugo.toml archetypes assets content data i18n layouts static themes
git commit -m "chore: scaffold hugo site with hugo-bearblog theme"
```

---

### Task 3: Configurar hugo.toml

**Files:**
- Modify: `hugo.toml` (sobrescribir todo el archivo generado por `hugo new site`).

**Interfaces:**
- Consumes: scaffolding de la Task 2.
- Produces: `hugo.toml` con título, menú, idioma y markup listos para las tareas de contenido.

- [ ] **Step 1: Sobrescribir `hugo.toml` con la configuración del sitio**

```toml
baseURL = "https://example.com/"
languageCode = "es-ES"
title = "Salva"
author = "Salva"
copyright = "© Salva"
theme = "hugo-bearblog"
enableRobotsTXT = true

[params]
  description = "Web personal de Salva: programación, inteligencia artificial y proyectos."
  dateFormat = "2006-01-02"

[[menu.main]]
  name = "Blog"
  url = "/blog/"
  weight = 1

[[menu.main]]
  name = "Proyectos"
  url = "/proyectos/"
  weight = 2

[[menu.main]]
  name = "Sobre mí"
  url = "/sobre-mi/"
  weight = 3

[[menu.main]]
  name = "RSS"
  url = "/index.xml"
  weight = 4

[markup]
  [markup.highlight]
    style = "friendly"
    lineNos = true
    lineNumbersInTable = false
    codeFences = true
```

- [ ] **Step 2: Verificar que el sitio compila y muestra el título**

```bash
hugo
grep -o "Salva" public/index.html | head -1
```

Expected: build sin errores y el título "Salva" aparece en `public/index.html`.

- [ ] **Step 3: Commit**

```bash
git add hugo.toml
git commit -m "feat: configure site title, navigation and markup"
```

---

### Task 4: Crear las páginas principales

**Files:**
- Create: `content/_index.md`
- Create: `content/blog/_index.md`
- Create: `content/sobre-mi.md`
- Create: `content/proyectos.md`

**Interfaces:**
- Consumes: menú de la Task 3 (rutas `/blog/`, `/proyectos/`, `/sobre-mi/`).
- Produces: páginas navegables para los subagentes de contenido (Task 5).

- [ ] **Step 1: Crear `content/_index.md`**

```markdown
---
title: "Salva"
---
Bienvenido a mi rincón de la web.

Escribo sobre programación, inteligencia artificial y los proyectos en los que trabajo.

Puedes leer mis artículos en el [blog](/blog/).
```

- [ ] **Step 2: Crear `content/blog/_index.md`**

```markdown
---
title: "Blog"
---
```

- [ ] **Step 3: Crear `content/sobre-mi.md`**

```markdown
---
title: "Sobre mí"
---
Hola, soy Salva.

Me dedico a la programación y me interesa la inteligencia artificial. En esta web comparto notas, tutoriales y reflexiones sobre los temas que me apasionan.

[Contacto →](/)
```

- [ ] **Step 4: Crear `content/proyectos.md`**

```markdown
---
title: "Proyectos"
---
Aquí irán mis proyectos. Cada proyecto tendrá su propia entrada con descripción, tecnología y enlace al repositorio.
```

- [ ] **Step 5: Verificar la generación de las páginas**

```bash
hugo
test -f public/sobre-mi/index.html && test -f public/proyectos/index.html && echo OK
```

Expected: salida `OK` y build sin errores.

- [ ] **Step 6: Commit**

```bash
git add content
git commit -m "feat: add home, blog, about and projects pages"
```

---

### Task 5: Crear posts de ejemplo con tags

**Files:**
- Create: `content/blog/hola-mundo.md`
- Create: `content/blog/introduccion-a-la-ia.md`
- Create: `content/blog/mi-primer-proyecto.md`

**Interfaces:**
- Consumes: `content/blog/` y `hugo.toml` de las Tasks 3–4.
- Produces: posts listados en `/blog/` y páginas de tags en `/tags/`.

- [ ] **Step 1: Crear `content/blog/hola-mundo.md`**

```markdown
---
title: "Hola mundo"
date: 2026-08-08
tags: ["programación"]
---
Este es el primer artículo del blog. Aquí escribiré sobre programación: lenguajes, herramientas y buenas prácticas.

```python
print("Hola, mundo!")
```
```

- [ ] **Step 2: Crear `content/blog/introduccion-a-la-ia.md`**

```markdown
---
title: "Introducción a la IA"
date: 2026-08-08
tags: ["inteligencia artificial"]
---
Un repaso a los conceptos básicos de la inteligencia artificial y el aprendizaje automático, pensado para empezar desde cero.
```

- [ ] **Step 3: Crear `content/blog/mi-primer-proyecto.md`**

```markdown
---
title: "Mi primer proyecto"
date: 2026-08-07
tags: ["proyectos", "programación"]
---
La experiencia de crear mi primera aplicación completa: qué aprendí, qué haría distinto y qué viene después.
```

- [ ] **Step 4: Verificar posts, tags y RSS**

```bash
hugo
test -f public/blog/hola-mundo/index.html && echo "post OK"
ls public/tags/ | grep -i program && echo "tag OK"
test -f public/index.xml && echo "RSS OK"
```

Expected: `post OK`, `tag OK` (directorio de tag generado; Hugo conserva la tilde, por eso no se fuerza el slug sin acento), `RSS OK`. Build sin errores.

- [ ] **Step 5: Commit**

```bash
git add content/blog
git commit -m "feat: add sample blog posts with tags"
```

---

### Task 6: Ajustes finales y verificación completa

**Files:**
- Create: `.gitignore`

**Interfaces:**
- Consumes: todo lo anterior.
- Produces: sitio final reproducible y repo limpio.

- [ ] **Step 1: Crear `.gitignore`**

```gitignore
/public/
/.hugo_build.lock
```

- [ ] **Step 2: Generar el sitio completo y verificar todo**

```bash
hugo
test -f public/index.html && echo "home OK"
test -f public/blog/index.html && echo "blog OK"
test -f public/sobre-mi/index.html && echo "about OK"
test -f public/proyectos/index.html && echo "projects OK"
test -f public/index.xml && echo "RSS OK"
ls public/tags/ | grep -i program && echo "tags OK"
```

Expected: build sin errores y todas las comprobaciones muestran `OK`.

- [ ] **Step 3: Commit**

```bash
git add .gitignore
git commit -m "chore: ignore hugo build output"
```

---

## Self-Review

**1. Cobertura del spec:**
- Inicio con título "Salva" → Task 3 (título) + Task 4 (`_index.md`). ✅
- Blog con posts ordenados por fecha → Task 4 (`blog/_index.md`) + Task 5 (posts con `date`). ✅
- Página Sobre mí → Task 4. ✅
- Página Proyectos → Task 4. ✅
- Página de Tags → Task 5 (tags activos, taxonomía no deshabilitada). ✅
- RSS automático → Task 3 (habilitado por defecto) + Task 5 (verificación). ✅
- Navegación `Salva · Blog · Proyectos · Sobre mí · RSS` → Task 3 (menú). ✅
- Tags predefinidos → Task 5. ✅
- Configuración en `hugo.toml` → Task 3. ✅

**2. Placeholder scan:** No hay TBD/TODO. Todos los pasos contienen comandos y contenido concretos. ✅

**3. Consistencia de tipos/rutas:** Las rutas del menú (`/blog/`, `/proyectos/`, `/sobre-mi/`) coinciden con los archivos creados (`content/blog/`, `content/proyectos.md`, `content/sobre-mi.md`). Los posts apuntan a `/tags/<tag>/`, que genera Hugo por defecto. ✅
