# Portal de Herramientas — The Group

Hub único donde los empleados ven, abren y descargan las mini-apps HTML y
skills (.md) creadas en cada sesión, clasificadas por área de negocio.

## Estructura de carpetas

```
portal/
├── index.html               ← el portal
├── assets/
│   └── logo.jpg
├── apps/
│   ├── cobros/               ← apps del área de Cobros
│   ├── legales/              ← apps de Legales / Contratos
│   ├── marketing/            ← apps de Marketing
│   ├── ventas/               ← apps de Ventas
│   └── comercial/            ← apps de Comercial
└── skills/                    ← todos los .md (skills)
```

Las pestañas del portal (Cobros, Legales / Contratos, Marketing, Ventas,
Comercial, Skills) son fijas: aparecen aunque el área todavía no tenga
herramientas, con un mensaje de "próximamente".

---

## FLUJO DE TRABAJO COMPLETO

### FASE 1 — Configuración inicial (se hace UNA sola vez, ~20 min)

**Paso 1. Crear la cuenta y el repositorio en GitHub**
1. Entra a github.com y crea una cuenta (si no tienen una de empresa).
2. Botón "New repository" → nombre: `portal-herramientas` (o similar).
3. Visibilidad: **Public** es lo más simple para Vercel gratis.
   ⚠️ Importante: público significa que cualquiera con el link puede ver los
   archivos. Como las apps son plantillas (no contienen datos de clientes),
   normalmente está bien — pero revisen que ningún HTML tenga datos reales
   embebidos antes de subirlo.
4. Marca "Add a README" y crea el repositorio.

**Paso 2. Subir el portal al repositorio**
1. Dentro del repo → botón "Add file" → "Upload files".
2. Arrastra TODO el contenido de la carpeta `portal-the-group`
   (index.html, assets/, apps/, skills/) — no la carpeta contenedora,
   sino lo que hay adentro, para que index.html quede en la raíz del repo.
3. Abajo escribe un mensaje tipo "Versión inicial del portal" → "Commit changes".

**Paso 3. Conectar Vercel**
1. Entra a vercel.com → "Sign Up" → elige "Continue with GitHub"
   (así quedan conectadas las dos cuentas automáticamente).
2. En el dashboard → "Add New… → Project".
3. Selecciona el repositorio `portal-herramientas` → "Import".
4. No cambies ninguna configuración (Framework Preset: "Other" está bien,
   no hay build) → botón "Deploy".
5. En ~30 segundos te da la URL pública, tipo:
   `https://portal-herramientas.vercel.app`
6. (Opcional) En Settings → Domains puedes ponerle un dominio propio
   más adelante.

Ese link es el que compartes con los empleados. Guárdalo en la firma de
correo, grupo de WhatsApp, etc.

---

### FASE 2 — Flujo recurrente (después de CADA sesión, ~5 min)

Este es el ciclo que repites cada vez que crean una herramienta nueva:

```
Sesión de taller          GitHub                    Vercel
─────────────────         ────────────────          ─────────────
1. Se crea el HTML   →    2. Subir archivo a    →   4. Deploy
   o skill (.md)             la carpeta del área       automático
                          3. Editar ITEMS en          (sin hacer nada)
                             index.html
```

**Paso a paso:**

1. **Sube el archivo nuevo** — En GitHub, navega a la carpeta del área
   (ej. `apps/marketing/`) → "Add file → Upload files" → arrastra el HTML
   → Commit. Para skills, la carpeta es `skills/`.

2. **Registra la herramienta en el manifiesto** — Abre `index.html` en
   GitHub → clic en el ícono de lápiz (Edit) → busca `const ITEMS = [` →
   copia un bloque existente y edítalo:

```js
{
  id: "calculadora-comisiones",
  title: "Calculadora de Comisiones",
  description: "Calcula comisiones de venta según tabla de metas.",
  category: "Ventas",                    // debe coincidir con un área
  type: "app",                            // "app" o "skill"
  file: "apps/ventas/calculadora-comisiones.html",
  dateAdded: "2026-07-10",
  tags: ["ventas", "comisiones"]
},
```

3. **Commit** — mensaje tipo "Agrega calculadora de comisiones (Ventas)".

4. **Nada más.** Vercel detecta el commit y publica la actualización solo,
   en menos de un minuto. Todos los que abran el link ven lo nuevo.

**Valores válidos de `category`:** `"Cobros"`, `"Legales / Contratos"`,
`"Marketing"`, `"Ventas"`, `"Comercial"`, `"Skills"` (tal cual, con
mayúsculas y espacios — deben coincidir con la lista AREAS del index).

---

### FASE 3 — Buenas prácticas para que se use

- Cada vez que subas algo, manda un mensaje de 2 líneas al grupo del área:
  "Nueva herramienta en el portal: Calculadora de Comisiones → [link]".
- Nombra los archivos sin espacios ni tildes: `calculadora-comisiones.html`
  mejor que `Calculadora Comisiones (v2).html`.
- Una persona del equipo cliente debería ser "dueña" del portal: revisar
  que lo nuevo funcione y avisar a su equipo.

---

## Preguntas frecuentes

**¿Vercel gratis alcanza?** Sí, de sobra: el plan Hobby es gratuito y este
portal es solo archivos estáticos (sin backend, sin base de datos). El
límite de banda ancha gratuito está lejísimos de lo que un equipo interno
consume.

**¿Y si me equivoco editando index.html?** GitHub guarda el historial:
en el repo → "History" puedes ver cada versión anterior y revertir.

**¿Puedo probarlo antes de publicar?** Sí: descarga el repo, abre
index.html localmente y revisa. O simplemente publica — revertir un commit
toma segundos.

**¿Se puede restringir el acceso?** Con Vercel gratis el link es público
(aunque nadie lo encuentra sin conocerlo). Si más adelante necesitan
protección con contraseña, las opciones son el plan de pago de Vercel,
o mover el hosting a algo interno. Mientras las apps sean plantillas sin
datos reales, el riesgo es bajo.
