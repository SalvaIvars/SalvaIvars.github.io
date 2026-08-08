# Diseño — Web personal de Salva (Hugo + Bear)

**Fecha:** 2026-08-08
**Estado:** Aprobado

## Objetivo

Crear una página web personal minimalista para **Salva**, inspirada en
[vickiboykis.com](https://vickiboykis.com/): un blog técnico con sección
"sobre mí" y proyectos, 100% estático, sin backend.

## Contexto

- Directorio de trabajo: `personal-web/` (vacío, nuevo proyecto).
- Referencia: vickiboykis.com — blog técnico minimalista hecho con Hugo + tema Bear.

## Stack

- **Generador:** Hugo (binario local, sin instalación global requerida).
- **Tema:** `hugo-bearblog` (mismo tema que el sitio de referencia).
- **Contenido:** archivos Markdown.
- **Salida:** sitio estático generado con `hugo`.

## Estructura general

- Página de inicio con el nombre **Salva** como título principal.
- Blog con lista de posts ordenados por fecha (título + fecha).
- Página **Sobre mí** (About).
- Página **Proyectos**.
- Página de **Tags** para navegar por tema.
- Feed RSS automático de Hugo.

## Navegación

`Salva · Blog · Proyectos · Sobre mí · RSS`

## Contenido y temas

- Posts escritos en Markdown en `content/blog/`.
- Tags predefinidos: `programación`, `inteligencia artificial`, `proyectos`.
- Cada post puede incluir varios tags.

## Configuración

- Archivo de configuración `hugo.toml`.
- Personalización de título, descripción, autor y enlaces de navegación.

## Workflow de publicación

- Desarrollo local: `hugo server`.
- Generación del sitio estático: `hugo`.

## Criterios de éxito

1. El sitio se genera sin errores con `hugo`.
2. La home muestra el blog con posts ordenados por fecha.
3. Las páginas About, Proyectos y Tags funcionan.
4. El RSS se genera correctamente.
5. El estilo es minimalista, similar a la referencia.
