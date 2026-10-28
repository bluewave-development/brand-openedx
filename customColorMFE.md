# Guía de Relación entre Design Tokens (`brand-openedx`) y Variables CSS de Paragon en MFEs

Este documento explica cómo los colores personalizados definidos en este repositorio (`brand-openedx`) se transforman en variables CSS y clases utilitarias de **Paragon**, y cómo utilizarlos correctamente dentro de cualquier Micro-Frontend (MFE) de Open edX.

---

## 1. Regla de Transformación (JSON → Variable CSS)

Paragon utiliza **Style Dictionary** para compilar los archivos JSON de tokens en variables CSS (Custom Properties). La convención para construir el nombre de la variable CSS consiste en agregar el prefijo `--pgn-` y unir cada nivel de la jerarquía del JSON con un guion (`-`):

```text
--pgn- + <categoría> + - + <elemento> + - + <subelemento/variante>
```

Por ejemplo, para el bloque definido en `paragon/tokens/themes/light/_stub.json`:

```json
{
  "color": {
    "primary": {
      "base": {
        "$value": "#0076be",
        "$description": "color base de Blue Wave",
        "$type": "color"
      }
    }
  }
}
```

La ruta `color` → `primary` → `base` se convierte directamente en:

```css
--pgn-color-primary-base: #0076BEFF;
```

---

## 2. Tabla de Mapeo de Colores de Blue Wave

A continuación se detalla la relación exacta entre las propiedades personalizadas en `paragon/tokens/themes/light/_stub.json` y las variables CSS resultantes en Paragon (`variables.css` / `light.min.css`):

| Ruta en `_stub.json` | Valor Blue Wave | Variable CSS Base | Alias Nivel 500 | Escala Autogenerada | Descripción |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `color.primary.base` | `#0076be` | `--pgn-color-primary-base` | `--pgn-color-primary-500` | `--pgn-color-primary-100` a `--pgn-color-primary-900` | Color base primario de Blue Wave |
| `color.secondary.base` | `#FF0000` | `--pgn-color-secondary-base` | `--pgn-color-secondary-500` | `--pgn-color-secondary-100` a `--pgn-color-secondary-900` | Color secundario de Blue Wave |
| `color.brand.base` | `#002F6C` | `--pgn-color-brand-base` | `--pgn-color-brand-500` | `--pgn-color-brand-100` a `--pgn-color-brand-900` | Color de marca (botones e identidad) |

---

## 3. Escalas de Color Autogeneradas (Niveles `100` a `900`)

Cuando defines únicamente el color `base` en `_stub.json`, el compilador de tokens de Paragon (`paragon build-tokens`) **calcula automáticamente toda la paleta de variaciones** mezclando tu color base con blanco (niveles claros) y con negro (niveles oscuros):

| Nivel | Variable CSS (Ejemplo Primario) | Cómo lo calcula Paragon | Uso recomendado |
| :--- | :--- | :--- | :--- |
| **100** | `var(--pgn-color-primary-100)` | 94% blanco + 6% color base | Fondos muy suaves, *badges* o *hover* sutil |
| **200** | `var(--pgn-color-primary-200)` | 75% blanco + 25% color base | Bordes suaves, separadores o fondos secundarios |
| **300** | `var(--pgn-color-primary-300)` | 50% blanco + 50% color base | Elementos deshabilitados o bordes activos |
| **400** | `var(--pgn-color-primary-400)` | 25% blanco + 75% color base | Acentos intermedios |
| **500** | `var(--pgn-color-primary-500)` | **100% `var(--pgn-color-primary-base)`** | **Color principal exacto (`#0076be`)** |
| **600** | `var(--pgn-color-primary-600)` | 10% negro + 90% color base | Variación ligeramente más oscura |
| **700** | `var(--pgn-color-primary-700)` | 20% negro + 80% color base | Estado `:hover` en botones y enlaces |
| **800** | `var(--pgn-color-primary-800)` | 25% negro + 75% color base | Estado `:focus` / bordes de alto contraste |
| **900** | `var(--pgn-color-primary-900)` | 30% negro + 70% color base | Estado `:active` (clic presionado) o cabeceras oscuras |

> Esta misma escala del `100` al `900` existe para `secondary` (`--pgn-color-secondary-100` ... `900`) y para `brand` (`--pgn-color-brand-100` ... `900`).

---

## 4. ¿Por qué en el repositorio de Paragon (`variables.css`) aparece otro color?

1. **En el repositorio oficial de Paragon (`styles/css/themes/light/variables.css`):**
   Se encuentran los valores por defecto de Open edX sin marca (por ejemplo, `--pgn-color-primary-base: #0A3055FF;` en la línea 653 y `--pgn-color-brand-base: #9D0054FF;` en la línea 654).
2. **Al compilar `brand-openedx` (`make build` / `npm run build`):**
   * `paragon build-tokens` fusiona los tokens originales de Paragon con los de `./paragon/tokens/` de este repositorio, reemplazando los valores por defecto con los de `_stub.json`.
   * `paragon build-scss` empaqueta el resultado en `dist/paragon/light.min.css` y `dist/paragon/core.min.css`.
3. **Al publicar en `bluewave-brand-assets` e inyectar en Tutor (`PARAGON_THEME_URLS`):**
   El navegador descarga el `light.min.css` personalizado de Blue Wave, donde `--pgn-color-primary-base` ya tiene el valor `#0076BEFF`.

---

## 5. Cómo usar los colores en un MFE

### A. Mediante Variables CSS en archivos `.scss` o `.css`

En cualquier hoja de estilos de un MFE (por ejemplo `src/App.scss` o estilos de componentes), referencia directamente las variables CSS de Paragon:

```scss
.custom-header-banner {
  // Color primario exacto de Blue Wave (#0076be)
  background-color: var(--pgn-color-primary-base);
  color: var(--pgn-color-white);
  border-bottom: 2px solid var(--pgn-color-brand-base); // #002F6C

  &:hover {
    // Tono oscuro autogenerado a partir de #0076be
    background-color: var(--pgn-color-primary-700);
  }
}

.custom-card {
  background-color: var(--pgn-color-primary-100);
  border: 1px solid var(--pgn-color-primary-200);

  .custom-card-title {
    color: var(--pgn-color-primary-base);
  }

  .custom-card-badge {
    background-color: var(--pgn-color-secondary-base); // #FF0000
  }
}
```

### B. Mediante Clases Utilitarias de Paragon / Bootstrap en JSX

Paragon también conecta estas variables CSS con sus clases utilitarias listas para usar en tus componentes React (`className`):

| Propósito | Clases para `primary` (`#0076be`) | Clases para `brand` (`#002F6C`) | Clases para `secondary` (`#FF0000`) |
| :--- | :--- | :--- | :--- |
| **Color de fondo** | `bg-primary`, `bg-primary-100` ... `bg-primary-900` | `bg-brand`, `bg-brand-100` ... `bg-brand-900` | `bg-secondary`, `bg-secondary-100` ... `bg-secondary-900` |
| **Color de texto** | `text-primary`, `text-primary-100` ... `text-primary-900` | `text-brand`, `text-brand-100` ... `text-brand-900` | `text-secondary`, `text-secondary-100` ... `text-secondary-900` |
| **Color de borde** | `border-primary` | `border-brand` | `border-secondary` |
| **Variante de `<Button>`** | `<Button variant="primary">` / `"outline-primary"` | `<Button variant="brand">` / `"outline-brand"` | `<Button variant="secondary">` / `"outline-secondary"` |

---

## 6. Nota importante para Desarrollo Local (`npm start`)

Cuando trabajas un MFE en modo local e importas temporalmente los estilos de Paragon en `src/App.scss`:

* Si importas `@import "~@openedx/paragon/dist/light.min.css";` desde `node_modules/@openedx/paragon`, verás los colores por defecto de Open edX (`#0A3055`) porque ese paquete aún no tiene tus overrides de `brand-openedx`.
* Para visualizar en local los colores reales de Blue Wave (`#0076be`, `#002F6C`, `#FF0000`), asegúrate de que el MFE cargue el `light.min.css` compilado desde `brand-openedx` / `bluewave-brand-assets` (ya sea mediante `@edx/brand` instalado localmente o apuntando a los archivos compilados).
* En ambos escenarios, el nombre de la variable que debes usar en tu código siempre será **`var(--pgn-color-primary-base)`**.

