# Operación: agregar, cambiar y reconstruir

Guía de uso diario y de recuperación. Para entender el sistema, ver [ARQUITECTURA.md](ARQUITECTURA.md).

## Agregar un link

### Con el CLI (recomendado)

Preparación única en [README.md](../README.md#instalación). Después:

```sh
tmsa https://ejemplo.com/pagina-larga mi-codigo
```

Antes de correrlo: estar en `main` (o en la rama que quieras publicar), con el árbol limpio (`git status`) y con SSH configurado. El comando publica **sin pedir confirmación** (ver [qué hace el CLI](ARQUITECTURA.md#qué-hace-el-cli-tmsa-link-código)).

### A mano (sin Node.js)

1. Editá `_redirects` desde GitHub (botón de lápiz) o localmente.
2. Agregá una línea **arriba de `# Fallback` / `/libro/*` / `/*`**, nunca debajo del `/*` (esa regla captura todo y las de abajo no se ejecutan):
   ```
   /mi-codigo   https://ejemplo.com/pagina-larga
   ```
3. Commit a `main`. Netlify despliega solo.

### Reglas para elegir el código

- Minúsculas, sin espacios ni tildes; guiones o `/` para agrupar (`/solar54/charla1`).
- Que no exista ya: `grep -i "^/mi-codigo " _redirects`.
- Que no coincida con un archivo/carpeta del repo (`package.json`, `README.md`, `LICENSE`, `cli.js`, `docs`...): el archivo ganaría.
- No reutilizar ni tocar `/libro/qrNN`: están impresos en el libro.
- Si dudás entre dos escrituras comunes, agregá las dos (como `/nasa` y `/NASA`).

## Cambiar o borrar un link

Editá o borrá la línea en `_redirects` y commiteá a `main`.

Dos advertencias:

- Las redirecciones son `301` (permanentes) y **los navegadores las cachean**: quien ya visitó el link puede seguir yendo al destino viejo un buen tiempo. Si un link va a cambiar seguido (formularios, eventos), usá `302`:
  ```
  /inscripcion   https://forms.gle/xxxx   302
  ```
  Ojo: el CLI no entiende esa tercera columna (por lectura de su código, tomaría `302` como destino al reformatear y desalinea el archivo, aunque no rompe la redirección). Para líneas con `302`, editá a mano.
- Borrar un link publicado lo deja caer en el comodín `/*` (web principal), no en un 404. Es lo esperado.

## Verificar que un link funciona

```sh
curl -sI https://tmsa.ar/mi-codigo | grep -iE "^(HTTP|location)"
```

Debe mostrar `301` y el `Location:` esperado. Netlify tarda unos segundos a un par de minutos en desplegar; el estado se ve en el badge del README o en el panel de Netlify (Deploys).

## Problemas frecuentes

| Síntoma | Causa probable | Qué hacer |
|---|---|---|
| El link nuevo redirige a la web principal | Línea debajo de `/*`, o deploy todavía en curso, o error de tipeo | Mover la línea arriba; esperar; revisar el deploy en Netlify |
| El link nuevo sirve un archivo en vez de redirigir | El código coincide con un archivo del repo | Elegir otro código |
| `A link with this code already exists` | Código repetido | Elegir otro, o editar la línea existente |
| `git push` falla con `Permission denied (publickey)` | SSH no configurado | [Configurar llaves SSH](https://docs.github.com/es/authentication/connecting-to-github-with-ssh) |
| `tmsa: command not found` | No se hizo `npm link` | Usar `npm run shorten <link> <código>` desde la carpeta del repo |
| El CLI se cuelga/falla al copiar al portapapeles (Linux) | Falta `xclip`/`xsel` | Instalarlo, o editar `_redirects` a mano |
| El deploy falla en Netlify | `_redirects` con sintaxis rota | Revertir el último commit (`git revert`) y volver a pushear |
| Un link cambió pero hay gente que sigue yendo al destino viejo | Caché del 301 en navegadores | Esperar, o ver nota de `302` arriba |

## Reconstruir el servicio desde cero

Plan de recuperación si se pierde el sitio de Netlify o hay que migrar de hosting. Tiempo estimado: menos de una hora, más la propagación de DNS.

1. **Repo**: este repo contiene todo (links + CLI). Verificar que exista copia en GitHub (`MarsArgentina/tmsa.ar`) o clonar una local.
2. **Sitio en Netlify**: *Add new site → Import an existing project* → GitHub → `MarsArgentina/tmsa.ar`. Rama `main`, build command vacío, publish directory vacío (raíz). Nombre sugerido: `tmsa-short`.
3. **Dominio**: *Domain management → Add a domain* → `tmsa.ar`. Netlify indica los registros DNS (apex: registro `A`/`ALIAS` hacia Netlify; `www`: `CNAME`). Cargarlos en el proveedor DNS del dominio. Hoy el dominio está delegado a `dns1`–`dns4.p05.nsone.net` (probablemente Netlify DNS de la cuenta anterior): si no se tiene acceso a esa zona, el titular debe **cambiar los servidores DNS en NIC Argentina** (Mi cuenta → dominio `tmsa.ar` → delegación) por los que indique el nuevo sitio de Netlify (o el DNS elegido) y recrear allí cualquier otro registro que existiera (por ejemplo, mail, si se usa). Antes de cambiar la delegación, listar los registros actuales con `nslookup -type=any tmsa.ar` o `dig tmsa.ar ANY` para no perder ninguno.
4. **HTTPS**: Netlify emite el certificado Let's Encrypt automáticamente cuando el DNS resuelve; revisar *Domain management → HTTPS*.
5. **Probar** con `curl -sI https://tmsa.ar/nasa` (debe dar `301` a `marssociety.com.ar/NASA/`) y con una ruta inexistente (debe redirigir a `argentina.marssociety.org/...`).
6. **Badge del README**: el badge de estado apunta al ID del sitio viejo. Regenerarlo en *Site configuration → Status badges* y reemplazarlo en `README.md`.

Para migrar a otro proveedor (Cloudflare Pages, etc.) ver [NETLIFY.md](NETLIFY.md#alternativas).

## Checklist de traspaso (cuando alguien se va del equipo)

- [ ] Hay al menos **dos personas** con acceso admin a Netlify.
- [ ] Hay al menos dos personas con permiso de push al repo y la cuenta de Netlify no depende del mail de una sola persona.
- [ ] El contacto de NIC Argentina del dominio `tmsa.ar` es un mail de la organización, no personal. Hoy el titular es Gabriel Caballero y renueva al recibir la notificación; **próximo vencimiento 03/11/2026**. Si el aviso llega a un mail personal, avisar a un segundo responsable para que no se pierda la renovación.
- [ ] Completada la tabla de [datos fuera del repo](ARQUITECTURA.md#datos-fuera-del-repo-a-completar).
