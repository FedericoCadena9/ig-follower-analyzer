# Instagram Follower Analyzer

Herramienta para visualizar qué cuentas de Instagram no te siguen de vuelta.

## Uso

1. Abre `index.html` en tu navegador
2. Descarga tus datos de Instagram:
   - Instagram → Configuración → Tu actividad → Descargar información
   - Selecciona formato JSON
3. Carga los archivos:
   - `followers_*.json` en "Followers"
   - `following.json` en "Following"
   - (Opcional) Tu `whitelist.json`
4. Revisa los resultados y gestiona tu whitelist

## Características

- 📱 Mobile-first responsive design
- 🔍 Búsqueda y filtrado de resultados
- 📋 Whitelist persistente (localStorage)
- ⚠️ Detección de edge cases (duplicados, usernames cambiados, datos antiguos)
- 📥 Exportar whitelist como JSON

## Privacidad

- Todo el procesamiento ocurre localmente en tu navegador
- Ningún dato se envía a servidores externos
- Los archivos JSON nunca salen de tu dispositivo

## Estructura de archivos de Instagram

### followers.json (Formato A)
```json
[{ "title": "username", "string_list_data": [{ "href": "...", "value": "...", "timestamp": ... }] }]
```

### following.json (Formato B)
```json
{ "relationships_following": [{ "title": "username", "string_list_data": [...] }] }
```

## Tech Stack

- Vue 3 (CDN)
- Tailwind CSS (CDN)
- Sin dependencias ni build tools
