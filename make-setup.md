# Configuración Make — Guías Turísticas UCA

## Escenario: 4 módulos en cadena

```
[Webhook] → [HTTP Gemini] → [GitHub API] → [Gmail]
```

---

## Módulo 1 — Webhook (Custom Webhook)

- Tipo: **Webhooks > Custom webhook**
- Nombre: `recibir-formulario-uca`
- Copiar la URL generada → pegarla en `index.html` como valor de `MAKE_WEBHOOK_URL`
- Hacer clic en "Determine data structure" y enviar el formulario una vez para que Make detecte los campos.

---

## Módulo 2 — HTTP: llamada a Gemini

- Tipo: **HTTP > Make a request**
- URL: `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=TU_API_KEY_GEMINI`
- Método: `POST`
- Headers: `Content-Type: application/json`
- Body type: `Raw`
- Content type: `JSON`
- Body (copiar exactamente):

```json
{
  "contents": [
    {
      "parts": [
        {
          "text": "Eres un experto en turismo y diseño web. Genera una guía turística interactiva completa en HTML para el siguiente viajero:\n\n- Nombre: {{1.nombre}}\n- Destino: {{1.destino}}\n- Llegada: {{1.llegada}}\n- Salida: {{1.salida}}\n- Duración: {{1.dias}} días\n- Viajeros: {{1.viajeros}}\n- Presupuesto: {{1.presupuesto}}\n- Alojamiento preferido: {{1.alojamiento}}\n- Intereses: {{1.intereses}}\n- Ritmo (1=tranquilo, 5=intenso): {{1.ritmo}}/5\n- Notas especiales: {{1.notas}}\n\nGenera ÚNICAMENTE código HTML completo (desde <!DOCTYPE html> hasta </html>) con estas características:\n\n1. CSS inline en etiqueta <style> — sin archivos externos excepto Google Fonts\n2. Diseño responsive y moderno, funciona perfecto en móvil\n3. Colores inspirados en el destino (mediterráneo para España, cálidos para Asia, etc.)\n4. Navegación por pestañas con JavaScript vanilla: Resumen, Itinerario, Gastronomía, Alojamiento, Consejos\n5. Itinerario día a día con mañana/tarde/noche para cada día\n6. Sección de gastronomía: platos típicos y restaurantes recomendados por precio\n7. Sección de alojamiento: 3 opciones según el presupuesto indicado\n8. Consejos prácticos: transporte, moneda, clima, seguridad, frases útiles\n9. Header con el nombre del destino y un emoji representativo\n10. Footer con 'Guía generada por Universidad de Cádiz · UCA'\n11. Botón de imprimir la guía\n12. Todo el contenido en español\n\nNo incluyas texto antes ni después del HTML. Solo el código HTML completo."
        }
      ]
    }
  ],
  "generationConfig": {
    "temperature": 0.8,
    "maxOutputTokens": 8192
  }
}
```

- En `{{1.campo}}` Make sustituye los datos del webhook. Ajusta los números si el módulo Webhook no es el módulo 1.

**Cómo obtener la API Key de Gemini (gratis):**
1. Ir a https://aistudio.google.com/app/apikey
2. Click "Create API key"
3. Copiar y pegar en la URL del módulo 2

---

## Módulo 3 — GitHub API: publicar la guía

- Tipo: **HTTP > Make a request** (usamos la API REST de GitHub directamente)
- URL: `https://api.github.com/repos/epratsbeltran-cpu/Guias-uca/contents/guides/{{1.destino}}-{{formatDate(now; "YYYYMMDD-HHmm")}}.html`
- Método: `PUT`
- Headers:
  - `Authorization`: `token {{GITHUB_TOKEN}}` ← usar variable de Make (ver nota abajo)
  - `Content-Type`: `application/json`
  - `Accept`: `application/vnd.github.v3+json`
- Body (Raw JSON):

```json
{
  "message": "Nueva guía: {{1.destino}}",
  "content": "{{base64(2.data.candidates[0].content.parts[0].text)}}"
}
```

**Notar:** `2.` se refiere al módulo HTTP de Gemini (módulo 2). Ajustar si cambia el orden.

**Repositorio ya configurado:** `https://github.com/epratsbeltran-cpu/Guias-uca`
**GitHub Pages activado:** `https://epratsbeltran-cpu.github.io/Guias-uca/`

La URL de cada guía será: `https://epratsbeltran-cpu.github.io/Guias-uca/guides/[destino]-[fecha].html`

> **Nota — token de GitHub en Make:**
> En Make ve a **Variables** (icono engranaje) → añade variable `GITHUB_TOKEN` con tu token personal.
> El token es: el que generaste con scope `repo` (no lo pongas nunca en texto plano en archivos públicos).

---

## Módulo 4 — Gmail: enviar la guía

- Tipo: **Gmail > Send an email**
- To: `{{1.email}}`
- Subject: `✈️ Tu guía de {{1.destino}} está lista, {{1.nombre}}`
- Content type: `HTML`
- Body:

```html
<div style="font-family:Inter,Arial,sans-serif;max-width:560px;margin:0 auto">
  <div style="background:#003087;padding:28px 32px;border-radius:12px 12px 0 0">
    <p style="color:white;font-size:13px;margin:0;opacity:.8">Universidad de Cádiz · Guías Turísticas</p>
    <h1 style="color:#f0b323;font-size:24px;margin:8px 0 0">✈️ Tu guía de {{1.destino}} está lista</h1>
  </div>
  <div style="background:#f5f7fc;padding:28px 32px;border-radius:0 0 12px 12px;border:1px solid #d0d9ea;border-top:none">
    <p style="color:#1a1a2e;font-size:16px">Hola <strong>{{1.nombre}}</strong>,</p>
    <p style="color:#5a6072;font-size:15px;line-height:1.6">Hemos preparado tu guía personalizada para <strong>{{1.dias}} días en {{1.destino}}</strong>. Incluye itinerario día a día, gastronomía, alojamiento y consejos prácticos.</p>
    <a href="https://epratsbeltran-cpu.github.io/Guias-uca/guides/{{1.destino}}-{{formatDate(now; "YYYYMMDD-HHmm")}}.html"
       style="display:inline-block;background:#003087;color:white;padding:14px 28px;border-radius:8px;text-decoration:none;font-weight:700;font-size:15px;margin:16px 0">
      Ver mi guía →
    </a>
    <p style="color:#5a6072;font-size:13px;margin-top:20px">El enlace funciona en móvil y PC. Puedes imprimirla desde la propia guía.</p>
    <hr style="border:none;border-top:1px solid #d0d9ea;margin:20px 0"/>
    <p style="color:#999;font-size:12px">Universidad de Cádiz · Servicio gratuito de guías turísticas</p>
  </div>
</div>
```

---

## Problema CORS del webhook (solución)

El formulario usa `fetch` para enviar al webhook. Make responde con `200 OK` pero sin cabecera CORS, lo que hace que el navegador muestre error de red — aunque Make **ya recibió los datos**. El código en `index.html` captura ese error silenciosamente con `try/catch` y muestra la confirmación de todas formas. No es un problema real.

Si quieres eliminar el error: en Make, en el módulo Webhook → Respuestas inmediatas → activar "Return response immediately" y configurar respuesta con header `Access-Control-Allow-Origin: *`.
