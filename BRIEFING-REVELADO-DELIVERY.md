# Briefing para el agente que retoca la herramienta de revelado

Pega este archivo al agente de Dishlook que mantiene el cuarto oscuro (API **Google Nano Banana Pro / Gemini 3 Pro Image**). No es el plan de 30 días de visitas. No es el plan de UGC.

Objetivo: que la herramienta **revele el disparo real** y **exporte el encuadre que Glovo y Uber Eats aceptan**, sin alterar el plato y sin mencionar IA al cliente.

---

## Qué es y qué no es el lab

- Entrada: foto real disparada por el fotógrafo (iPhone) en el local.
- Trabajo: revelado tipo Lightroom (luz, color, recorte, suciedad menor de mesa). **El plato no se inventa ni se reescribe.**
- Salida: JPEGs listos para **Glovo**, **Uber Eats** y **carta / web / Instagram**.
- En público solo existe el después. Cero antes/después. Cero “filtro mágico”. Cero chispa visible de Gemini.
- SynthID (marca invisible de Google) **se hereda sí o sí** en todo output de Nano Banana Pro. No se quita. No se menciona al restaurante. Hoy Glovo/Uber **no publican** un baneo por SynthID; el riesgo para la cuenta del partner es el plato **no representativo** o el **encuadre malo**, no el detector invisible.
- Si alguien sube la entrega a Gemini y pregunta si la tocó Google AI, puede decir que sí. El pitch sigue siendo: “disparo yo y revelo con un flujo propio”. Nunca: “esto no tiene IA”.

---

## Lo que Glovo y Uber exigen de verdad (agosto 2026)

No piden un plato de supermercado sobre infinito blanco. Piden **foto fiel, un solo ítem, que sobreviva a su recorte**.

### Glovo (partners)

- Foto **realista y representativa**. Proporciones e ingredientes reales.
- El producto ocupa **~60–70 %** del marco.
- Muestran en **cuadrado 1:1**. Si subes un apaisado, **ellos recortan** y pueden cortar el bol.
- Recomendado en sus guías: ~**1000×1000**, JPG, a menudo **&lt; 1 MB** en tips de ES; en otras plazas admiten hasta ~10 MB. Entregar **1000–2000 px**, JPG sRGB, y un peso que no pise 1 MB si se puede sin matar calidad.
- Vista frontal o ~45°. Fondo limpio. Sin texto, logos, collages ni marca de agua **visible**.
- Sin manos/cuerpo. Plato en superficie, no sostenido.

### Uber Eats (fotos del local / Manager)

Norma oficial *Store submitted menu photo guidelines*:

- **Debe** representar con exactitud **un solo** ítem del menú.
- **Centrado**: el plato no puede ir a las esquinas ni fuera de cuadro.
- Relación de aspecto recomendada **entre 5:4 y 6:4** (apaisado). Mínimo ~550×440, máximo 10 000 px, ≤10 MB, JPG/PNG.
- **No:** varios platos (pizza + hamburguesa), personas (salvo manos), desenfoque, sombras duras / poca luz, entorno sucio, **logos o marcas de agua visibles**, texto.
- No mencionan IA. La regla es **representativo**. Las fotos de **clientes** en reseñas sí prohíben “AI-generated or heavily edited”; eso no es el Manager del restaurante.

“Centrada” = **zona segura para el crop de la app**, no “círculo perfecto en el medio”.

### Lo que tumba (o recorta mal) una foto

1. El plato no se parece a lo que sirven → quejas, bajan la foto, el dueño no repite. Eso protege la **cuenta del cliente**.
2. Desenfoque, suciedad, varios platos, texto/logo visible.
3. Hero editorial con mucho mármol/aire: en el thumbnail de Glovo el tartar se ve del tamaño de un garbanzo.

DoorDash (EE. UU.) ya rechaza “parece IA”. No es el canal de Barcelona; las normas pueden copiarse.

---

## Pipeline que tiene que implementar la herramienta

**No pidas a Nano Banana Pro un segundo pase “para centrar”.** Recentrar generativamente mueve o reescribe comida. El modelo **edita y vuelve a pintar píxeles**; cada pase extra es riesgo de cambiar el plato.

Orden fijo:

1. **Revelar una vez** (un solo generate/edit) sobre el disparo real.
2. **Recortar en código** (geométrico, no generativo) tres salidas.
3. Entregar al fotógrafo los tres archivos con nombres claros.

### Tres salidas por plato

| Archivo | Uso | Aspecto | Encuadre |
|---|---|---|---|
| `local-plato-master.jpg` | Carta, web, IG | El del disparo (3:2, 4:3 o el nativo) | Hero de estudio. Puede tener aire. |
| `local-plato-glovo.jpg` | Glovo | **1:1** | Bol/plato **centrado**, ocupa **60–70 %**. Margen ~15–20 % a cada lado para que el crop de la app no roce el borde. El **sujeto comestible** (el tartar, no el mármol) tiene que leerse a ~200 px. |
| `local-plato-uber.jpg` | Uber Eats | **5:4** apaisado | Mismo sujeto, no en esquinas. Un solo ítem. |

Nombres: `local-plato-01-master.jpg`, `…-glovo.jpg`, `…-uber.jpg`.

### Zona segura (para el prompt de revelado y para el crop)

Imagina un cuadrado en el centro del master:

- **Todo el plato y el bol** caben dentro de ese cuadrado.
- El alimento hero (cilindro de tartar, hamburguesa, etc.) ocupa la mitad central de ese cuadrado, no un punto perdido en un caldo enorme.
- Izquierda/derecha del cuadrado puede haber mesa: eso alimenta el 5:4 de Uber.
- Si el disparo viene muy ancho y el bol es pequeño, el revelado **no fabrica un bol más grande**. El crop se acerca. Si al acercar se corta comida, avisa al fotógrafo: “este disparo no tiene margen; hay que repetir o aceptar un recorte justo”.

---

## Prompt de revelado (Nano Banana Pro)

Usar como instrucción de **edición** (imagen de entrada = el disparo). No generar el plato desde texto.

```
You are a private darkroom for a real restaurant photograph.

NON-NEGOTIABLE
- This is a real photo of a real plated dish. Develop it. Do not reinvent it.
- Do not change ingredients, portion size, shape of the food, garnish count, or plating geometry.
- Do not add or remove food, sauce pools, roe, herbs, lemon, bread, or a second plate.
- Do not replace the plate, bowl, or table with a different one unless the user explicitly asked only to clean crumbs/stains on the table (not the food).
- Do not add text, logos, captions, frames, collages, or any visible watermark (no Gemini sparkle, no brand mark).
- Do not produce a before/after diptych. Output only the finished frame.
- Do not make the food look like a different recipe. Color grading must stay believable for tuna / meat / sauces (no neon, no plastic shine).

DEVELOPMENT (allowed)
- Exposure, white balance, gentle contrast, true-to-life color.
- Clean minor table crumbs or grease spots that are not part of the dish.
- Soften distracting background mess only if the dish itself does not change.
- Keep sanitary: no dirty cutlery, no filthy cloth in frame if it can be cropped out without touching the food.

FRAMING FOR DELIVERY (compose the developed frame so crops work)
- The dish is the only subject. Center the bowl/plate in the frame.
- Keep the entire bowl/plate inside a center safe square (nothing important at the corners or bleeding off the edge).
- The food should read at thumbnail size: the edible hero should be large, not a tiny island in a sea of table or broth.
- Leave a small even margin around the bowl (about 15%) so a 1:1 crop and a 5:4 crop both keep the full plate.
- Single menu item only. Accompanying garnish that is ON the same plate may stay. A second plate, drink, or extra bread basket should be cropped out if possible without cutting the main dish.
- Angle: keep the photographer’s angle (top-down or ~45°). Do not invent a new camera position that restages the food.

OUTPUT
- One finished photograph, no border, no text.
```

Si el fotógrafo pide **solo recorte de entrega** (el look ya está): **no llames a la API**. Recorta en código desde el master.

---

## Checklist automática antes de marcar “listo para Glovo/Uber”

El agente/herramienta debe fallar o avisar si:

- [ ] Hay texto, logo o marca visible.
- [ ] Se ven dos platos o un vaso que no es el ítem.
- [ ] El bol toca o se sale de una esquina.
- [ ] En un preview 200×200 no se entiende qué se come.
- [ ] El recorte 1:1 corta comida.
- [ ] Sombras duras que comen el plato, o está muy oscuro.
- [ ] Mesa sucia, cubiertos usados, entorno “poco sanitario” (criterio Uber).
- [ ] El revelado cambió ingredientes o la forma del emplatado respecto al disparo (comparar con la entrada).

Ejemplo ya visto (tartar de atún, el después): hero de estudio **válido**. El bol está centrado, un solo plato, mármol limpio, sin texto. **No subir el master tal cual a Glovo.** Exportar cuadrado más cerrado para que el cilindro + huevas se lean en el móvil; Uber 5:4 puede ir más cerca del master.

---

## Qué no toques

- No enseñes el lab ni Nano Banana Pro al restaurante.
- No generes platos de muestra atribuidos a un local inventado.
- No hagas un segundo generate “para que parezca menos IA”.
- No intentes quitar SynthID.
- No conviertas esto en un SaaS que el dueño usa solo.

---

## Entrega al fotógrafo

Por cada plato, un bloque listo para Notas / Drive:

```
plato: [nombre]
master: [archivo]  → web / IG / carta
glovo:  [archivo]  1:1  1000–2000px
uber:   [archivo]  5:4
avisos: [si el disparo no tenía margen / si el thumbnail no lee]
```
