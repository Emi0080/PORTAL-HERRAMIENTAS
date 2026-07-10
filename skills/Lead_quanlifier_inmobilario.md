---
name: lead-qualifier-inmobiliario
description: >
  Califica leads inmobiliarios para The Group Panama: muestra un dashboard interactivo
  con sliders y selecciones para que el agente cargue los datos del prospecto, analiza
  su capacidad de compra real, emite un veredicto claro (CALIFICA / NO CALIFICA /
  CALIFICA CON CONDICIONES) y genera un PDF de presentación para compartirle al cliente
  con proyectos recomendados y próximos pasos.

  Usá este skill cuando un agente diga "tengo un lead", "me llegó un interesado",
  "quiero calificar a este cliente", "¿puede pagar esta propiedad?", o cuando ingrese
  datos de un prospecto y quiera saber si tiene capacidad de compra. También triggeá
  cuando mencionen ingreso del cliente, presupuesto del comprador, perfil del lead
  inmobiliario, o cuando pidan un resumen para enviarle al cliente. Usá siempre este
  skill ante cualquier consulta de calificación de lead, incluso si el agente solo
  menciona un número de ingreso o un tipo de propiedad.
---

# Lead Qualifier — The Group Panama

## Objetivo
Calificar la capacidad real de compra de un lead y generar dos outputs:
1. **Análisis para el agente** — veredicto accionable con match de proyectos
2. **PDF para el cliente** — resumen visual para compartir por WhatsApp/email

---

## PASO 1 — Mostrar dashboard interactivo

Siempre que se active este skill, mostrar el dashboard con `show_widget`. El agente completa sliders y selecciones. El campo de nombre es obligatorio para el PDF.

```html
<form class="elicit">
  <div class="elicit-header">
    <svg viewBox="0 0 20 20" fill="currentColor"><path d="M11.586 2a1.5 1.5 0 0 1 1.06.44l2.914 2.914a1.5 1.5 0 0 1 .44 1.06V16.5a1.5 1.5 0 0 1-1.5 1.5h-9a1.5 1.5 0 0 1-1.492-1.347L4 16.5v-13A1.5 1.5 0 0 1 5.5 2zM5.5 3a.5.5 0 0 0-.5.5v13a.5.5 0 0 0 .5.5h9a.5.5 0 0 0 .5-.5V7h-2.5A1.5 1.5 0 0 1 11 5.5V3zm7.04 10.304a.5.5 0 0 1 .92.392c-.295.69-.871 1.304-1.66 1.304-.487 0-.892-.234-1.2-.574-.309.34-.713.574-1.2.574-.486 0-.892-.233-1.2-.574-.31.34-.714.574-1.2.574a.5.5 0 0 1 0-1c.212 0 .52-.18.74-.696l.034-.067a.5.5 0 0 1 .886.067c.221.516.528.696.74.696.213 0 .52-.18.74-.696l.035-.067a.5.5 0 0 1 .885.067c.22.516.527.696.74.696s.519-.18.74-.696m0-4a.5.5 0 0 1 .92.392c-.295.69-.871 1.304-1.66 1.304-.487 0-.892-.234-1.2-.574-.309.34-.713.574-1.2.574-.486 0-.892-.233-1.2-.574-.31.34-.714.574-1.2.574a.5.5 0 0 1 0-1c.212 0 .52-.18.74-.696l.034-.067a.5.5 0 0 1 .886.067c.221.516.528.696.74.696.213 0 .52-.18.74-.696l.035-.067a.5.5 0 0 1 .885.067c.22.516.527.696.74.696s.519-.18.74-.696M12 5.5a.5.5 0 0 0 .5.5h2.293L12 3.207z"/></svg>
    <span>Datos del lead</span>
  </div>
  <div class="elicit-body">

    <div class="elicit-group">
      <label class="elicit-question">Nombre del cliente</label>
      <input type="text" class="elicit-textarea" data-name="nombre" placeholder="Ej: Juan Pérez" style="height:42px; resize:none; padding:10px 14px;">
    </div>

    <div class="elicit-group">
      <label class="elicit-question">¿Cuál es el ingreso mensual neto del cliente?</label>
      <div style="display:flex; align-items:center; gap:12px;">
        <span style="font-size:13px; color:var(--color-text-secondary);">$500</span>
        <input type="range" class="elicit-range" data-name="ingreso" min="500" max="10000" step="100" value="2000" style="flex:1;" oninput="document.getElementById('ing-out').textContent='$'+Number(this.value).toLocaleString()">
        <span style="font-size:13px; color:var(--color-text-secondary);">$10,000</span>
      </div>
      <div style="text-align:center; font-size:18px; font-weight:500; color:var(--color-text-primary); margin-top:4px;" id="ing-out">$2,000</div>
    </div>

    <div class="elicit-group">
      <label class="elicit-question">¿Cuánto paga mensualmente en deudas?</label>
      <div style="display:flex; align-items:center; gap:12px;">
        <span style="font-size:13px; color:var(--color-text-secondary);">$0</span>
        <input type="range" class="elicit-range" data-name="deudas" min="0" max="3000" step="50" value="0" style="flex:1;" oninput="document.getElementById('deu-out').textContent='$'+Number(this.value).toLocaleString()">
        <span style="font-size:13px; color:var(--color-text-secondary);">$3,000</span>
      </div>
      <div style="text-align:center; font-size:18px; font-weight:500; color:var(--color-text-primary); margin-top:4px;" id="deu-out">$0</div>
    </div>

    <div class="elicit-group">
      <label class="elicit-question">¿Cuánto tiene disponible de cuota inicial?</label>
      <div style="display:flex; align-items:center; gap:12px;">
        <span style="font-size:13px; color:var(--color-text-secondary);">$0</span>
        <input type="range" class="elicit-range" data-name="inicial" min="0" max="150000" step="1000" value="20000" style="flex:1;" oninput="document.getElementById('ini-out').textContent='$'+Number(this.value).toLocaleString()">
        <span style="font-size:13px; color:var(--color-text-secondary);">$150,000</span>
      </div>
      <div style="text-align:center; font-size:18px; font-weight:500; color:var(--color-text-primary); margin-top:4px;" id="ini-out">$20,000</div>
    </div>

    <div class="elicit-group">
      <label class="elicit-question">¿Qué tipo de propiedad busca?</label>
      <div class="elicit-pills" data-name="tipo_propiedad" data-multi="false">
        <button type="button" class="elicit-pill" data-value="Apartamento" style="border-radius:12px; padding:14px 16px; display:flex; gap:10px; align-items:flex-start; text-align:left; min-width:120px; box-shadow:0 1px 2px rgba(0,0,0,0.04)">
          <svg width="18" height="18" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5"><rect x="3" y="3" width="14" height="14" rx="2"/><path d="M3 8h14M8 8v9M12 8v9"/></svg>
          <span style="font-size:13px; font-weight:500">Apartamento</span>
        </button>
        <button type="button" class="elicit-pill" data-value="Casa" style="border-radius:12px; padding:14px 16px; display:flex; gap:10px; align-items:flex-start; text-align:left; min-width:120px; box-shadow:0 1px 2px rgba(0,0,0,0.04)">
          <svg width="18" height="18" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M3 10L10 3l7 7M5 8.5V17h10V8.5"/><rect x="8" y="13" width="4" height="4"/></svg>
          <span style="font-size:13px; font-weight:500">Casa</span>
        </button>
        <button type="button" class="elicit-pill" data-value="Local comercial" style="border-radius:12px; padding:14px 16px; display:flex; gap:10px; align-items:flex-start; text-align:left; min-width:120px; box-shadow:0 1px 2px rgba(0,0,0,0.04)">
          <svg width="18" height="18" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5"><rect x="2" y="8" width="16" height="9" rx="1"/><path d="M2 8l2-5h12l2 5"/><path d="M8 17v-4h4v4"/></svg>
          <span style="font-size:13px; font-weight:500">Local comercial</span>
        </button>
        <button type="button" class="elicit-pill" data-value="Oficina" style="border-radius:12px; padding:14px 16px; display:flex; gap:10px; align-items:flex-start; text-align:left; min-width:120px; box-shadow:0 1px 2px rgba(0,0,0,0.04)">
          <svg width="18" height="18" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5"><rect x="3" y="4" width="14" height="12" rx="2"/><path d="M7 4V2m6 2V2M3 9h14"/></svg>
          <span style="font-size:13px; font-weight:500">Oficina</span>
        </button>
      </div>
    </div>

    <div class="elicit-group">
      <label class="elicit-question">¿Cuántas recámaras necesita?</label>
      <div class="elicit-pills" data-name="recamaras" data-multi="false">
        <button type="button" class="elicit-pill" data-value="1">1 rec.</button>
        <button type="button" class="elicit-pill" data-value="2">2 rec.</button>
        <button type="button" class="elicit-pill" data-value="3">3 rec.</button>
        <button type="button" class="elicit-pill" data-value="4+">4+</button>
      </div>
    </div>

    <div class="elicit-group">
      <label class="elicit-question">¿Es para vivir o para invertir?</label>
      <div class="elicit-pills" data-name="proposito" data-multi="false">
        <button type="button" class="elicit-pill" data-value="Vivir">Para vivir</button>
        <button type="button" class="elicit-pill" data-value="Invertir">Para invertir</button>
        <button type="button" class="elicit-pill" data-value="Ambos">Ambos</button>
      </div>
    </div>

    <div class="elicit-group">
      <label class="elicit-question">¿Cuál es la urgencia del cliente?</label>
      <div class="elicit-pills" data-name="urgencia" data-multi="false">
        <button type="button" class="elicit-pill" data-value="Inmediata" data-accent="warning">Inmediata</button>
        <button type="button" class="elicit-pill" data-value="3-6 meses">3-6 meses</button>
        <button type="button" class="elicit-pill" data-value="Explorando">Explorando</button>
      </div>
    </div>

    <div class="elicit-group">
      <label class="elicit-question">¿Es panameño o extranjero?</label>
      <div class="elicit-pills" data-name="nacionalidad" data-multi="false">
        <button type="button" class="elicit-pill" data-value="Panameño">Panameño</button>
        <button type="button" class="elicit-pill" data-value="Extranjero">Extranjero</button>
      </div>
    </div>

    <div class="elicit-group">
      <label class="elicit-question">¿Tipo de empleo?</label>
      <div class="elicit-pills" data-name="empleo" data-multi="false">
        <button type="button" class="elicit-pill" data-value="Formal">Empleo formal</button>
        <button type="button" class="elicit-pill" data-value="Independiente">Independiente / freelance</button>
        <button type="button" class="elicit-pill" data-value="Empresario">Empresario</button>
      </div>
    </div>

    <div class="elicit-group">
      <label class="elicit-question">¿Tiene preaprobación bancaria?</label>
      <div class="elicit-pills" data-name="preaprobacion" data-multi="false">
        <button type="button" class="elicit-pill" data-value="Sí">Sí</button>
        <button type="button" class="elicit-pill" data-value="No">No</button>
        <button type="button" class="elicit-pill" data-value="En proceso">En proceso</button>
      </div>
    </div>

    <div class="elicit-group">
      <label class="elicit-question">Zona o proyecto de interés (opcional)</label>
      <textarea class="elicit-textarea" data-name="zona" placeholder="Ej: Bella Vista, Coronado, sin preferencia..."></textarea>
    </div>

  </div>
  <div class="elicit-footer">
    <button type="button" class="elicit-skip">Omitir</button>
    <button type="button" class="elicit-submit">Analizar lead ↗</button>
  </div>
</form>
```

---

## PASO 2 — Análisis de capacidad de compra

Al recibir los datos del formulario, calcular y emitir el análisis.

### Reglas del mercado panameño

**Cuota máxima:** 30-35% del ingreso neto disponible (ingreso - deudas).

**Estimación de precio máximo financiable** (25 años, ~7%):

| Cuota mensual | Precio estimado |
|---|---|
| $500 | ~$70,000 |
| $800 | ~$112,000 |
| $1,000 | ~$140,000 |
| $1,500 | ~$210,000 |
| $2,000 | ~$280,000 |
| $2,500 | ~$350,000 |
| $3,000 | ~$420,000 |

**Cuota inicial mínima:**
- Banco convencional panameño: 10-20% del precio
- Extranjeros: 20-30% + documentación adicional
- Ingresos independientes: mayor riesgo, recomendar preaprobación urgente

### Formato de análisis para el agente

```
📊 ANÁLISIS DE CAPACIDAD — [Nombre]

💰 PERFIL FINANCIERO
- Ingreso neto mensual: $X
- Deudas mensuales: $X
- Ingreso disponible para hipoteca: $X
- Cuota máxima recomendada: $X - $X (30-35%)
- Cuota inicial disponible: $X

🏠 PREFERENCIAS
- Tipo: [tipo]
- Recámaras: X
- Zona: [zona o "abierto a opciones"]
- Propósito: [vivir/invertir]
- Urgencia: [inmediata/3-6 meses/explorando]

🎯 RANGO DE PROPIEDAD ALCANZABLE: $X - $X

✅/❌/⚠️ VEREDICTO: [CALIFICA / NO CALIFICA / CALIFICA CON CONDICIONES]

📌 RECOMENDACIÓN PARA EL AGENTE:
[2-3 líneas accionables: qué hacer con este lead, qué propiedad mostrarle,
si conviene derivarlo a banco primero, si hay gap entre deseo y capacidad.]
```

---

## PASO 3 — Generar PDF para el cliente

Inmediatamente después del análisis, generar el PDF con reportlab. Instalar si no está: `pip install reportlab --break-system-packages -q`

### Estructura del PDF

- **Header:** The Group Panama en fondo #6B2D8B, nombre del cliente a la derecha
- **Banner de veredicto:** color según resultado (amber=condiciones, verde=califica, rojo=no califica)
- **4 métricas clave:** ingreso, disponible para hipoteca, cuota inicial, rango alcanzable
- **Proyectos recomendados:** 1-2 cards con precio, cuota quincenal, inicial necesaria y nota
- **3 pasos concretos** numerados como próximos pasos
- **Footer:** The Group Panama · fecha del análisis

### Colores corporativos

```python
PURPLE = '#6B2D8B'       # Header, steps, acentos
PURPLE_LIGHT = '#EDE3F4' # Bordes de cards
PURPLE_MID = '#9B5DC0'   # Header proyecto secundario
AMBER = '#BA7517'        # Veredicto con condiciones
AMBER_LIGHT = '#FAEEDA'
GREEN = '#3B6D11'        # Veredicto califica
GREEN_LIGHT = '#EAF3DE'
RED = '#A32D2D'          # Veredicto no califica
RED_LIGHT = '#FCEBEB'
GRAY_DARK = '#2C2C2A'
GRAY_MID = '#888780'
GRAY_LIGHT = '#F1EFE8'
```

### Lógica de color del veredicto banner

- CALIFICA → fondo `GREEN_LIGHT`, borde `GREEN`, texto `GREEN`
- NO CALIFICA → fondo `RED_LIGHT`, borde `RED`, texto `RED`
- CALIFICA CON CONDICIONES → fondo `AMBER_LIGHT`, borde `AMBER`, texto `AMBER`

### Output

Guardar en `/home/claude/resumen_[nombre_lowercase].pdf`, copiar a `/mnt/user-data/outputs/` y presentar con `present_files`.

---

## Portafolio de proyectos — The Group Panama

### APARTAMENTOS — CIUDAD DE PANAMÁ

| Proyecto | Zona | Recámaras | Precio desde | Notas |
|---|---|---|---|---|
| **Mystic Village** | Don Bosco | 2 rec / 2 baños | $99,000 | Letra quincenal ~$286, ingreso mínimo ~$1,700. A pasos del metro. |
| **Signature Point** (gama completa) | La Cresta | 1-2-3 rec | $135,000–$351,601 | Desde 49.3m² hasta 152.87m² con terraza. Cuota semilla $1,500. |
| **43 GV Green View** | Bella Vista | 2 rec / 2 baños | $231,384–$262,944 | 96-110m². Penthouse desde $450,000. A pasos de Cinta Costera. |
| **Lumière** | Costa del Este | 3 rec / 2.5 baños | $500,000 | Desde 210m². Segmento premium. |
| **Dynasty Residences** | Bella Vista | 3 rec / 2 baños | $540,000 | Desde 285m², 2 parking. Lujo. |

### FUERA DE CIUDAD / OTROS TIPOS

| Proyecto | Zona | Tipo | Precio desde | Notas |
|---|---|---|---|---|
| **Mystic Valley** | El Crisol | Apto 2-3 rec | $120,000 | Letra quincenal ~$286, ingreso ~$1,700. Balcón. |
| **Mystic City Modelo A** | La Chorrera | Casa 3 rec / 2 baños | $99,000 | Preventa. Letra ~$170, ingreso ~$1,200. |
| **Mystic City Modelo C** | La Chorrera | Casa 2 plantas, 3 rec | $130,000 | 185m² construcción. Letra ~$316, ingreso ~$2,100. |
| **Mystic City Plaza** | La Chorrera | Local comercial | $90,000 | Desde 50m², alquiler desde $450/mes. |
| **Mystic Garden** | Alto Boquete | Casa 3 rec | $125,000 | 105m², lote 400m². Vista volcán Barú. Ideal retiro/inversión. |
| **Paradise Point** | Coronado | Casa 3 rec | $227,500 | Lote 210m², construcción 180m². |

### OFICINAS
| Proyecto | Zona | Precio desde |
|---|---|---|
| **Obarrio** | Obarrio | $187,392 (alquiler desde $850/mes) |

### Match rápido por rango

| Rango alcanzable | Proyectos recomendados |
|---|---|
| Hasta $100,000 | Mystic Village, Mystic City Modelo A |
| $100K–$135K | Mystic Valley, Mystic City Modelo C, Mystic Garden |
| $135K–$200K | Signature Point (1-2 rec), Mystic City Modelo C |
| $200K–$270K | Signature Point (2-3 rec), 43 GV, Paradise Point |
| $270K–$450K | Signature Point terraza, 43 GV Penthouse |
| $450K+ | Lumière, Dynasty Residences |

---

## Casos especiales

- **Extranjero:** Exigir 20-30% de inicial + documentación adicional. Mencionarlo en el análisis y en el PDF.
- **Ingresos independientes:** Mayor riesgo bancario. Recomendar preaprobación como primer paso no negociable.
- **Sin preaprobación + urgencia inmediata:** Indicar arrancar proceso bancario esta semana.
- **Gap entre deseo y capacidad:** Decirlo con claridad y sugerir alternativas realistas del portafolio.

---

## Tono

- **Análisis para el agente:** directo, sin rodeos, orientado a acción.
- **PDF para el cliente:** profesional, cálido, enfocado en posibilidades y próximos pasos.
