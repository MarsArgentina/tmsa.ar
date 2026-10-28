# Evaluación: ¿conviene hostear tmsa.ar en Netlify?

Fecha del análisis: 28/10/2026.

**Conclusión: sí, sigue siendo una buena elección, con dos ajustes.** El sitio ya corre en Netlify (se verificó con `Server: Netlify` en las respuestas de `tmsa.ar`). No hay motivo urgente para migrar; hay dos cosas que revisar: el plan/créditos del equipo y qué se publica.

## Por qué encaja

- **El diseño depende de Netlify**: `netlify-shortener` está hecho específicamente para `_redirects` de Netlify. Toda la gracia es que no hay servidor que mantener.
- **Costo**: un sitio estático sin build ni funciones entra holgado en el plan gratuito para este volumen.
- **Mantenimiento cero**: deploy automático por push, HTTPS automático, sin servidores.
- **Latencia y disponibilidad**: las redirecciones se resuelven en el borde de la CDN.
- **Portabilidad**: el archivo `_redirects` también lo entienden otros hosts (ver [alternativas](#alternativas)), así que no hay lock-in fuerte.

## Límites del plan gratuito

> Las fuentes consultadas **no coinciden** en las tarifas por unidad (varía entre 10 y 20 créditos por GB de ancho de banda, y entre 2 y 3 créditos cada 10.000 requests). Netlify actualizó tarifas el [14/04/2026](https://www.netlify.com/changelog/2026-04-14-pricing-updates-april-2026/). Verificar los números vigentes en la página oficial de precios antes de decidir algo en base a ellos.

Lo que sí es consistente entre fuentes:

- Las cuentas creadas desde septiembre de 2025 usan **créditos**: 300 por mes en el plan gratuito, sin posibilidad de sobreuso; al agotarse, **se pausan todos los sitios del equipo** hasta el siguiente ciclo.
- Cada **deploy de producción cuesta 15 créditos**.
- Las cuentas anteriores a sep/2025 (*legacy*) conservan el esquema viejo (100 GB de ancho de banda y 300 minutos de build por mes).

Qué significa para este repo:

| Aspecto | Impacto |
|---|---|
| **Cada link nuevo = un push = un deploy** | En plan por créditos serían 15 créditos por link: unos 20 links al mes consumirían todo el cupo. Es muy poco probable en la práctica, pero en campañas de muchos links (como los QR del libro) hay que agrupar cambios en un solo commit. |
| **Tráfico** | Cada visita a un link es una request de redirección, de respuesta mínima. Con el tráfico esperado de una organización chica, no debería ser relevante. `[CONFIRMAR con Analytics/uso real del panel]` |
| **Plan de la cuenta** | `[PENDIENTE]`: la cuenta es de Pablo Mayobre y el equipo actual no tiene acceso. Pedirle que revise en Netlify → Team → Billing si es legacy o por créditos y cuánto se consumió en los últimos meses. Es el dato que cierra este análisis. |
| **Acceso a la cuenta** | Es el riesgo principal hoy, más que el plan: ver [ARQUITECTURA.md](ARQUITECTURA.md#riesgo-de-dependencia-en-una-sola-persona). |
| **Riesgo de pausa** | Si el equipo de Netlify tiene **otros sitios** de la organización, comparten cupo: un sitio pesado podría pausar el acortador. Conviene un equipo dedicado o vigilar el consumo. |

## Riesgos y ajustes recomendados

1. **Se publica toda la raíz del repo** (README, `package.json`, `cli.js`, `docs/`). Es de bajo impacto, pero ensucia el espacio de nombres de los códigos. Ver [mejora recomendada](#mejora-recomendada-netlifytoml).
2. **Configuración en el panel, no en el repo**: rama, build y dominio solo existen en la interfaz de Netlify. Si alguien cambia algo ahí, el repo no lo refleja. `netlify.toml` lo versiona.
3. **Un solo punto de acceso humano** a la cuenta de Netlify y al dominio (ver checklist en [OPERACION.md](OPERACION.md#checklist-de-traspaso-cuando-alguien-se-va-del-equipo)).
4. **Push a `main` = producción**, sin revisión. Es cómodo, pero un typo en `_redirects` (o un `/*` movido) se publica al instante. Si se quiere más control: activar Deploy Previews por PR (Netlify lo hace gratis por rama) y pedir PR para cambios no triviales.
5. **Analytics**: Netlify no da estadísticas de clics por link en este esquema. Si hace falta saber cuántas veces se usó un QR o un link, hace falta otra capa (p. ej. Cloudflare delante del dominio, como sugiere el autor del CLI) o migrar a un acortador con métricas.

## Mejora recomendada: `netlify.toml`

**No está aplicada**, porque cambia lo que se despliega y no se pudo probar sin acceso al panel de Netlify. Propuesta para evaluar en una rama con Deploy Preview:

```toml
[build]
  # Publicar solo _redirects, no el resto del repo
  command = "mkdir -p public && cp _redirects public/"
  publish = "public"
```

Qué logra: deja el sitio definido como código (replicable sin depender de la memoria de nadie) y evita publicar `package.json`, `README.md`, etc. El CLI sigue funcionando porque lee y escribe `_redirects` en la raíz del repo; la carpeta `public/` es solo un artefacto del build (agregarla a `.gitignore`).

Probar antes de mergear: que el deploy preview responda `301` en `/nasa` y que `/package.json` pase a redirigir (al comodín) en lugar de servir el archivo.

## Alternativas

| Opción | Compatibilidad con este repo | Notas |
|---|---|---|
| **Cloudflare Pages** | Lee el mismo formato `_redirects` | Límite de 2.000 redirecciones estáticas + 100 dinámicas (con comodín). Hoy hay ~161 estáticas y 2 dinámicas (`/libro/*`, `/*`), o sea, entra. Cada declaración puede tener hasta 1.000 caracteres. Fuente: [docs de Cloudflare](https://developers.cloudflare.com/pages/platform/limits). Daría además analytics. Verificar el comportamiento del `:splat` en un preview antes de cambiar el DNS. |
| **Vercel / GitHub Pages** | No usan `_redirects` | Vercel necesitaría convertir a `vercel.json`; GitHub Pages no soporta redirecciones del lado del servidor. Pierde la gracia del CLI. No recomendado. |
| **Servicio de acortador pago/SaaS** (Short.io, Rebrandly, etc.) | Migración manual de ~160 links | Da métricas y panel, pero tiene costo y los códigos `/libro/qrNN` impresos dependen del proveedor para siempre. |

**Recomendación**: quedarse en Netlify. Antes de que escale el uso, revisar el plan/créditos de la cuenta (dato pendiente arriba) y probar el `netlify.toml` propuesto. Si se necesitan métricas de clics, la migración a Cloudflare Pages es de bajo esfuerzo porque el archivo de reglas es compatible.

## Fuentes

- [Netlify: Redirects overview](https://docs.netlify.com/manage/routing/redirects/overview/)
- [Netlify changelog: actualización de precios, abril 2026](https://www.netlify.com/changelog/2026-04-14-pricing-updates-april-2026/)
- [Netlify Pricing and Limits in 2026 (terceros)](https://netli.fyi/blog/netlify-pricing-and-limits)
- [Cloudflare Pages: límites](https://developers.cloudflare.com/pages/platform/limits)
- Código de [`netlify-shortener`](https://github.com/kentcdodds/netlify-shortener) v2.3.0 (leído del paquete npm)
