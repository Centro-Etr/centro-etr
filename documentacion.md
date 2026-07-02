# Documentación — Plataforma de Ventas Centro ETR

## ¿Qué se construyó?

Una tienda en línea completa en un único archivo HTML (`index.html`) que funciona directamente en el navegador, sin necesidad de servidor, base de datos externa ni conexión a internet (salvo para abrir WhatsApp al enviar un pedido). Sigue el mismo enfoque técnico que `Inventario de mi negocio/inventario.html`.

---

## Tecnologías utilizadas

| Tecnología | Uso |
|---|---|
| HTML5 | Estructura de la página |
| CSS3 (variables, grid, flexbox) | Diseño y estilos (tema claro, colores de marca Centro ETR) |
| JavaScript (Vanilla) | Lógica de catálogo, carrito, checkout y panel admin |
| localStorage / sessionStorage | Persistencia del catálogo, el carrito y la sesión de administrador |

No se utilizaron librerías externas ni frameworks. Todo está en un solo archivo.

---

## Estructura del archivo

```
index.html
├── <style>              → Estilos CSS completos (tema claro, verde/azul/marino)
├── #view-tienda         → Vista pública: header, hero, planes, equipos, contacto, footer
├── #view-admin          → Vista de administración (oculta hasta iniciar sesión)
├── #cart-drawer         → Panel lateral del carrito
├── #checkout-overlay    → Modal para datos del cliente antes de enviar por WhatsApp
├── #admin-gate-overlay  → Modal de clave de acceso al panel admin
├── #admin-product-overlay → Modal para crear/editar productos
├── #confirm-overlay     → Diálogo de confirmación genérico (vaciar carrito, eliminar producto, restaurar catálogo)
├── #toast               → Notificaciones emergentes
├── <script id="catalog-data">      → Catálogo inicial (12 productos) en JSON
├── <script id="admin-config-data"> → Clave de acceso por defecto en JSON
└── <script>              → Toda la lógica JavaScript
```

---

## Modelo de datos de cada producto

```json
{
  "id": "p06",
  "tipo": "plan_internet",
  "nombre": "Plan Gold+",
  "velocidadMbps": 250,
  "precio": 1900,
  "tipoPrecio": "mensual",
  "requiereInstalacionAparte": true,
  "costoInstalacion": 1600,
  "notaInstalacion": "",
  "equipoIncluido": "Router 6G",
  "descripcionLarga": "Texto de venta persuasivo...",
  "bullets": ["Juegos en línea sin lag", "Streaming 4K sin interrupciones"],
  "badge": "Más popular",
  "activo": true,
  "orden": 60,
  "imagen": "data:image/jpeg;base64,...",
  "iconoFallback": null,
  "variantes": null
}
```

- `tipo`: `"plan_internet"` o `"equipo"` — decide en qué sección del catálogo aparece y cómo se comporta en el carrito.
- `tipoPrecio`: `"mensual"` o `"pago_unico"` — nunca se suman ambos tipos en un mismo total.
- `requiereInstalacionAparte` + `costoInstalacion` + `notaInstalacion`: si `costoInstalacion` tiene un número, se muestra como costo fijo ("Instalación: RD$X"); si es `null`, se muestra el texto de `notaInstalacion` (costo variable, "a confirmar por WhatsApp").
- `imagen`: foto del producto/plan incrustada en base64 (`data:image/...;base64,...`), o `null` si no hay.
- `iconoFallback`: emoji que se muestra en lugar de la foto cuando `imagen` es `null` (ej. 🏠 para Plan Pueblo).
- `variantes`: array opcional `[{"nombre":"...","precio":...}]` para productos con varios niveles de precio (ej. el paquete Internet + Cable con niveles de IPTV). Si existe, la tarjeta muestra un selector y el precio cambia según la opción elegida.
- `activo`: permite ocultar un producto de la tienda sin borrarlo (soft-delete).
- `orden`: controla el orden de aparición dentro de su sección.

---

## Catálogo incluido (12 productos)

**Planes de internet por fibra óptica (todos simétricos), con su costo de instalación (pago único, aparte de la mensualidad):**
- Plan Pueblo — 5 Mbps — RD$500/mes — instalación RD$1,000
- Plan Básico — 50 Mbps — RD$850/mes — instalación RD$1,000
- Plan Básico 5G — 60 Mbps — RD$1,000/mes — instalación RD$1,200
- **Internet + Cable (IPTV)** — 50 Mbps + TV — selector de nivel: IPTV Básico RD$1,100/mes, IPTV Plus RD$1,200/mes, IPTV Ultra RD$1,500/mes — instalación RD$1,500
- Plan Premium+ — 100 Mbps — RD$1,350/mes — instalación RD$1,600
- Plan Gold — 150 Mbps — RD$1,650/mes — instalación RD$1,600
- Plan Gold+ — 250 Mbps — RD$1,900/mes — instalación RD$1,600
- Plan Gold++ — 400 Mbps — RD$2,400/mes — instalación RD$1,600
- Plan Diamond — 500 Mbps — RD$3,000/mes — instalación RD$1,600

**Equipos:** Repetidor WiFi Mercusys 3 Antenas (RD$1,100 + instalación a confirmar), Repetidor WiFi Mercusys 4 Antenas (RD$1,300 + instalación a confirmar), Mini UPS para Router/ONT (RD$1,100 pago único, sin mensualidad, sin instalación).

Cada plan (excepto Plan Pueblo, que usa el ícono 🏠) y cada equipo tiene su foto real incrustada en el archivo. Las fotos de planes son los volantes de marketing completos; las fotos de equipos están recortadas para mostrar solo el producto.

---

## Tarjetas desplegables

Cada tarjeta de plan/equipo se ve compacta por defecto: imagen, nombre, velocidad (Mbps, solo planes), precio (con selector de nivel si el producto tiene variantes) y el botón de acción, siempre visibles. Al tocar la tarjeta se despliega el detalle: equipo incluido, costo de instalación, descripción de venta y "ideal para". Cada tarjeta se expande/colapsa de forma independiente (estado en memoria, `expandedCards`, no persiste al recargar la página). Los controles interactivos (botón, selector, stepper de cantidad) detienen la propagación del clic para no disparar el despliegue al usarlos.

## Banner de promoción

Sección "¡Pásalo pa'l Fuerte!" entre el hero y "Planes de internet": programa de referidos (recomienda a alguien, si se instala con Centro ETR ganas 50% de descuento en tu próxima factura), con la imagen de marketing tal cual y un botón de WhatsApp con mensaje prellenado. Es contenido estático, no forma parte del catálogo ni del carrito.

---

## Carrito de pedidos

- Solo se permite **un plan de internet activo a la vez** (incluye el paquete Internet + Cable): elegir un plan nuevo reemplaza automáticamente al anterior (con una notificación), ya que un cliente contrata un solo servicio de internet.
- Los equipos se agregan con cantidad libre (botones +/-), sin límite de stock (venta bajo pedido).
- El carrito muestra **líneas separadas**: "Cargo mensual recurrente", "Instalación del plan (pago único)" y "Equipos (pago único)" — nunca se suman entre sí, para no confundir un cargo recurrente con uno de una sola vez.
- El carrito vive en `localStorage['etr_cart_v1']`, separado del catálogo, y **nunca** se incluye en la página exportada desde el panel admin.

---

## Pedido por WhatsApp

Al hacer clic en "Enviar pedido por WhatsApp", se pide Nombre y Sector/Dirección (obligatorios) más teléfono alterno y comentario (opcionales). Se arma un mensaje de texto con el resumen del pedido y se abre:

```
https://wa.me/18297835887?text=...
```

(829-783-5887 en formato internacional). El carrito se vacía automáticamente después de enviar.

---

## Panel de administración

- Se accede desde el enlace discreto **"Acceso administrador"** en el pie de página.
- Clave por defecto: `CentroETR2026` (se puede cambiar desde el propio panel, en "Cambiar clave de acceso").
- **Esta clave es solo una barrera básica contra accesos accidentales, no un sistema de seguridad real** — cualquier persona con conocimientos técnicos podría verla en el código fuente.
- Permite: crear, editar y eliminar productos/servicios, activar/desactivar productos sin borrarlos, y restaurar el catálogo a los valores de fábrica.

### Cómo publicar tus cambios de precios/productos

Como este archivo no tiene servidor, los cambios que hagas en el panel admin se guardan **solo en el navegador donde los hiciste**. Para que los clientes los vean:

1. Entra al panel admin y edita/agrega los productos que necesites.
2. Haz clic en **"⬇ Descargar página actualizada"**.
3. Se descarga un archivo `index.html` con tu catálogo editado ya incluido como los datos de fábrica.
4. Reemplaza el `index.html` de tu repositorio de GitHub con este archivo nuevo (ver sección siguiente) para publicar los cambios.

---

## Cómo publicar en GitHub Pages

La carpeta `Plataforma Centro ETR` ya es un repositorio git local (con `index.html` y este documento). Para que la tienda tenga una dirección web real y gratuita:

1. **Crear cuenta en [github.com](https://github.com)** si todavía no tienes una.
2. **Crear un repositorio nuevo**, público (el plan gratuito de GitHub Pages requiere repos públicos). Nombre sugerido: `centro-etr-tienda`. No marcar "Add a README" (ya tenemos archivos locales).
3. **Conectar el repositorio local con el remoto** (GitHub te muestra estos comandos exactos al crear el repo vacío; en general son):
   ```
   git remote add origin https://github.com/<tu-usuario>/centro-etr-tienda.git
   git branch -M main
   git push -u origin main
   ```
   La primera vez puede pedirte iniciar sesión en una ventana del navegador.
4. **Activar GitHub Pages**: en el repo, ve a *Settings → Pages*, en "Branch" selecciona `main` y carpeta `/ (root)`, y guarda.
5. Espera 1-2 minutos y visita `https://<tu-usuario>.github.io/centro-etr-tienda/` — ahí debe cargar la tienda.
6. **Para publicar cambios futuros**: descarga el `index.html` actualizado desde el panel admin (paso anterior) y reemplázalo en el repositorio. Puedes hacerlo de dos formas:
   - **Sin usar git**: en github.com, entra al repo, abre `index.html`, botón de lápiz (editar) o "Add file → Upload files" para subir el nuevo archivo, y confirma ("Commit changes").
   - **Usando git** (si lo tienes instalado): reemplaza el archivo local y corre `git add index.html`, `git commit -m "Actualizar catálogo"`, `git push`.
   - En ambos casos, GitHub Pages actualiza el sitio publicado en 1-2 minutos.

Un dominio propio (ej. `centroetr.com`) se puede agregar más adelante desde *Settings → Pages → Custom domain*, sin perder nada de lo ya publicado.

---

## Diseño visual

- **Paleta de marca priorizada verde > blanco > azul**: el verde es el color dominante en superficies grandes (botón del carrito, hero, footer, botones principales), el blanco/gris muy claro sigue siendo la base de fondo y tarjetas, y el azul queda como acento secundario (hovers, badges, número de velocidad). El azul marino (`#0B2B5C`) se usa únicamente como tinta de texto/encabezados, no como bloque de color.
- **Logo real** de Centro ETR (recortado con fondo transparente a partir del material de marca) en el header, el panel admin y el footer, más como favicon de la pestaña del navegador.
- Tarjetas de planes con foto real, velocidad destacada, precio mensual, costo de instalación y distintivos ("Nuevo", "Más popular", "Tope de gama", "Sin mensualidad", "Paquete").
- Sección de **Preguntas frecuentes** (instalación y métodos de pago/contrato, ambas remiten a WhatsApp) y tarjeta de **Zona de cobertura** (todo Elías Piña) en la sección de contacto.
- Meta-etiquetas Open Graph para que el link se vea con logo y descripción al compartirse (el `og:image` usa una imagen embebida en base64, que algunas plataformas no renderizan hasta que la página esté publicada en una URL real).
- Responsive — se adapta a pantallas pequeñas (móvil).

---

## Cómo usar el archivo

1. Abrir `index.html` en cualquier navegador moderno (Chrome, Edge, Firefox).
2. No requiere instalación ni conexión a internet (salvo para abrir WhatsApp al enviar un pedido).
3. Los cambios del panel admin se guardan automáticamente en el navegador donde se hicieron.
4. Para publicar cambios a los clientes, usar el flujo de "Descargar página actualizada" descrito arriba.

---

## Control de versiones y respaldo

La carpeta `Plataforma Centro ETR` tiene su propio repositorio git (separado de otros proyectos en `Proyecto de claude`, como el inventario). Esto permite:
- Ver el historial de cambios (`git log`) y revertir si algo se rompe por una edición manual.
- Servir como respaldo real, independiente del navegador de cualquier dispositivo.
- Conectarse a GitHub para publicar la tienda (ver sección "Cómo publicar en GitHub Pages").

Además, se recomienda conservar la carpeta `C:\Users\lirex\Empresa Centro ETR` (fotos originales sin procesar) como respaldo, ya que las imágenes incrustadas en `index.html` están redimensionadas/recortadas y no podrían regenerarse en mejor calidad si se pierden los originales.

## Posibles mejoras futuras

- Historial de pedidos enviados (actualmente cada pedido solo vive en el chat de WhatsApp).
- Dominio propio (ej. `centroetr.com`) una vez publicada en GitHub Pages.
- Integrar un mapa embebido de la ubicación en la sección de contacto.
- Agregar imagen para el Plan Pueblo cuando esté disponible (hoy usa el ícono 🏠).
- Permitir subir imágenes de productos directamente desde el panel admin (hoy solo se editan desde el archivo/código).
- Reducir el peso del archivo (~1.4 MB) si el tiempo de carga en conexiones lentas se vuelve un problema.
