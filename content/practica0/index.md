+++
date = '2026-02-20T21:20:16-08:00'
draft = false
title = 'Practica0: Manejo de repositorios'
+++

# Reporte de Prácticas: Creación de Portafolio con Markdown, Git y Hugo

**Enlaces del Proyecto:**
* **Repositorio en GitHub:** [Inserta aquí tu enlace, ej: https://github.com/tu-usuario/tu-repo]
* **Página Estática (GitHub Pages):** [Inserta aquí tu enlace, ej: https://tu-usuario.github.io/tu-repo]

---

## Descripcion de la practica

El objetivo de esta práctica integral es dominar las herramientas fundamentales para la documentación y publicación de proyectos en el desarrollo de software. A lo largo de tres sesiones, se exploró el uso de Markdown para la redacción de contenido, el sistema Git y la plataforma GitHub para el control de versiones, y finalmente, el generador de sitios estáticos Hugo junto con GitHub Actions para automatizar y publicar un portafolio profesional en la nube.

## Subsecciones

1. **Primera sesión:** Sintaxis y uso de Markdown.
2. **Segunda sesión:** Uso de Git y GitHub.
3. **Tercera sesión:** Páginas estáticas con Hugo y GitHub Actions.

---

## Desarrollo de la practica

### Primera sesión: Sintaxis y uso de Markdown

**¿Qué es Markdown?**
Markdown es un lenguaje de marcado ligero creado con el objetivo de maximizar la legibilidad y la facilidad de publicación. Permite escribir texto usando un formato de texto plano que es fácil de leer y escribir, y que luego puede ser convertido estructuralmente a HTML (o a otros formatos como PDF).

**¿Cómo se utiliza?**
Se utiliza principalmente para formatear archivos `README`, redactar mensajes en foros de discusión, crear documentación técnica y, como en este caso, escribir artículos para generadores de sitios estáticos. Solo requiere un editor de texto simple.

**Sintaxis Básica:**
* **Encabezados:** Se utiliza el símbolo `#`. (Ej: `# Título 1`, `## Título 2`).
* **Énfasis:** Para **negritas** se usan asteriscos dobles `**texto**`, y para *cursivas* un asterisco `*texto*`.
* **Listas:** Se usa `-` o `*` para listas desordenadas, y números `1.` para ordenadas.
* **Enlaces:** `[Texto del enlace](URL)`
* **Imágenes:** `![Texto alternativo](URL_de_la_imagen)`
* **Código:** Para bloques de código se usan tres comillas invertidas (\`\`\`).

### Segunda sesión: Uso de Git y GitHub

**¿Qué son Git y GitHub?**
* **Git** es un sistema de control de versiones distribuido que permite registrar los cambios en los archivos de un proyecto a lo largo del tiempo, facilitando el trabajo colaborativo y la recuperación de versiones anteriores.
* **GitHub** es una plataforma basada en la nube que aloja repositorios de Git. Facilita compartir el código, colaborar con otros desarrolladores y publicar proyectos.

**Comandos Esenciales de Git:**
* `git init`: Inicializa un nuevo repositorio local.
* `git status`: Muestra el estado de los archivos (modificados, sin seguimiento, listos para commit).
* `git add <archivo>` o `git add .`: Añade los cambios al área de preparación (staging).
* `git commit -m "Mensaje"`: Guarda los cambios preparados con un mensaje descriptivo.
* `git push`: Envía los commits locales al repositorio remoto (GitHub).
* `git clone <url>`: Descarga un repositorio remoto a tu computadora.

**Cómo crear un repositorio y subir información:**
1. Se crea un nuevo repositorio en la página de GitHub.
2. En la terminal local, dentro de la carpeta del proyecto, se ejecuta `git init`.
3. Se vincula el repositorio local con el remoto usando `git remote add origin <URL>`.
4. Se agregan los archivos (`git add .`), se guardan (`git commit -m "commit inicial"`) y se suben a la nube (`git push -u origin main`).

### Tercera sesión: Páginas estáticas con Hugo y GitHub Actions

**¿Qué son Hugo y GitHub Actions?**
* **Hugo** es uno de los generadores de sitios estáticos de código abierto más populares y rápidos del mundo. Está escrito en Go y toma tus archivos de configuración y contenido (en Markdown) para renderizar páginas HTML completas.
* **GitHub Actions** es una plataforma de integración y despliegue continuo (CI/CD) que permite automatizar flujos de trabajo (workflows) directamente desde tu repositorio en GitHub.

**Creación de un sitio estático en Hugo y subida a GitHub:**
1. Se ejecuta el comando `hugo new site nombre-del-sitio`.
2. Se descarga un tema (como Ananke) y se configura en el archivo `hugo.toml`.
3. Se crea contenido usando `hugo new posts/mi-primer-post.md`.
4. Se prueba localmente con `hugo server`.
5. Una vez listo, se sube todo el código fuente al repositorio de GitHub utilizando los comandos de Git vistos en la sesión anterior.

**Configurar GitHub Actions para publicar en GitHub Pages:**
En lugar de compilar el sitio manualmente, se utiliza GitHub Actions. 
1. Se crea un archivo de flujo de trabajo (workflow) en la ruta `.github/workflows/hugo.yaml` dentro del repositorio.
2. Este archivo le da instrucciones a GitHub para que, cada vez que se haga un `push` a la rama principal, instale Hugo en un servidor temporal, construya los archivos HTML estáticos y los publique automáticamente en la rama o entorno de **GitHub Pages**.
3. Finalmente, se activa GitHub Pages en la configuración del repositorio, resultando en un sitio web público y funcional.

---

## Conclusiones

La integración de Markdown, Git, Hugo y GitHub Actions representa un flujo de trabajo moderno y altamente eficiente para cualquier desarrollador. Markdown agiliza la redacción sin preocuparnos por complejas etiquetas de diseño; Git y GitHub aseguran que nuestro trabajo esté respaldado y versionado de manera segura; y Hugo junto a GitHub Actions automatizan el tedioso proceso de construir y publicar la web. Con dominar estas herramientas, la creación de portafolios y documentación técnica se convierte en un proceso rápido, limpio y profesional.