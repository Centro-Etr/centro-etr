# Historial del proyecto — Plataforma de Ventas Centro ETR

Este documento registra, en orden cronológico, todo lo que se hizo para construir y publicar la tienda en línea de Centro ETR. Es un registro de proceso (qué se hizo y por qué); para la referencia técnica de cómo funciona el sitio hoy, ver `documentacion.md`.

---

## Fase 1 — Construcción inicial de la plataforma

**Objetivo:** crear una página donde los clientes de Centro ETR pudieran elegir planes de internet y equipos, con buenas descripciones de venta.

- Se tomó como base el patrón técnico de un proyecto hermano ya existente del usuario (`Inventario de mi negocio/inventario.html`): un único archivo HTML + CSS + JavaScript, sin frameworks, sin servidor, con `localStorage` como persistencia.
- Se extrajo toda la información real de la empresa (nombre, dirección, WhatsApp, catálogo, precios) de material de marketing que el usuario compartió en la carpeta `C:\Users\lirex\Empresa Centro ETR`. No se inventó ningún dato.
- Se construyó `centro-etr-tienda.html` con:
  - **Catálogo inicial de 11 productos**: 8 planes de internet por fibra óptica (Pueblo, Básico, Básico 5G, Premium+, Gold, Gold+, Gold++, Diamond) y 3 equipos (Repetidor 3 antenas, Repetidor 4 antenas, Mini UPS), cada uno con una descripción de venta persuasiva redactada específicamente para la tienda.
  - **Carrito de compras**: un solo plan de internet activo a la vez (elegir uno nuevo reemplaza al anterior), equipos con cantidad libre.
  - **Checkout por WhatsApp**: el cliente llena sus datos (nombre, dirección) y se genera un mensaje prellenado que abre WhatsApp hacia el número del negocio (829-783-5887).
  - **Panel de administración**: protegido con una clave básica, permite crear/editar/eliminar productos sin tocar código, y un botón para "descargar" una versión actualizada del HTML con los cambios ya incrustados (ya que al no haber servidor, es la forma de "publicar" cambios).
- Se creó `documentacion.md` como referencia técnica del proyecto.

---

## Fase 2 — Rediseño visual y datos reales

**Objetivo:** mejorar la apariencia y usar información 100% real del negocio.

- **Logo real**: se localizó el archivo de logo original (`Empresa Centro ETR/logo/...jpeg`), se recortó con PowerShell + `System.Drawing` de .NET (sin necesitar herramientas externas), se le quitó el fondo blanco (transparencia), y se incrustó en el header, panel admin y footer.
- **Paleta de colores**: se repriorizaron los colores de marca para que el **verde** fuera el color dominante (antes predominaba un azul marino), manteniendo el **blanco** como base y el **azul** como acento secundario.
- **Fotos reales**: el usuario proporcionó fotos organizadas por producto/plan en `Empresa Centro ETR/Imagenes de los productos/`. Los volantes de los planes se usaron tal cual (sin recortar); las fotos de los equipos se recortaron para mostrar solo el producto.
- **Costos de instalación reales**: el usuario proporcionó los montos exactos (Básico y Pueblo RD$1,000, Básico 5G RD$1,200, Premium+ en adelante RD$1,600), que se agregaron a cada plan y se separaron claramente de la mensualidad en el carrito y en el mensaje de WhatsApp.
- **Nuevo producto — Internet + Cable (IPTV)**: paquete con el Plan Básico incluido más un selector de nivel de TV (IPTV Básico RD$1,100, Plus RD$1,200, Ultra RD$1,500/mes), instalación fija RD$1,500. Esto requirió extender el modelo de datos del catálogo para soportar "variantes" de precio.
- Se agregaron: sección de **Preguntas Frecuentes** (instalación y pagos, ambas remiten a WhatsApp), tarjeta de **Zona de Cobertura** (todo Elías Piña), favicon, texto alternativo en imágenes y meta-etiquetas Open Graph.

---

## Fase 3 — Tarjetas desplegables y promoción

**Objetivo:** limpiar la interfaz y sumar una promoción activa del negocio.

- Se rediseñaron las tarjetas de planes/equipos para mostrarse **compactas por defecto** (imagen, nombre, velocidad, precio y botón) y **desplegar el detalle al tocarlas** (descripción, "ideal para", costos de instalación), cada una de forma independiente.
- Se agregó el banner de la promoción **"¡Pásalo pa'l Fuerte!"** (programa de referidos: 50% de descuento en la próxima factura por cada recomendado que se instale), con la imagen real de marketing y un botón de WhatsApp.

---

## Fase 4 — Diagnóstico de confiabilidad y preparación para publicar

**Objetivo:** responder si la plataforma era confiable y prepararla para tener una dirección web real.

- Se hizo un diagnóstico: el código en sí es robusto (archivo estático, sin servidor que pueda "caerse"), pero existían dos puntos débiles reales: **no estaba publicada en ningún lado** y **no había respaldo/control de versiones**.
- Se decidió junto al usuario: publicar con **GitHub Pages** (gratis, confiable, resuelve ambos problemas a la vez), comenzando con una dirección gratuita (sin dominio propio por ahora).
- Se renombró el archivo principal de `centro-etr-tienda.html` a **`index.html`** (requisito de GitHub Pages para servir la página automáticamente).
- Se modificó el botón "Descargar página actualizada" del panel admin para que descargue directamente como `index.html`.
- Se inicializó un **repositorio git local**, con alcance únicamente a la carpeta `Plataforma Centro ETR` (independiente de otros proyectos del usuario), y se hizo el primer commit.
- Se documentó en `documentacion.md` una guía paso a paso para publicar en GitHub Pages.

---

## Fase 5 — Publicación en GitHub Pages

**Objetivo:** dejar la tienda visible en internet con una URL real.

1. El usuario creó su cuenta de GitHub y el repositorio público `centro-etr` (nombre elegido junto con el usuario).
2. Se conectó el repositorio local al remoto (`git remote add origin ...`) y se subió el proyecto (`git push`) — el primer intento requirió que el usuario completara un inicio de sesión/autorización en una ventana del navegador (Git Credential Manager), ya que esa parte no se puede automatizar sin las credenciales del usuario.
3. El usuario activó **GitHub Pages** en la configuración del repositorio (rama `main`, carpeta raíz), confirmado con una captura de pantalla.
4. **Problema encontrado:** los dos primeros intentos de despliegue fallaron con el error genérico de GitHub "Deployment failed, try again later". Se investigó usando la API pública de GitHub (sin necesitar contraseña) y se determinó que la causa más probable era que **la cuenta de GitHub se había creado apenas unos 30-40 minutos antes** — las cuentas nuevas a veces tienen una restricción temporal para publicar en Pages hasta verificar el correo o hasta que pase un poco de tiempo.
5. **Solución:** se esperó un poco más y se reintentó el despliegue con un nuevo commit — el tercer intento fue exitoso.
6. Se confirmó que el sitio carga correctamente en:

   ### 🌐 https://centro-etr.github.io/centro-etr/

---

## Fase 6 — Corrección de bug en móvil y subida de imágenes desde el panel admin

**Objetivo:** corregir un problema reportado en dispositivos móviles y añadir una función que faltaba en el panel de administración.

- El usuario reportó (con capturas de su celular) que el botón "Enviar pedido por WhatsApp" no hacía nada en móvil, y que en su lugar aparecía una franja de búsqueda de Android/Gboard sobre el botón. Comparando las capturas y confirmando que en PC sí funcionaba, se diagnosticó **selección de texto accidental**: sin `user-select:none`, un toque impreciso en móvil se interpretaba como selección de texto en vez de un tap, lo que activaba la barra de búsqueda de Android y bloqueaba el clic real. Se corrigió agregando `user-select:none`, `-webkit-touch-callout:none` y `touch-action:manipulation` a botones, tarjetas y demás elementos táctiles.
- El usuario también señaló que el panel de administración no permitía agregar una foto al crear/editar un producto (antes solo se podía incrustar una imagen editando el código). Se agregó un campo para **subir una foto directamente desde el formulario**, que se redimensiona automáticamente en el navegador (máximo 640px de ancho, usando `canvas`) antes de guardarse como parte del producto — sin necesitar ninguna herramienta externa.
- Ambos cambios se publicaron de inmediato con `git commit` + `git push`, y GitHub Pages actualizó el sitio en vivo en 1-2 minutos, sin necesitar repetir ningún paso de configuración ni crear nada nuevo.

---

## Estado final

- La tienda está **públicamente disponible** en la URL de arriba, con todo el catálogo, carrito, checkout por WhatsApp y panel administrativo funcionando.
- El proyecto tiene **control de versiones real** (git + GitHub), sirviendo también como respaldo.
- **Flujo para futuros cambios:** panel admin → editar → "Descargar página actualizada" (genera `index.html`) → reemplazar ese archivo en el repositorio de GitHub (vía web o con `git push`) → GitHub Pages actualiza el sitio en 1-2 minutos.
- Clave de acceso al panel admin: `CentroETR2026` (cambiable desde el propio panel).
- Pendientes opcionales para el futuro: dominio propio, imagen para el Plan Pueblo, banner de promoción adicional si se envían más fotos, reducir el peso del archivo si se vuelve un problema de carga.
