# Instrucciones para el Asistente / Agentes

Este proyecto es un sitio web estático desarrollado con **Hugo** y el tema **Blowfish**.

## Directivas principales
- **Skill obligatorio:** Utilizar siempre el skill `blowfish` para cualquier tarea relacionada con la configuración, creación de contenido, layouts, partials, shortcodes o estilos del sitio.
- **Estructura de configuración:** Todos los archivos de configuración residen en `config/_default/` (`hugo.toml`, `params.toml`, `languages.<lang>.toml`, `menus.<lang>.toml`, etc.). No usar archivos de configuración redundantes en la raíz.
- **Contenido del blog:** Los artículos del blog deben organizarse bajo `content/posts/<slug>/` con soporte bilingüe (`index.es.md` e `index.en.md`).
- **Personalizaciones del tema:** Realizar sobreescrituras en `layouts/`, `assets/` y hooks oficiales como `layouts/partials/extend-head.html` y `layouts/partials/extend-footer.html`.
