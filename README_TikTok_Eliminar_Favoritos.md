# TikTok — Eliminador de favoritos

Guía para usar los scripts en **Google Chrome en Mac**.

## 1. Extensión: Tampermonkey

La extensión que debes utilizar para instalar los scripts como userscripts es **Tampermonkey**.

Web oficial: https://www.tampermonkey.net/

### Instalación

1. Abre Google Chrome.
2. Instala Tampermonkey.
3. Pulsa el icono de Tampermonkey.
4. Selecciona **Crear un nuevo script**.
5. Borra todo el contenido.
6. Pega el script.
7. Guarda con **⌘ + S**.
8. Abre TikTok Web: https://www.tiktok.com/
9. Recarga TikTok.

---

# 2. Último script: Favoritos + carruseles

Este es el último script preparado.

### Funciones

- Panel **▶ Iniciar / ■ Parar**.
- Detecta el marcador amarillo de Favoritos.
- Lo pulsa para quitar el favorito.
- Comprueba que deja de estar amarillo.
- Cuenta los eliminados.
- Pasa a la siguiente publicación.
- Intenta recuperarse cuando encuentra publicaciones de tipo carrusel.
- Comprueba que realmente ha cambiado la publicación antes de continuar.

### Instalación

En Tampermonkey:

**Crear un nuevo script → borrar todo → pegar el script → ⌘ + S**

Después abre TikTok y pulsa **▶ Iniciar**.

---

# 3. Script alternativo: Auto Eliminar Favoritos

Este script **no tiene botón**. Empieza automáticamente al ejecutarse.

Pégalo en Tampermonkey:

```javascript
// ==UserScript==
// @name         TikTok Auto Eliminar Favoritos
// @namespace    tiktok-auto-favoritos
// @version      2.0
// @description  Elimina favoritos automáticamente y pasa al siguiente vídeo
// @match        https://www.tiktok.com/*
// @run-at       document-idle
// @grant        none
// ==/UserScript==

(async function () {

    'use strict';

    const ESPERA_FAVORITO = 2500;
    const ESPERA_SIGUIENTE = 3500;

    let eliminados = 0;
    let ultimoVideo = location.href;

    const esperar = ms =>
        new Promise(resolve => setTimeout(resolve, ms));

    function info(el) {
        return [
            el.getAttribute?.('aria-label'),
            el.getAttribute?.('title'),
            el.getAttribute?.('data-e2e'),
            el.getAttribute?.('data-testid'),
            el.getAttribute?.('data-test'),
            el.textContent
        ]
        .filter(Boolean)
        .join(' ')
        .toLowerCase();
    }

    function buscarFavorito() {
        const elementos = [
            ...document.querySelectorAll(
                'button, [role="button"], div[tabindex="0"]'
            )
        ];

        for (const el of elementos) {
            const t = info(el);

            if (
                t.includes('favorite') ||
                t.includes('favourite') ||
                t.includes('favorito') ||
                t.includes('bookmark') ||
                t.includes('guardar') ||
                t.includes('guardado')
            ) {
                return el;
            }
        }

        for (const svg of document.querySelectorAll('svg')) {
            const contenido = svg.outerHTML.toLowerCase();

            if (
                contenido.includes('bookmark') ||
                contenido.includes('favorite') ||
                contenido.includes('favourite')
            ) {
                const boton = svg.closest(
                    'button,[role="button"],div[tabindex="0"]'
                );

                if (boton) return boton;
            }
        }

        for (const el of elementos) {
            const svg = el.querySelector('svg');

            if (!svg) continue;

            const partes = [
                svg,
                ...svg.querySelectorAll('*')
            ];

            for (const parte of partes) {
                const estilo = getComputedStyle(parte);

                const colores = [
                    estilo.color,
                    estilo.fill,
                    estilo.stroke
                ];

                for (const color of colores) {
                    if (!color) continue;

                    const rgb = color.match(/\d+/g);

                    if (!rgb || rgb.length < 3) continue;

                    const r = Number(rgb[0]);
                    const g = Number(rgb[1]);
                    const b = Number(rgb[2]);

                    if (
                        r > 180 &&
                        g > 130 &&
                        b < 120 &&
                        g > b * 1.5
                    ) {
                        return el;
                    }
                }
            }
        }

        return null;
    }

    function buscarSiguiente() {
        const elementos = [
            ...document.querySelectorAll(
                'button,[role="button"],div[tabindex="0"]'
            )
        ];

        for (const el of elementos) {
            const t = info(el);

            if (
                t.includes('next') ||
                t.includes('siguiente')
            ) {
                return el;
            }
        }

        return null;
    }

    function siguiente() {
        const boton = buscarSiguiente();

        if (boton) {
            boton.click();
            console.log('➡️ Siguiente');
            return;
        }

        document.dispatchEvent(
            new KeyboardEvent('keydown', {
                key: 'ArrowDown',
                code: 'ArrowDown',
                keyCode: 40,
                which: 40,
                bubbles: true
            })
        );

        console.log('⬇️ Siguiente');
    }

    async function encontrarFavorito() {
        for (let i = 0; i < 20; i++) {
            const boton = buscarFavorito();

            if (boton) return boton;

            await esperar(500);
        }

        return null;
    }

    console.clear();

    console.log(
        '⭐ TikTok Auto Eliminar Favoritos iniciado'
    );

    await esperar(2500);

    while (true) {
        console.log('🔎 Buscando favorito...');

        const favorito = await encontrarFavorito();

        if (!favorito) {
            console.log(
                '❌ No encuentro Favoritos. Reintentando...'
            );

            await esperar(2000);
            continue;
        }

        console.log('⭐ Favorito encontrado');

        favorito.click();

        eliminados++;

        console.log(
            `✅ Favorito eliminado: ${eliminados}`
        );

        await esperar(ESPERA_FAVORITO);

        siguiente();

        await esperar(ESPERA_SIGUIENTE);
    }

})();
```

---

# 4. Ejecutarlo directamente desde la consola de Chrome

También puedes ejecutar el código directamente sobre TikTok sin Tampermonkey.

### Abrir la consola

En Chrome para Mac:

**⌘ + ⌥ + J**

Se abrirá **Console / Consola**.

### Si Chrome muestra “Allow pasting”

Chrome puede bloquear el pegado de código en la consola.

Si aparece el aviso, **escribe manualmente**:

```text
allow pasting
```

y pulsa **Enter**.

Después podrás pegar el código.

> Importante: `allow pasting` debe escribirse manualmente cuando Chrome lo solicite.

### Pasos

1. Abre TikTok.
2. Pulsa **⌘ + ⌥ + J**.
3. Si Chrome lo pide, escribe manualmente `allow pasting`.
4. Pulsa Enter.
5. Pega el script.
6. Pulsa Enter.

---

# 5. Diferencias

| Método | Extensión | Botón | Inicio |
|---|---|---|---|
| Userscript con panel | Tampermonkey | Sí | Manual |
| Auto Eliminar Favoritos | Tampermonkey | No | Automático |
| Consola Chrome | Ninguna | No | Al pulsar Enter |

### Uso habitual

**Chrome → Tampermonkey → script con panel → TikTok → ▶ Iniciar**

### Ejecución puntual

**Chrome → TikTok → ⌘ + ⌥ + J → Consola → script → Enter**

---

## Precauciones

Los scripts realizan clics automáticamente y modifican tus favoritos. Es recomendable probar primero con pocos vídeos.

TikTok puede cambiar su interfaz, atributos HTML o navegación. Si eso ocurre, alguno de los detectores puede necesitar ajustes.
