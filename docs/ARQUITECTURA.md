# Arquitectura: cómo funciona tmsa.ar

tmsa.ar es un acortador de links **sin servidor ni base de datos**. Toda la "base de datos" es un archivo de texto (`_redirects`) versionado en este repo; el hosting (Netlify) lo lee en cada deploy y hace las redirecciones por nosotros.

```
 Persona              Repo GitHub               Netlify (sitio "tmsa-short")        Visitante
 ───────              ───────────               ───────────────────────────        ─────────
 tmsa <url> <código>  _redirects  ──push a main──▶  deploy automático  ──▶  https://tmsa.ar/<código>
 (o edición manual)   (1 línea por link)            (publica la carpeta)          └─ 301 ─▶ URL larga
```

## Piezas

| Pieza | Qué es | Dónde vive |
|---|---|---|
| `_redirects` | Lista de reglas `/código  URL-destino`. Es **la única fuente de verdad**. | Raíz de este repo |
| `package.json` | Declara `baseUrl` (`https://tmsa.ar`), el binario `tmsa` y la dependencia `netlify-shortener`. | Raíz |
| `cli.js` | Una línea: `require('netlify-shortener')`. Permite registrar el comando global `tmsa` con `npm link`. | Raíz |
| [`netlify-shortener`](https://github.com/kentcdodds/netlify-shortener) | CLI de terceros (MIT, Kent C. Dodds). Edita `_redirects` y hace commit + push. Este repo es una instancia de ese proyecto. | npm (`devDependencies`) |
| Sitio de Netlify `tmsa-short` | Hosting: publica el repo y aplica las reglas de `_redirects`. | Panel de Netlify (fuera del repo) |
| Dominio `tmsa.ar` | Apunta al sitio de Netlify por DNS. | Registro NIC Argentina + proveedor DNS (ver [Datos fuera del repo](#datos-fuera-del-repo-a-completar)) |

## Qué hace el CLI (`tmsa <link> <código>`)

Verificado leyendo el código de `netlify-shortener@2.3.0`:

1. `git pull` en el repo (trae los links que agregó otra persona).
2. Si no das código, genera uno de 5 caracteres aleatorios.
3. Agrega `https://` si el link no tiene protocolo y valida que sea una URL real.
4. Falla si el código ya existe en `_redirects`.
5. **Inserta la línea nueva al principio del archivo** y reformatea (alinea) todo el archivo.
6. `git commit -am "/<código> -> <link>"` y `git push` **a la rama en la que estés parado**.
7. Copia `https://tmsa.ar/<código>` al portapapeles.

Consecuencias que conviene tener presentes:

- Parado en `main`, el comando **publica directo a producción**, sin revisión.
- `commit -am` incluye **cualquier otro cambio** de archivos ya versionados que tengas sin commitear. Dejá el árbol limpio antes de usarlo.
- Si `git pull` o `git push` fallan (sin SSH, sin red, conflicto), el CLI sigue y muestra el error, pero la línea ya quedó escrita en tu copia local.
- En Linux, copiar al portapapeles necesita `xclip` o `xsel`.

## Cómo se resuelve un pedido en Netlify

Reglas del archivo `_redirects` (documentación de [Netlify](https://docs.netlify.com/manage/routing/redirects/overview/)):

- Se aplica **la primera regla que coincide, de arriba hacia abajo**. Por eso el CLI inserta arriba y el comodín `/*` tiene que quedar **siempre al final**.
- Una regla sin código de estado responde `301` (permanente). Se comprobó en vivo: `https://tmsa.ar/nasa` → `301` a `https://marssociety.com.ar/NASA/`.
- Si existe un **archivo real** en esa ruta, gana el archivo y no la redirección (salvo que la regla lleve `!`). Ver [riesgo de archivos expuestos](#archivos-del-repo-expuestos-públicamente).
- Una línea que empieza con `#` es un comentario.

El final del archivo hoy es:

```
/libro/*  http://argentina.marssociety.org/libro     # cualquier /libro/xxx no definido → página del libro
/marte    https://argentina.marssociety.org/producto/marte-pasado-presente-y-futuro/
/*        http://argentina.marssociety.org/:splat    # todo lo demás → mismo path en la web de Mars Society
```

El comodín `/*` con `:splat` hace que `tmsa.ar/cualquier/cosa` redirija a `argentina.marssociety.org/cualquier/cosa`. Verificado en vivo con una ruta inexistente. Por eso un código mal escrito no da 404: cae en la web principal.

## Convenciones que ya existen en `_redirects`

- Agrupado por temas con comentarios (`#SpaceApps`, etc.).
- `/libro/qrNN`: códigos QR impresos en el libro. **Son links ya impresos en papel: no se pueden cambiar de código ni borrar sin romper ejemplares físicos.** Las líneas `#/libro/qrNN #reservado para futuro contenido` marcan números reservados.
- Algunos códigos existen en dos capitalizaciones (`/nasa` y `/NASA`) para tolerar cómo lo escribe la gente.

## Archivos del repo expuestos públicamente

No hay `netlify.toml` ni carpeta de publicación, así que Netlify publica la **raíz del repo completa**. Comprobado en vivo el 28/10/2026: `https://tmsa.ar/package.json`, `/README.md`, `/LICENSE` y `/cli.js` responden `200` (en vez de redirigir). `/_redirects` responde 404 (Netlify lo consume y no lo sirve).

Impacto: bajo (es un repo público; lo expuesto ya es público en GitHub), pero tiene dos efectos prácticos:

- **No uses como código** `package.json`, `README.md`, `LICENSE`, `cli.js` ni el nombre de ninguna carpeta/archivo del repo (p. ej. `docs`): el archivo le gana a la redirección.
- Cualquier archivo que se agregue al repo queda publicado en `tmsa.ar/<archivo>`. Esta carpeta `docs/` también (`tmsa.ar/docs/ARQUITECTURA.md`).

Mitigación posible (no aplicada, ver [NETLIFY.md](NETLIFY.md#mejora-recomendada-netlifytoml)): publicar solo una carpeta que contenga únicamente `_redirects`.

## Datos fuera del repo (a completar)

Estos datos **no se pueden deducir del repo** y son lo primero que se pierde si alguien se va. Estado al 28/10/2026:

| Dato | Valor |
|---|---|
| Dueño de la cuenta de Netlify (sitio `tmsa-short`) | Pablo Mayobre (autor del repo) |
| Otras personas con acceso admin a Netlify | `[PENDIENTE]` — hoy el equipo actual **no tiene acceso** a la cuenta |
| Plan de Netlify (legacy anterior a sep/2025 o por créditos) | `[PENDIENTE]` — solo lo ve quien tenga acceso a la cuenta; ver [NETLIFY.md](NETLIFY.md#límites-del-plan-gratuito) |
| Titular del dominio `tmsa.ar` en NIC Argentina | Gabriel Caballero |
| Renovación | La hace el titular al recibir la notificación de NIC Argentina. **Próximo vencimiento: 03/11/2026** |
| Servidores DNS (delegación en NIC Argentina) | `dns1`–`dns4.p05.nsone.net` (verificado con `nslookup` el 28/10/2026) |
| Proveedor de DNS | `[PENDIENTE]` — ver nota abajo |
| Registro `A` actual del apex | `54.232.119.62` (observado el 28/10/2026; puede cambiar, no depender de este valor) |
| Rama que Netlify despliega | `main` (confirmado). Comando de build y carpeta de publicación: inferidos (sin build, raíz), `[CONFIRMAR]` |
| Quién puede hacer push a `MarsArgentina/tmsa.ar` | Gabriel Caballero, Marcos Peralta y Pablo Mayobre (confirmado el 28/10/2026) |

**Sobre el DNS:** `nsone.net` es el dominio de NS1. Netlify usa NS1 como infraestructura de su servicio *Netlify DNS*, por lo que lo más probable es que la zona DNS de `tmsa.ar` esté administrada **dentro de la cuenta de Netlify de Pablo** (no en un proveedor aparte). Es una inferencia: no se pudo confirmar sin acceso a la cuenta. Si es así, cualquier cambio de DNS (por ejemplo, migrar de hosting) requiere acceso a esa cuenta, o bien que el titular del dominio cambie la delegación de nameservers en NIC Argentina por otros nuevos.

### Riesgo de dependencia en una sola persona

Hoy el hosting, el despliegue y (probablemente) el DNS dependen de una cuenta de Netlify a la que el equipo actual no accede, mientras que el dominio lo controla otra persona. Si la cuenta se cierra, se pausa o Pablo deja de estar disponible, el equipo puede perder los links sin poder repararlos desde el panel. Lo que **sí** controla el equipo y alcanza para recuperar el servicio:

- El repo (todos los links están en `_redirects`).
- El dominio: como titular, se puede cambiar la delegación en NIC Argentina hacia otro DNS/hosting (ver [reconstruir desde cero](OPERACION.md#reconstruir-el-servicio-desde-cero)).

Acción recomendada: pedirle a Pablo que **agregue a un segundo administrador** al equipo de Netlify (o transfiera el sitio a una cuenta de la organización) y que informe el plan y el consumo de créditos.
