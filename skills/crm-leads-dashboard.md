---
name: crm-leads-dashboard
description: >
  Analiza un archivo .xlsx exportado del CRM (Zoho u otro) con leads inmobiliarios y genera
  automáticamente DOS outputs: (1) dashboard visual interactivo en el chat con 4 tabs
  (seguimiento activo, análisis de descartados, calidad de fuentes, conclusiones), y
  (2) un Excel descargable SOLO con los leads a contactar, ordenado por prioridad, con
  columnas de gestión para el vendedor y análisis de descartados.

  Triggeá SIEMPRE que el usuario suba un .xlsx de leads, mencione "leads del CRM",
  "seguimiento de leads", "dashboard de leads", "leads sin movimiento", "prioridad de llamadas",
  "leads duplicados", "exporté mis leads", "bajé el CRM", "quiero ver mis leads",
  o pida analizar un archivo de prospectos inmobiliarios. También triggeá con exports
  completos del CRM que tengan columnas como "Lead Descartado", "Razón de Descartado",
  "Fuente de Lead Generado", "Descripción", "Comentarios".
---

# CRM Leads Dashboard — Inmobiliario

Convierte cualquier export del CRM en un dashboard accionable + Excel de trabajo para el vendedor.
Siempre genera DOS outputs en paralelo: dashboard visual en el chat y Excel descargable.

---

## Paso 0 — Detectar tipo de archivo

Leer columnas con pandas y clasificar:

### Tipo A — Export simple (≤10 columnas)
Columnas típicas: `ID de registro`, `Lead Generado Name`, `Proyecto`, `Teléfono`, `Estado de Lead Generado`, `Móvil`
→ Solo tab de seguimiento activo. No hay datos de descartados ni fuentes.

### Tipo B — Export completo (30+ columnas)
Incluye: `Lead Descartado`, `Razón de Descartado`, `Explicación de Descartado`, `Fuente de Lead Generado`, `Campaña`, `Descripción`, `Comentarios`, `Hora de creación`, `Nombre`, `Apellidos`
→ Dashboard completo con los 4 tabs + Excel enriquecido.

---

## Paso 1 — Limpieza y normalización

```python
import pandas as pd

df = pd.read_excel('archivo.xlsx')
df.columns = df.columns.str.strip()

# Teléfono consolidado
for col in ['Teléfono/Móvil', 'Móvil', 'Teléfono']:
    if col in df.columns:
        df['Contacto'] = df.get('Teléfono', pd.Series()).fillna(df.get('Móvil', pd.Series()))
        break

# Nombre limpio — el CRM a veces duplica "Juan Perez Juan Perez"
def limpiar_nombre(nombre):
    if pd.isna(nombre): return ''
    nombre = str(nombre).strip()
    palabras = nombre.split()
    mitad = len(palabras) // 2
    if mitad > 0 and palabras[:mitad] == palabras[mitad:]:
        return ' '.join(palabras[:mitad])
    return nombre

# Tipo B: combinar Nombre + Apellidos
if 'Nombre' in df.columns and 'Apellidos' in df.columns:
    df['Nombre_limpio'] = (df['Nombre'].fillna('') + ' ' + df['Apellidos'].fillna('')).str.strip()
    df['Nombre_limpio'] = df['Nombre_limpio'].apply(limpiar_nombre)
elif 'Lead Generado Name' in df.columns:
    df['Nombre_limpio'] = df['Lead Generado Name'].apply(limpiar_nombre)
```

---

## Paso 2 — Separar activos de descartados

```python
# Tipo B
if 'Lead Descartado' in df.columns:
    activos = df[df['Lead Descartado'] != True].copy()
    descartados = df[df['Lead Descartado'] == True].copy()
else:
    # Tipo A — todos son activos
    activos = df.copy()
    descartados = pd.DataFrame()
```

---

## Paso 3 — Calcular prioridad de activos

```python
prioridad_map = {
    'Nuevo': (1, 'URGENTE', 'Primer contacto hoy'),
    'Primer Contacto': (1, 'URGENTE', 'Llamar hoy'),
    'Cita agendada': (1, 'URGENTE', 'Confirmar cita'),
    'En contacto': (2, 'HOY', 'Dar seguimiento'),
    'Seguimiento nro 2': (2, 'HOY', 'Llamar — segundo intento'),
    'Seguimiento luego del 1er contacto': (2, 'HOY', 'Dar seguimiento'),
    'Seguimiento nro 3': (3, 'ESTA SEMANA', 'Tercer intento'),
    'Contactado sin respuesta': (3, 'ESTA SEMANA', 'Intentar canal alternativo'),
    'Cita concretada': (4, 'SEGUIMIENTO', 'Enviar materiales'),
    'Seguimiento a largo plazo': (5, 'BAJO', 'Retomar en 2+ semanas'),
}

col_estado = 'Estado de Lead Generado'
activos['Orden'] = activos[col_estado].map(lambda x: prioridad_map.get(x, (5,'BAJO','-'))[0])
activos['Prioridad'] = activos[col_estado].map(lambda x: prioridad_map.get(x, (5,'BAJO','-'))[1])
activos['Accion'] = activos[col_estado].map(lambda x: prioridad_map.get(x, (5,'BAJO','-'))[2])
activos = activos.sort_values(['Orden','Nombre_limpio']).reset_index(drop=True)
```

---

## Paso 4 — Detectar duplicados en activos

```python
tel_limpio = activos['Contacto'].astype(str).str.replace(r'\D','',regex=True)
dup_tel = tel_limpio.duplicated(keep=False) & (tel_limpio != '') & (tel_limpio != 'nan')
nombre_corto = activos['Nombre_limpio'].str.lower().str.split().str[:2].str.join(' ')
dup_nombre = nombre_corto.duplicated(keep=False) & (nombre_corto != '')
activos['Es_Dup'] = dup_tel | dup_nombre
```

---

## Paso 5 — Métricas de descartados (solo Tipo B)

```python
if len(descartados) > 0:
    razones = descartados['Razón de Descartado'].value_counts(dropna=False)
    
    por_fuente = df.groupby('Fuente de Lead Generado').agg(
        total=('Lead Descartado','count'),
        desc=('Lead Descartado','sum')
    ).assign(tasa=lambda x: (x['desc']/x['total']*100).round(1))
    por_fuente = por_fuente[por_fuente['total'] >= 20].sort_values('tasa', ascending=False)

    por_proyecto = df.groupby('Proyecto').agg(
        total=('Lead Descartado','count'),
        desc=('Lead Descartado','sum')
    ).assign(tasa=lambda x: (x['desc']/x['total']*100).round(1))
    por_proyecto = por_proyecto[por_proyecto['total'] >= 20].sort_values('tasa', ascending=False)

    competencia = descartados[descartados['Razón de Descartado'] == 'Compro en otro proyecto']
    
    # Año de creación
    df['año'] = pd.to_datetime(df['Hora de creación'], errors='coerce').dt.year
    por_año = df.groupby('año').size()
```

---

## Paso 6 — Dashboard visual (4 tabs)

Llamar `read_me` con módulo `mockup` antes de construir el widget.
Usar `show_widget` con HTML. El dashboard tiene pestañas con JS simple.

### Tab 1: Seguimiento activo
- 4 cards: urgentes hoy, seguimiento día, esta semana, duplicados
- Barras horizontales por estado con colores semáforo (rojo/naranja/verde/gris)
- Tabla por proyecto con cantidad y %
- Top 5 prioridad: nombre, proyecto, teléfono como `<a href="tel:...">`, badge de acción
- Si no hay temperaturas: aviso pequeño al pie

### Tab 2: Descartados (Tipo B) / mensaje vacío (Tipo A)
- Alerta roja con tasa global de descarte
- 3 cards: total CRM, descartados, activos
- Barras por motivo de descarte (ordenadas de mayor a menor)
- Tabla por proyecto con tasa %
- Tabla "Se fueron a la competencia" con proyecto + cantidad + razón extraída de comentarios
- Evolución anual de leads como barras horizontales

### Tab 3: Calidad de fuentes (Tipo B)
- Barra por fuente: ancho = % descartados, muestra % activos al lado
- Destacar con ★ la fuente con mejor conversión relativa
- Tabla de top motivo de descarte por fuente (3 columnas)
- Nota al pie normalizando "ig/fb" como probable Facebook sin etiquetar

### Tab 4: Conclusiones
- Máximo 3 problemas críticos (border izquierdo rojo)
- Máximo 4 oportunidades (border naranja o verde)
- Derivadas de los datos reales — no genéricas

---

## Paso 7 — Excel de contactos a llamar

**Solo incluir activos** — NO incluir descartados del CRM (Tipo B).
Excluir Prioridad = BAJO salvo que el total activo sea < 20.

### Hoja 1: "A Contactar"

Columnas (en orden):

| # | Nombre columna | Tipo | Quién llena |
|---|---------------|------|-------------|
| A | Nombre | dato CRM | — |
| B | Proyecto | dato CRM | — |
| C | Teléfono | dato CRM | — |
| D | Estado CRM | dato CRM | — |
| E | Prioridad | calculado | — |
| F | Acción sugerida | calculado | — |
| G | Duplicado | calculado | — |
| H | Fecha último contacto | vacío | vendedor |
| I | Canal usado | dropdown | vendedor |
| J | Temperatura | dropdown | vendedor |
| K | Señal de interés | dropdown | vendedor |
| L | Próxima acción | texto | vendedor |
| M | Fecha próxima acción | fecha | vendedor |
| N | Descartado | dropdown | vendedor |
| O | Motivo de descarte | dropdown | vendedor |

Formato filas:
- URGENTE: `#FFE8E8`
- HOY: `#FFF3E0`
- ESTA SEMANA: `#FFFDE7`
- SEGUIMIENTO: `#F5F5F5`
- Duplicado: `#FCE4EC`
- Columnas H–O: `#E3F2FD` (azul muy claro — vendedor las llena)
- Header: bg `#1A252F`, texto blanco, bold
- Freeze pane en A2, autofilter en row 1

### Hoja 2: "Dashboard"

Sin líneas de cuadrícula. Secciones:
1. Resumen ejecutivo con fórmulas COUNTIF
2. Por estado (COUNTIF sobre col D)
3. Por proyecto (COUNTIF sobre col B)
4. Temperaturas (COUNTIF sobre col J)
5. Top 5 calientes — fórmulas array INDEX/SMALL filtrando col J = "🔥 Muy caliente"
6. Contador "contactados hoy" — COUNTIF sobre col H = TEXT(TODAY())

### Hoja 3: "Duplicados"
Lista de duplicados detectados: nombre, proyecto, teléfono, estado, prioridad.

### Hoja 4: "Descartados" (siempre incluir)
Estructura para que el vendedor registre sus propios descartados.
Si es Tipo B: agregar también resumen de razones del CRM como referencia (valores estáticos).

Fórmulas en Descartados:
- Total descartados por vendedor: `COUNTIF(col N, "Sí")`
- Por motivo: `COUNTIFS(col N, "Sí", col O, motivo)`
- % sobre total activos: fórmula dinámica

---

## Paso 8 — Recalcular y entregar

```bash
python scripts/recalc.py /home/claude/output.xlsx 30
```

Verificar que `status = "success"` y `total_errors = 0`.
Copiar a `/mnt/user-data/outputs/` y llamar `present_files`.

Nombre del archivo: `Contactos_[Proyecto o "CRM"]_[fecha del archivo o hoy].xlsx`

---

## Mensaje final al usuario

3 líneas máximo:
1. Cuántos leads activos hay y cuántos son urgentes hoy
2. El hallazgo más importante (descartados o fuente si Tipo B)
3. Qué hacer primero

Tono directo. Sin teoría ni recomendaciones no pedidas.
