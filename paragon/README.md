# Guía de Personalización de Estilos y Tokens — Blue Wave (`paragon/`)

Este directorio contiene la configuración de **Design Tokens** y hojas de estilo **SCSS** que personalizan la apariencia de **Open edX (Paragon 23+)** para **Blue Wave** sin necesidad de modificar directamente el código fuente de cada Microfrontend (MFE).

---

## 1. Arquitectura de Archivos

```text
paragon/
├── tokens/
│   ├── core/                     # Tokens base (tipografía, espaciados, breakpoints)
│   └── themes/
│       └── light/
│           └── _stub.json        # Tokens de colores del tema claro de Blue Wave
├── images/                       # Imágenes de respaldo (ej. card-imagecap-fallback.png)
├── _fonts.scss                   # Importación de fuentes web (@import)
├── _custombw.scss                # Reglas CSS/SCSS personalizadas de Blue Wave
├── core.scss                     # Punto de entrada que ensambla tokens + _custombw.scss
└── README.md                     # Esta documentación
```

---

## 2. Cómo Funciona la Personalización de Tokens de Colores

En Paragon 23+, los colores de la marca se definen mediante **Design Tokens** en formato JSON ([Style Dictionary](https://styledictionary.com/)) dentro del archivo [`tokens/themes/light/_stub.json`](./tokens/themes/light/_stub.json).

### Colores configurados actualmente

```json
{
    "color": {
        "primary": {
            "base": {
                "$value": "#002F6C",
                "$description": "Color base de Blue Wave (azul fuerte).",
                "$type": "color"
            }
        },
        "secondary": {
            "base": {
                "$value": "#00A9E0",
                "$description": "Color secundario de Blue Wave (azul claro).",
                "$type": "color"
            }
        },
        "brand": {
            "base": {
                "$value": "#002F6C",
                "$description": "Color principal de botones y elementos de marca.",
                "$type": "color"
            }
        }
    }
}
```

### ¿Cómo se transforman en variables CSS?

Al compilar los tokens, Paragon convierte automáticamente la jerarquía del JSON en **variables CSS nativas** con el prefijo `--pgn-` y calcula su escala de tonalidades (`100` a `900`):

| Ruta en JSON (`_stub.json`) | Variable CSS generada | Valor base |
| :--- | :--- | :--- |
| `color.primary.base` | `var(--pgn-color-primary-base)` | `#002F6C` |
| `color.secondary.base` | `var(--pgn-color-secondary-base)` | `#00A9E0` |
| `color.brand.base` | `var(--pgn-color-brand-base)` | `#002F6C` |

> **Regla importante:** En [`_custombw.scss`](./_custombw.scss) **nunca uses colores hexadecimales fijos** (como `#002F6C`). Utiliza siempre las variables CSS `var(--pgn-color-primary-base)`, `var(--pgn-color-secondary-base)`, `var(--pgn-color-white)`, etc.

---

## 3. Cómo se Ejecuta y Despliega la Personalización de Estilos

El flujo completo para que los cambios de tokens, estilos y logos se reflejen en la plataforma **Open edX** consta de **3 pasos**:

### Paso 1: Hacer el build del paquete (`brand-openedx`)

> **⚠️ Importante:** El proceso de compilación (`build`) **debe ejecutarse desde Ubuntu (Linux / WSL)** por temas de compatibilidad en la compilación (el `Makefile` utiliza comandos nativos de Unix como `make`, `rm -rf`, `mkdir -p` y `cp -r`).

Ejecuta la compilación desde tu terminal de **Ubuntu** en la raíz de `brand-openedx` para transformar los tokens JSON y los archivos SCSS (`core.scss` y `_custombw.scss`) en los archivos finales dentro de la carpeta `/dist`:

* **Compilación completa (Recomendado):**
  Limpia `/dist`, compila los tokens JSON, compila el SCSS y copia los assets (`.svg`, `.png`, `.ico`).
  ```bash
  npm run build
  # Equivalente a: make build
  ```
* **Otros comandos útiles durante desarrollo:**
  * `npm run build-tokens`: Compila únicamente los Design Tokens (`./paragon/tokens/` → `./paragon/build`).
  * `npm run build-scss`: Compila únicamente el SCSS (`core.scss` + `_custombw.scss` → `dist/core.min.css` y `dist/light.min.css`).
  * `npm run watch:build`: Modo observador que recompila automáticamente al guardar cambios.

### Paso 2: Publicar los artefactos en el repositorio con GitHub Pages
Una vez finalizado el build en la carpeta `/dist`, copia los siguientes archivos generados hacia el repositorio de GitHub que tiene habilitado **GitHub Pages** (el cual actúa como CDN de recursos estáticos):

* **Hojas de estilo compiladas:**
  * `dist/core.min.css` (y `dist/core.css`)
  * `dist/light.min.css` (y `dist/light.css`)
* **Logos e íconos de marca:**
  * `dist/logo.svg`, `dist/logo-trademark.svg`, `dist/logo-white.svg`, `dist/favicon.ico`
  * `dist/paragon/images/card-imagecap-fallback.png`

Realiza el `commit` y `push` en dicho repositorio para que **GitHub Pages** publique la nueva versión de los recursos.

### Paso 3: Integración en Open edX mediante Plugin (`PARAGON_THEME_URLS`)
En la instancia de **Open edX** se cuenta con un **plugin** donde están configurados los enlaces públicos de **GitHub Pages** hacia los recursos CSS (`core.min.css` y `light.min.css`) y los logos.
* Los Microfrontends (MFEs) obtienen estas URLs en tiempo de ejecución desde la configuración del plugin, por lo que al actualizarse los archivos en GitHub Pages los estilos se reflejan en Open edX sin necesidad de reconstruir cada MFE.

---

## 4. Orden de Prioridad para Personalizar Estilos

Para garantizar un **mantenimiento sencillo** y evitar conflictos al actualizar versiones de Open edX, sigue siempre este orden de prioridad antes de hacer un cambio visual:

### Prioridad 1: Design Tokens (`tokens/themes/light/_stub.json`)
* **Cuándo usarlo:** Cuando quieres cambiar un color global de la marca, tipografía base o bordes en toda la plataforma.
* **Ventaja:** No requiere escribir CSS ni tocar ningún MFE; Paragon actualiza todos sus componentes automáticamente.

### Prioridad 2: Overrides de Clases Nativas en `_custombw.scss` (Secciones 2 y 3)
* **Cuándo usarlo:** Cuando un componente específico de Paragon o de un MFE necesita un estilo particular y **ya tiene una clase nativa en el DOM** (por ejemplo: `.dropdown-item.active`, `.course-card-title`, `.course-list-title`, `.pgn__action-row a`).
* **Ventaja:** **No necesitas modificar el archivo `.jsx` del MFE**. Toda la personalización vive centralizada en `brand-openedx`.

### Prioridad 3: Clases Utilitarias `.bw-*` en `_custombw.scss` (Sección 1)
* **Cuándo usarlo:** Solo cuando el elemento en el MFE es una etiqueta genérica (ej. un `<div>` sin clase identificadora) y es indispensable asignarle una clase en el `className` del `.jsx`.
* **Ventaja:** La definición del estilo sigue centralizada en `_custombw.scss` y puede reutilizarse en otros MFEs.

### Prioridad 4 (Último recurso): Archivos `.scss` locales dentro del MFE
* **Cuándo usarlo:** Únicamente durante pruebas locales rápidas en el MFE antes de migrar la clase definitiva hacia `_custombw.scss`.

---

## 5. Nomenclatura y Estructura de `_custombw.scss`

### Convención de nombres para clases propias
Todas las clases nuevas creadas para Blue Wave deben utilizar el prefijo **`bw-`** al inicio en formato *kebab-case*:

* **Estructura:** `.bw-<categoría>-<modificador>`
* **Ejemplos:**
  * Fondos: `.bw-bg-primary`, `.bw-bg-brand`, `.bw-bg-primary-hover`
  * Textos: `.bw-text-primary`, `.bw-card-title`
  * Bordes: `.bw-line-border`
  * Botones / Menús: `.bw-btn-custom`, `.bw-btn-secondary`, `.bw-menu-secondary`

*(Nota: En `_custombw.scss` se conservan alias de clases anteriores con sufijo `*-bw` como `.line-border-bw` o `.btn-primary-bw` por retrocompatibilidad, pero toda clase nueva debe iniciar con `.bw-`).*

### Clasificación interna de `_custombw.scss`

El archivo [`_custombw.scss`](./_custombw.scss) está dividido en **3 secciones**:

1. **`1. CLASES UTILITARIAS GLOBALES (Prefijo .bw-*)`**
   Clases reutilizables que se invocan explícitamente desde el `className` de un componente.
2. **`2. OVERRIDES GLOBALES DE PARAGON Y COMPONENTES COMPARTIDOS`**
   Estilos que sobrescriben clases nativas de componentes transversales a todos los MFEs (como `@edx/frontend-component-header`, `@edx/frontend-component-footer` o componentes base de `@openedx/paragon`).
3. **`3. OVERRIDES POR MFE (Clases nativas de cada MFE)`**
   Subsecciones organizadas por cada Microfrontend (`3.1 Learner Dashboard`, `3.2 Learning`, `3.3 Account / Profile`, etc.) donde se sobrescriben las clases nativas exclusivas de cada aplicación.

