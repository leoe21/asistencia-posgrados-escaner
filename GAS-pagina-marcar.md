# Añadir la página "marcar" en Google Apps Script

Cuando el escáner (GitHub Pages) lee un QR, redirige al usuario a tu Web App con `?page=marcar&id=CODIGO&volver=URL`. Para que eso funcione, tu script debe atender esa ruta y devolver una página con el resultado.

## Qué añadir en `Código.gs`

Dentro de tu función **`doGet(e)`**, después de los `if (page === 'confirmar' ...)` y `if (page === 'escaner' ...)`, añade este bloque:

```javascript
  // Página de resultado al escanear desde GitHub Pages (o cualquier cliente externo)
  if (page === 'marcar' && id) {
    var res = marcarLlegada(id);
    var volver = params.volver || '';
    var urlVolver = volver ? decodeURIComponent(volver) : (ScriptApp.getService().getUrl() + '?page=escaner');
    var html = '<!DOCTYPE html><html><head><meta charset="UTF-8"><meta name="viewport" content="width=device-width, initial-scale=1">';
    html += '<title>Registro de llegada</title>';
    html += '<style>body{font-family:sans-serif;text-align:center;padding:2rem;max-width:400px;margin:0 auto;} .ok{color:#155724;background:#d4edda;padding:1rem;border-radius:8px;} .warn{color:#856404;background:#fff3cd;padding:1rem;border-radius:8px;} .err{color:#721c24;background:#f8d7da;padding:1rem;border-radius:8px;} a{color:#1a73e8;}</style></head><body>';
    if (res && res.ok) {
      html += '<div class="ok"><p><strong>Llegada registrada</strong></p><p>' + (res.nombre || '') + '</p><p>' + (res.fechaLlegada ? new Date(res.fechaLlegada).toLocaleString() : '') + '</p></div>';
    } else if (res && res.repetido) {
      html += '<div class="warn"><p><strong>Ya estaba registrado</strong></p><p>' + (res.nombre || '') + '</p><p>' + (res.fechaLlegada ? new Date(res.fechaLlegada).toLocaleString() : '') + '</p></div>';
    } else {
      html += '<div class="err"><p>No se encontró a nadie con ese código.</p></div>';
    }
    html += '<p style="margin-top:1.5rem;"><a href="' + urlVolver + '">Volver al escáner</a></p></body></html>';
    return HtmlService.createHtmlOutput(html)
      .setTitle('Registro de llegada')
      .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL);
  }
```

Asegúrate de que la variable `params` esté definida al inicio de `doGet` (por ejemplo `var params = e && e.parameter ? e.parameter : {};`) y que uses la misma `id` que obtienes de `params.id`.

## Orden sugerido en `doGet`

1. `var params = e && e.parameter ? e.parameter : {};`
2. `var page = params.page || '';` y `var id = params.id || '';`
3. `if (page === 'confirmar' && id) { ... }`
4. `if (page === 'escaner') { ... }`
5. **`if (page === 'marcar' && id) { ... }`**  ← el bloque de arriba
6. `return HtmlService.createHtmlOutput('Página no encontrada...');`

Después de editar, guarda y crea una **nueva versión** de la implementación (Implementar → Administrar implementaciones → Editar → Nueva versión).
