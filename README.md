# Navegador de misiones lunares CLPS

Página web interactiva que compara, para sitios del polo sur de la Luna, cuánto tiempo ve cada uno el **Sol** y la **Tierra** sobre su horizonte. Sirve para explorar por qué la iluminación (energía solar, temperatura) y la vista directa de la Tierra (comunicaciones) son dos de los factores que más pesan al elegir dónde aterrizar en el polo sur, como en las misiones comerciales del programa CLPS de la NASA.

**Ver en vivo:** https://xscarletx.github.io/navegador-misiones-lunares-clps/

## Qué hace

- Eliges una **fecha de inicio** y una **duración** (14 días, 30 días, 90 días o 1 año).
- Eliges **hasta 6 sitios**: 8 cráteres y macizos predefinidos, o uno propio con latitud y longitud.
- Para cada sitio muestra:
  - **Con Sol:** porcentaje del periodo con el Sol sobre el horizonte.
  - **Noche más larga:** el mayor tiempo seguido sin Sol (clave para baterías y supervivencia térmica).
  - **Con Tierra:** porcentaje del periodo con la Tierra a la vista (enlace directo posible).
  - **Sin enlace más largo:** el mayor tiempo seguido sin ver la Tierra.
  - Dos líneas de tiempo (Sol y Tierra) que se recorren tocando o arrastrando.
- Marca con **"mejor"** el valor más favorable de cada métrica entre los sitios elegidos.
- Abajo dibuja el **horizonte** del sitio seleccionado (acimut vs. elevación) con el Sol y la Tierra en el momento elegido, sus trayectorias durante todo el periodo y un botón para reproducir el paso del tiempo.
- Cada sitio tiene un campo **"Horizonte local"** para subir la línea del horizonte (de 0° a 20°) y simular terreno más alto alrededor.

## Cómo se calcula

1. **Efemérides.** Con [astronomy-engine](https://github.com/cosinekitty/astronomy) se obtienen, cada hora y en UTC, las posiciones geocéntricas de la Luna y del Sol en el sistema ecuatorial J2000. Restándolas salen los vectores **Luna→Sol** y **Luna→Tierra**.
2. **Orientación de la Luna.** Esos vectores se rotan al sistema fijo a la Luna con el modelo **IAU 2009** (ascensión recta α₀ y declinación δ₀ del polo, y ángulo del meridiano W, con sus términos periódicos):

   `r_luna = Rz(W) · Rx(90° − δ₀) · Rz(90° + α₀) · r_J2000`

3. **Cielo local.** Para cada sitio (latitud φ, longitud λ) se arma una base local **arriba / este / norte** y se usa la posición del observador sobre la superficie (radio 1737,4 km). Así la dirección de la Tierra incluye el paralaje (unos 0,26°), que importa al mirarla desde la superficie.

   `elevación = asin(arriba / |t|)`  ·  `acimut = atan2(este, norte)` (0° = norte, crece hacia el este)

4. **Criterios de visibilidad.**
   - Sol: visible cuando su **borde superior** pasa el horizonte (radio aparente 0,2665°).
   - Tierra: visible cuando su **centro** pasa el horizonte.
   - Si una racha sin Sol o sin Tierra toca el inicio o el final del periodo, se muestra con "≥", porque podría ser más larga.

## Validación

- La orientación lunar del código coincide con `Astronomy.RotationAxis('Moon')` de astronomy-engine (diferencias menores a 10⁻¹⁰°).
- Según las notas del proyecto, se contrastó con PyEphem en 80 fechas entre 2026 y 2030: dirección de la Tierra dentro de 0,03° y del Sol dentro de 0,3°. No se contrastó con SPICE ni con JPL Horizons.
- Calcular un año completo (8.760 muestras horarias) toma unos 160 ms.

## Estructura del código

Todo está en un solo archivo, [`index.html`](index.html):

- **`LunarEngine`** (primer `<script>`): el motor de cálculo, separado de la interfaz. También se puede cargar en Node (`module.exports`) para probarlo.
  - `frame(d)`: orientación IAU 2009 para `d` días desde J2000.
  - `toBody(x, y, z, fr)`: rotación de J2000 al sistema fijo a la Luna.
  - `buildEphemeris(inicio, horas, paso)`: vectores Sol y Tierra para todo el periodo.
  - `siteSky(eph, lat, lon)`: acimut y elevación del Sol y la Tierra para un sitio.
- **Interfaz** (segundo `<script>`): selección de sitios, tarjetas con métricas, líneas de tiempo, gráfico SVG del horizonte y reproducción.

## Usarlo en tu computador

Descarga `index.html` y ábrelo en el navegador. Necesita internet para cargar astronomy-engine (desde jsDelivr) y las fuentes (desde Google Fonts).

## Limitaciones conocidas

- **El terreno es plano.** El horizonte es una línea recta a la altura que indiques. Cerca del polo el Sol solo sube y baja unos pocos grados, así que el relieve real lo decide todo. Con terreno plano, un cráter cuyo fondo está en sombra permanente (como Shackleton) puede aparecer con Sol el 100 % del tiempo en algunas épocas. **Los resultados no sirven para elegir un sitio de aterrizaje.**
- **Los sitios son centros de cráteres o macizos**, no sitios de aterrizaje reales, que suelen estar en bordes, crestas y mesetas.
- **El horizonte solo puede subir** (0° a 20°). No se puede representar una cima elevada, desde la que el horizonte queda por debajo de 0°.
- **Coordenadas de Shackleton:** la página usa −89,9°; el centro según el [catálogo de la USGS](https://planetarynames.wr.usgs.gov/Feature/5450) es −89,67°, 129,78° E.
- **Enlace con la Tierra:** solo exige verla sobre el horizonte; no considera elevación mínima de antena ni disponibilidad de estaciones en la Tierra.

## Próximos pasos

- Perfiles de horizonte reales por acimut, calculados con el modelo de elevación **LOLA** del orbitador LRO (el motor ya calcula el acimut de cada hora, así que bastaría con comparar `elevación > horizonte[acimut]`).
- Agregar sitios de aterrizaje reales de misiones al polo sur.
- Permitir horizontes negativos para sitios elevados.
- Contrastar con JPL Horizons.

## Créditos

- [astronomy-engine](https://github.com/cosinekitty/astronomy), de Don Cross (licencia MIT).
- Modelo de rotación: Archinal et al. (2011), *Report of the IAU Working Group on Cartographic Coordinates and Rotational Elements: 2009*, Celestial Mechanics and Dynamical Astronomy 109, 101–135.
- Fuentes tipográficas: Bricolage Grotesque, IBM Plex Mono y Public Sans (Google Fonts).
