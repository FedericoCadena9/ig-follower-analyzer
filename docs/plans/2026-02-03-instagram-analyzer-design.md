# Instagram Follower Analyzer - Design Document

**Fecha:** 2026-02-03
**Estado:** Aprobado
**Objetivo:** Herramienta para visualizar qué cuentas de Instagram no te siguen de vuelta

---

## 1. Resumen Ejecutivo

Aplicación web single-page que compara archivos JSON exportados de Instagram (followers y following) para identificar cuentas que no te siguen de vuelta. Incluye sistema de whitelist para ignorar creadores/páginas que sigues intencionalmente.

### Decisiones Clave

| Aspecto | Decisión | Razón |
|---------|----------|-------|
| Enfoque | Solo visualización | Evita riesgos de bloqueo de cuenta por automatización |
| Stack | Vue 3 + Tailwind (CDN) | Sin build tools, máxima simplicidad |
| Comparación | Por `href` como key única | Funciona aunque usuario cambie username |
| Persistencia | localStorage | Whitelist persiste entre sesiones |

---

## 2. Investigación: API de Instagram

### Hallazgos

- **No existe API oficial** para unfollow programático
- Métodos no oficiales (Instagrapi, scripts de consola) conllevan riesgo de bloqueo
- Instagram detecta y penaliza acciones automatizadas
- No hay parámetro URL para eliminar seguidor directamente

### Fuentes
- [InstagramUnfollowers (GitHub)](https://github.com/davidarroyo1234/InstagramUnfollowers)
- [Instagram Unfollowers 2026](https://github.com/EdvinCodes/InstagramUnfollowers)
- [Mass Unfollow Guide](https://wilhelm.codes/blog/how-to-mass-unfollow-instagram-accounts/)

---

## 3. Arquitectura

```
┌─────────────────────────────────────────────────────────────┐
│                    Instagram Follower Analyzer               │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │ followers.json│  │ following.json│  │ whitelist.json│     │
│  │   (drop zone) │  │   (drop zone) │  │  (opcional)  │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘       │
│         │                 │                 │                │
│         ▼                 ▼                 ▼                │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │              JSON Parser & Normalizer                    │ │
│  │   - Detecta formato A o B automáticamente               │ │
│  │   - Normaliza a estructura común                         │ │
│  │   - Valida y detecta edge cases                          │ │
│  └──────────────────────┬──────────────────────────────────┘ │
│                         ▼                                    │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │              Comparison Engine                           │ │
│  │   - Compara por href (identificador único)              │ │
│  │   - Aplica whitelist                                     │ │
│  │   - Genera warnings de edge cases                        │ │
│  └──────────────────────┬──────────────────────────────────┘ │
│                         ▼                                    │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │              Results Table + Warnings Panel              │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## 4. Formatos de Datos Soportados

### Formato A: followers.json (Array directo)

```json
[
  {
    "title": "username",
    "media_list_data": [],
    "string_list_data": [
      {
        "href": "https://www.instagram.com/username",
        "value": "username",
        "timestamp": 1762128539
      }
    ]
  }
]
```

### Formato B: following.json (Objeto con key contenedora)

```json
{
  "relationships_following": [
    {
      "title": "username",
      "string_list_data": [
        {
          "href": "https://www.instagram.com/username",
          "value": "username",
          "timestamp": 1762128539
        }
      ]
    }
  ]
}
```

### Estructura Normalizada Interna

```typescript
interface NormalizedUser {
  username: string;      // value del JSON
  href: string;          // URL del perfil (key única)
  timestamp: number;     // cuándo se siguió
}
```

---

## 5. UI/UX Mobile-First

### Principios

- **Mobile-first**: Diseñar para 320px primero, escalar hacia arriba
- **Touch-friendly**: Botones mínimo 44x44px
- **Progressive disclosure**: Mostrar lo esencial, revelar detalles bajo demanda

### Layout Mobile (< 768px)

```
┌─────────────────────────┐
│  📊 IG Analyzer         │  ← Header fijo
├─────────────────────────┤
│ ┌─────────────────────┐ │
│ │  📁 Cargar Followers │ │  ← Botones grandes
│ │     (tap to upload) │ │     apilados verticalmente
│ └─────────────────────┘ │
│ ┌─────────────────────┐ │
│ │  📁 Cargar Following │ │
│ └─────────────────────┘ │
│ ┌─────────────────────┐ │
│ │  ⚪ Whitelist (opc.) │ │
│ └─────────────────────┘ │
├─────────────────────────┤
│ ⚠️ 3 warnings [ver]     │  ← Banner colapsable
├─────────────────────────┤
│ 🔍 [  Buscar...      ]  │  ← Search sticky
│ Ordenar: [Fecha ▼]      │
├─────────────────────────┤
│ 📋 142 no te siguen     │
├─────────────────────────┤
│ ┌─────────────────────┐ │
│ │ @usuario1           │ │  ← Cards en vez de tabla
│ │ Seguido: 15 Ene 24  │ │
│ │ [Ir perfil] [+WL]   │ │
│ └─────────────────────┘ │
└─────────────────────────┘
```

### Layout Desktop (≥ 768px)

```
┌────────────────────────────────────────────────────────────┐
│  📊 Instagram Follower Analyzer                             │
├────────────────────────────────────────────────────────────┤
│  ┌──────────┐ ┌──────────┐ ┌──────────┐  ⚠️ 3 warnings    │
│  │Followers │ │Following │ │Whitelist │  [ver detalles]   │
│  │ ✅ 1,204 │ │ ✅ 892   │ │ 📝 23    │                    │
│  └──────────┘ └──────────┘ └──────────┘                    │
├────────────────────────────────────────────────────────────┤
│ 🔍 [Buscar username...]  Ordenar: [Fecha ▼]  │ 142 result. │
├────────────────────────────────────────────────────────────┤
│ Username      │ Fecha seguido │ Acciones                   │
│───────────────┼───────────────┼────────────────────────────│
│ @usuario1     │ 15 Ene 2024   │ [Ir a perfil] [+ Whitelist]│
│ @usuario2     │ 03 Dic 2023   │ [Ir a perfil] [+ Whitelist]│
└────────────────────────────────────────────────────────────┘
```

---

## 6. Edge Cases y Warnings

### Detección

| Edge Case | Cómo se detecta | Severidad |
|-----------|-----------------|-----------|
| JSON malformado | `try/catch` en `JSON.parse()` | 🔴 Error (bloqueante) |
| Archivo vacío | Array length === 0 | 🔴 Error (bloqueante) |
| Duplicados | Mismo `href` aparece 2+ veces | 🟡 Warning |
| Username cambiado | Mismo `href`, diferente `value` | 🟡 Warning |
| Datos desactualizados | Timestamp más reciente > 6 meses | 🟠 Info |
| Cuenta posiblemente eliminada | `href` sin formato estándar | 🟡 Warning |

### UI de Warnings

```
┌─────────────────────────────────────────────────────────────┐
│ ⚠️ Se encontraron 3 problemas                    [Colapsar] │
├─────────────────────────────────────────────────────────────┤
│ 🟡 WARNING                                                  │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ 2 usuarios cambiaron su username                        │ │
│ │ • juan_2023 → juan_nuevo (mismo perfil)                │ │
│ │ 💡 Se usará el username más reciente para comparar     │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
│ 🟠 INFO                                                     │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ Datos tienen más de 6 meses de antigüedad               │ │
│ │ 💡 Considera descargar datos frescos desde Instagram   │ │
│ └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Estilos Tailwind para Severidades

```javascript
const warningStyles = {
  error: 'bg-red-50 border-l-4 border-red-500 text-red-800',
  warning: 'bg-amber-50 border-l-4 border-amber-500 text-amber-800',
  info: 'bg-blue-50 border-l-4 border-blue-500 text-blue-700'
}
```

---

## 7. Gestión de Whitelist

### Fuentes de Datos

```
┌─────────────┐      ┌─────────────┐      ┌───────────┐
│ JSON import │  +   │ localStorage│  +   │ UI manual │
│ (inicial)   │      │ (persistido)│      │ (on-the-go)│
└──────┬──────┘      └──────┬──────┘      └─────┬─────┘
       └────────────────────┼────────────────────┘
                            ▼
                 ┌─────────────────────┐
                 │   WHITELIST FINAL   │
                 │   (Set combinado)   │
                 └─────────────────────┘
```

### Estructura localStorage

```javascript
// Key: 'ig-analyzer-whitelist'
{
  "version": 1,
  "updatedAt": "2026-02-03T10:30:00Z",
  "usernames": ["creador1", "pagina_favorita"]
}
```

### Formato whitelist.json (importable)

```json
// Formato simple
["creador1", "pagina_favorita"]

// O con metadata
{
  "usernames": ["creador1", "pagina_favorita"],
  "note": "Mi whitelist exportada"
}
```

### Acciones UI

- Añadir username manualmente
- Añadir desde tabla de resultados (botón "+ Whitelist")
- Eliminar de whitelist
- Exportar whitelist como JSON
- Toast de confirmación con opción "Deshacer"

---

## 8. Lógica de Comparación

### Algoritmo

```javascript
// 1. Normalizar ambos archivos a Maps
const followersMap = new Map(); // href → NormalizedUser
const followingMap = new Map(); // href → NormalizedUser

// 2. Comparar usando href como key primaria
const notFollowingBack = [];

for (const [href, user] of followingMap) {
  const isFollower = followersMap.has(href);
  const isWhitelisted = whitelist.has(user.username.toLowerCase());

  if (!isFollower && !isWhitelisted) {
    notFollowingBack.push(user);
  }
}

// 3. Ordenar por timestamp (más recientes primero por defecto)
notFollowingBack.sort((a, b) => b.timestamp - a.timestamp);
```

### Rendimiento

- **Maps para O(1) lookups** en vez de arrays O(n)
- **Procesar en chunks** para no bloquear UI con miles de registros
- **Virtual scrolling** considerado pero no necesario para <10k registros

---

## 9. Estructura de Archivos

```
ig-follower-analyzer/
├── index.html          # App completa (Vue + Tailwind CDN)
├── README.md           # Instrucciones de uso
├── docs/
│   └── plans/
│       └── 2026-02-03-instagram-analyzer-design.md
└── sample-data/        # Datos de ejemplo para testing
    ├── followers.json
    ├── following.json
    └── whitelist.json
```

---

## 10. Componentes Vue

```
App (root)
├── HeaderSection
│   └── Logo + título
├── FileUploadSection
│   ├── DropZone (followers)
│   ├── DropZone (following)
│   └── DropZone (whitelist - opcional)
├── WarningsPanel
│   └── WarningCard (×n)
├── WhitelistManager
│   ├── AddUsernameInput
│   ├── WhitelistItem (×n)
│   └── ExportButton
├── ResultsSection
│   ├── SearchBar
│   ├── SortDropdown
│   ├── ResultsCount
│   └── ResultsList
│       ├── ResultCard (mobile)
│       └── ResultTable (desktop)
└── ToastNotification
```

---

## 11. Flujo de Usuario

```
1. Usuario abre index.html en navegador
              ▼
2. Carga followers.json (drag & drop o click)
              ▼
3. Carga following.json (drag & drop o click)
              ▼
4. [Opcional] Carga whitelist.json
              ▼
5. App procesa y muestra warnings si hay
              ▼
6. Tabla/cards muestran quién no te sigue
              ▼
7. Usuario puede:
   - Buscar/filtrar por username
   - Ordenar por fecha
   - Click "Ir a perfil" → abre Instagram
   - Click "+ Whitelist" → añade y oculta
   - Exportar whitelist actualizada
```

---

## 12. Criterios de Aceptación

- [ ] Cargar followers.json formato A (array directo)
- [ ] Cargar following.json formato B (objeto con key)
- [ ] Detectar y mostrar warnings visuales prominentes
- [ ] Whitelist: importar JSON, añadir manual, persistir en localStorage
- [ ] Tabla con búsqueda y ordenamiento por fecha
- [ ] Mobile: cards touch-friendly (min 44px)
- [ ] Desktop: tabla tradicional
- [ ] Botón "Ir a perfil" abre Instagram en nueva pestaña
- [ ] Botón "+ Whitelist" con toast y opción deshacer
- [ ] Exportar whitelist como JSON
