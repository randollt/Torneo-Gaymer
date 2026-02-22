# 🎮 Apertura Gaymers — Guía de setup completo

## Estructura de archivos
```
apertura-gaymers/
├── index.html              ← La app completa
├── database.rules.json     ← Reglas de seguridad Firebase
├── functions/
│   ├── index.js            ← Cloud Function (proxy API Football)
│   └── package.json
└── README.md
```

---

## ✅ Paso 1 — Subir index.html a GitHub Pages

1. Entrá a github.com → tu repo `apertura`
2. Subí `index.html` (reemplazá el anterior)
3. GitHub Pages lo publica automáticamente

---

## ✅ Paso 2 — Actualizar reglas de Firebase

1. Abrí Firebase Console → Realtime Database → Reglas
2. Reemplazá todo el contenido con lo que está en `database.rules.json`
3. Publicar

---

## ✅ Paso 3 — Activar Google Auth en Firebase

1. Firebase Console → Authentication → Sign-in method
2. Activar **Google** → Guardar
3. En "Authorized domains" asegurate de que esté tu dominio de GitHub Pages

---

## ✅ Paso 4 — Configurar Cloud Functions (desde la PC)

### 4.1 Instalar dependencias
```bash
npm install -g firebase-tools
firebase login
```

### 4.2 Inicializar en la carpeta del proyecto
```bash
cd apertura-gaymers
firebase init
```
Seleccioná:
- ✅ Functions
- ✅ Database
- Proyecto: apertura-tabla
- Language: JavaScript
- ESLint: No

### 4.3 Guardar tu API Key de forma segura
```bash
firebase functions:config:set apifootball.key="TU_API_KEY_AQUI"
```
(Reemplazá TU_API_KEY_AQUI con la key que te dio api-football.com)

### 4.4 Instalar dependencias de Functions
```bash
cd functions
npm install
cd ..
```

### 4.5 Deploy
```bash
firebase deploy --only functions
```

Cuando termine, te va a dar una URL como:
`https://us-central1-apertura-tabla.cloudfunctions.net/bocaFixtures`

### 4.6 Actualizar la URL en index.html
Abrí `index.html` y buscá esta línea:
```js
const PROXY_URL = "https://us-central1-apertura-tabla.cloudfunctions.net/bocaFixtures";
```
Confirmá que la URL sea exactamente la que te dio Firebase. Si es igual, no necesitás cambiar nada.

---

## ✅ Paso 5 — Autorizarte como admin

1. Abrí tu app en GitHub Pages
2. Iniciá sesión con **tu** cuenta de Google
3. El sistema te va a mostrar el selector de equipo — elegí "Admin" o tu nombre
4. Abrí la consola del navegador (F12 → Console) y ejecutá esto **una sola vez** para darte rol de admin:

```javascript
// Solo necesitás ejecutar esto una vez desde la consola
import("https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js").then(async ({getDatabase,ref,get,set})=>{
  const db = getDatabase();
  const snap = await get(ref(db,"autorizados"));
  const users = snap.val()||{};
  const myEntry = Object.entries(users).find(([k,v])=>v.email==="TU_EMAIL@gmail.com");
  if(myEntry) await set(ref(db,`autorizados/${myEntry[0]}/role`),"admin");
  console.log("✅ Admin listo");
});
```
(Reemplazá TU_EMAIL@gmail.com con tu email real)

---

## ✅ Paso 6 — Los jugadores se registran solos

Cada jugador:
1. Entra a la URL de la app
2. Hace clic en **"Entrar"** → botón de Google
3. Inicia sesión con su cuenta de Gmail
4. Ve la pantalla: **"¿Cuál es tu nombre en el torneo?"** con los nombres disponibles
5. Hace clic en su nombre → ¡listo!

Los nombres ya tomados aparecen en rojo como "Ocupado" para evitar duplicados.

---

## 🔐 Anti-fraude implementado

- ✅ Solo cuentas autorizadas por el admin pueden acceder
- ✅ Los pronósticos se cierran automáticamente a la hora del partido
- ✅ Una cuenta de Google = un solo voto, imposible duplicar
- ✅ Los pronósticos ajenos están ocultos hasta que empieza el partido
- ✅ El admin puede sumar/no sumar puntos manualmente si alguien olvidó cargar

---

## 🎯 Panel Admin — atajos

| Acción | Cómo |
|--------|------|
| Abrir/cerrar panel admin | Ctrl+Shift+A |
| Editar puntos de un equipo | Panel admin → Editar equipo |
| Agregar jugador | Panel admin → Jugadores autorizados |
| Procesar resultado | Panel admin → Resultado real |

---

## ❓ Preguntas frecuentes

**¿Qué pasa si un jugador no cargó su pronóstico?**
El admin puede sumarle o no sumarle puntos manualmente desde "Editar equipo" en el panel admin.

**¿Cada cuánto se actualizan los partidos de Boca?**
La Cloud Function llama a la API cada vez que se abre la pestaña "Partidos de Boca". La API gratuita tiene 100 requests/día, más que suficiente.

**¿Los datos en vivo se actualizan solos?**
Sí, cuando Boca está jugando, podés hacer clic en "🔄 Actualizar" o recargar la pestaña para ver el minuto y los goles actualizad
