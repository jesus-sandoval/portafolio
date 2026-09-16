# NutriDeck — spec para pasar del wireframe a la app real (Expo / React Native)

> Contexto: este documento resume las decisiones tomadas durante la sesión de
> wireframing en `diet-app/index.html` (HTML/CSS/JS puro, ver ese archivo como
> referencia visual e interactiva exacta de lo que hay que replicar). El
> objetivo es construir la versión real como app de iPhone con Expo, probando
> gratis en el teléfono vía Expo Go antes de pagar la cuenta de Apple Developer.

## Qué es la app

App personal (más adelante, opcionalmente, compartible con un nutriólogo/
paciente) para llevar el control diario de las comidas de una dieta, en vez
de un PDF/Word aburrido. Interacción central: un carrusel horizontal de días
arriba, y debajo un "rolodex" vertical de comidas del día que se completan
con swipe.

## Stack elegido

- **Expo (React Native)** — permite probar en iPhone real con la app Expo Go
  sin pagar nada, hasta que se quiera publicar en App Store.
- `react-native-gesture-handler` + `react-native-reanimated` — para el swipe
  de las cards.
- `@expo/vector-icons` — para los íconos (Material Symbols Rounded).
- `@expo-google-fonts/fira-code` y `@expo-google-fonts/inter` — tipografías.
- `@react-native-async-storage/async-storage` — reemplazo de `localStorage`
  para persistencia local (ya se decidió NO usar backend todavía; Supabase o
  Firebase se evaluará después).

## Diseño — tokens

Estilo minimalista tipo Notion/Claude: cards planas (sin 3D ni sombras duras),
colores pastel en las cards como acento fuerte, resto de la UI neutro.

**Tipografía**
- Títulos, números, horas → `Fira Code` (weight 500/600/700)
- Todo lo demás (cuerpo, labels) → `Inter` (400/500/600/700)

**Colores base (claro / oscuro)**
```
bg:          #ffffff / #191919
bg-raised:   #ffffff / #202020
bg-sunken:   #f7f7f5 / #1f1f1f
text:        #37352f / #e9e9e7
text-dim:    #9b9a97 / #8f8d88
border:      #eceae7 / #2f2f2f
```

**Colores de comida (bg / texto, claro → oscuro)**
```
1 (ámbar / desayuno):  #f4e9d8 #6b4a1e  →  #332a1a #e8c989
2 (verde / colación):  #dceae3 #1f5c46  →  #1c2f28 #8fd9bd
3 (azul / comida):     #dce6f0 #244a73  →  #1e2c3a #9cc2e8
4 (violeta / colación):#e6e0f0 #4a3b73  →  #292236 #c7b7e8
5 (rosa / cena):       #f2dede #7a3030  →  #332222 #e3a8a8
```

**Modo claro/oscuro**: toggle manual en la barra superior, persistido por
usuario (no hace falta sincronizar entre dispositivos).

## Layout

Una sola columna centrada, **igual en móvil y en pantallas grandes** (no hay
breakpoint de "vista de escritorio" separada): scroll vertical con, de arriba
a abajo:
1. Barra superior (logo + toggle de tema)
2. Rail de días (horizontal, scrolleable, con paginación ± semanas y acceso
   a un calendario mensual completo)
3. Stack de comidas del día ("rolodex")
4. Fila de comidas ya completadas (chips)
5. Botones "Agregar comida" / "Set dieta"
6. Panel de resumen: progreso semanal, avance del día, notificaciones, plan
   activo (con botón de compartir)
7. Barra de navegación flotante fija abajo (estilo Instagram)

## Funcionalidad — rail de días + calendario

- El rail muestra una ventana de ±7 días alrededor del día activo, navegable
  con scroll horizontal y botones ‹ › que desplazan la ventana 7 días.
- Ícono de calendario en el navbar abre un **modal de mes completo**
  (grid de 6 semanas, navegación entre meses, indicador de días completos).
  Tocar un día salta la app a esa fecha.
- Los datos de comidas se generan/cargan de forma perezosa por fecha (no
  existe un array fijo de "esta semana"): cualquier fecha pasada, presente o
  futura tiene su propio set de comidas.

## Funcionalidad — stack de comidas (el corazón de la app)

- Comidas pendientes del día se muestran apiladas: la de **enfrente** es la
  actual (grande, interactiva); las siguientes se abren **en abanico hacia
  arriba** detrás de ella (peek), cada vez un poco más chicas/tenues — y son
  **tocables** para traerlas al frente sin necesidad de deslizar.
- **Swipe izquierda** en la card de enfrente → **completar** (se colapsa a
  un chip tachado en la fila de abajo; tocar el chip la regresa a pendiente).
- **Swipe derecha** en la card de enfrente → **"después"**: la manda al final
  de la fila sin completarla (también sirve para hojear las comidas).
- Además de los gestos, hay botones pequeños (check / skip) para accesibilidad
  y para desktop/mouse.
- **Fuera de horario**: si ya pasó ~45 min de la hora programada (o es un día
  anterior), el tag de la card cambia a "fuera de horario" con ícono de
  reloj, pero la card **sigue siendo 100% completable o saltable
  manualmente** — el usuario decide si la marca hecha tarde o la salta.
- Card frontal muestra macros (kcal / proteína) con un badge "IA" — hoy son
  datos de mentira; la idea real es estimarlos con un LLM (Claude) a partir
  de la descripción del platillo (o foto), o cruzarlos con una base de datos
  nutricional (USDA FoodData Central / Edamam / Nutritionix) cuando haya match.

## Funcionalidad — notificaciones

- Preferencia de aviso antes de cada comida: **10 minutos antes por defecto**,
  configurable a 30/15/5/0 min o desactivadas. Guardar la preferencia
  localmente. (La implementación real usará `expo-notifications` para
  notificaciones locales programadas por comida.)

## Funcionalidad — compartir dieta

- Modal con link mock + botón "copiar", y opción de invitar por correo.
  Pensado para que un nutriólogo comparta el plan con su paciente (o
  viceversa). **No se construye el rol nutriólogo/paciente todavía** — se
  deja para después de tener sólida la experiencia personal de un solo
  usuario.

## Navbar flotante (bottom, estilo Instagram)

Íconos (Material Symbols Rounded, weight 300):
1. `calendar_month` — abre el modal de mes completo
2. `restaurant` — scroll a "Hoy toca" (seleccionado por defecto)
3. `add_circle` — abre el flujo de agregar comida
4. `bar_chart` — scroll al resumen
5. `person` — scroll al plan activo

**Regla de estilo importante**: no todos los íconos de Google tienen par
outline/filled distinguible, así que en el navbar **todos los íconos se
renderizan filled siempre**; lo que indica selección es un **fondo tipo pill
+ color más oscuro**, no el eje FILL. El estado seleccionado es persistente
(no depende de mantener presionado ni de hover) y se mantiene hasta que se
toca otro ícono del grupo. `calendar_month`, `restaurant`, `bar_chart` y
`person` pertenecen al mismo grupo de selección mutuamente excluyente
(tocar calendario lo deja marcado aunque abra el popup). `add_circle` nunca
tiene estado seleccionado, solo dispara una acción.

## Persistencia

- **Ahora**: todo en `localStorage` (web) / `AsyncStorage` (Expo), sin
  backend. Suficiente para pruebas personales diarias.
- **Después**: Supabase o Firebase, cuando se quiera sincronizar entre
  dispositivos o dar acceso a otra persona (nutriólogo/paciente).

## Modelo de datos (referencia)

```js
// clave: "YYYY-MM-DD"
daysData[dateKey] = {
  pending: [
    { id, time, tag, name, desc, kcal, protein, color },
    ...
  ],
  done: [ /* mismos objetos, ya completados */ ]
}
```

## Plan de construcción, paso a paso

1. Instalar Node.js (LTS) y la app Expo Go en el iPhone. ✅ (Expo Go ya
   instalada por el usuario)
2. `npx create-expo-app nutrideck` → `cd nutrideck` → `npx expo start`,
   escanear el QR con la cámara del iPhone para verla corriendo ahí mismo.
3. Instalar dependencias:
   `npx expo install react-native-gesture-handler react-native-reanimated
   @expo-google-fonts/fira-code @expo-google-fonts/inter expo-font
   @react-native-async-storage/async-storage`
4. Traducir cada sección del wireframe a un componente, apoyándose en
   `diet-app/index.html` como referencia exacta de comportamiento:
   - `DayRail.js` (ScrollView horizontal + botones de paginación)
   - `MealStack.js` (el rolodex — PanGestureHandler + Animated)
   - `MonthModal.js` (Modal nativo con el grid de 6 semanas)
   - `ShareModal.js`, `NavBar.js`, `SettingsPanel.js`, etc.
   - Theming claro/oscuro con `useColorScheme` + contexto propio para el
     toggle manual.
5. Probar constantemente en el iPhone vía Expo Go (hot reload automático).
6. (Más adelante) Conectar Supabase para persistencia en la nube.
7. (Solo al querer publicar) Cuenta de Apple Developer ($99/año) +
   `eas build` de Expo para generar el `.ipa` y subirlo a TestFlight/App Store.

**Estamos en el paso 1 → 2.** El siguiente entregable es el primer componente
real de React Native (`MealStack.js` con el gesto de swipe), una vez que el
proyecto Expo esté creado en la máquina local.
