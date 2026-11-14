# CLPS Lunar Mission Navigator

An interactive web page that compares, for sites at the Moon's south pole, how long each one sees the **Sun** and the **Earth** above its horizon. It is a way to explore why lighting (solar power, temperature) and a direct view of Earth (communications) are two of the factors that matter most when choosing where to land at the south pole, as in the commercial missions of NASA's CLPS program.

**Try it live:** https://xscarletx.github.io/clps-lunar-mission-navigator/

## What it does

- Pick a **start date** and a **duration** (14 days, 30 days, 90 days or 1 year).
- Pick **up to 6 sites**: 8 preset craters and massifs, or your own by latitude and longitude.
- For each site it shows:
  - **Sunlit:** percentage of the period with the Sun above the horizon.
  - **Longest night:** the longest continuous stretch without Sun (key for batteries and thermal survival).
  - **Earth in view:** percentage of the period with Earth in view (direct link possible).
  - **Longest link gap:** the longest continuous stretch without seeing Earth.
  - Two timelines (Sun and Earth) you can tap or drag through.
- Marks the most favorable value of each metric among the selected sites as **"best"**.
- Below, it draws the **horizon** of the selected site (azimuth vs. elevation), with the Sun and Earth at the chosen moment, their paths over the whole period, and a button to play time forward.
- Each site has a **"Local horizon"** field to raise the horizon line (0° to 20°) and simulate higher terrain around it.

## How it is computed

1. **Ephemeris.** [astronomy-engine](https://github.com/cosinekitty/astronomy) gives, every hour and in UTC, the geocentric positions of the Moon and the Sun in the J2000 equatorial frame. Subtracting them gives the **Moon→Sun** and **Moon→Earth** vectors.
2. **Lunar orientation.** Those vectors are rotated into the Moon-fixed frame with the **IAU 2009** model (right ascension α₀ and declination δ₀ of the pole, and the prime meridian angle W, with their periodic terms):

   `r_moon = Rz(W) · Rx(90° − δ₀) · Rz(90° + α₀) · r_J2000`

3. **Local sky.** For each site (latitude φ, longitude λ) it builds a local **up / east / north** basis and uses the observer's position on the surface (radius 1737.4 km). This way Earth's direction includes parallax (about 0.26°), which matters when looking at it from the surface.

   `elevation = asin(up / |t|)`  ·  `azimuth = atan2(east, north)` (0° = north, increasing toward the east)

4. **Visibility criteria.**
   - Sun: visible when its **upper limb** clears the horizon (apparent radius 0.2665°).
   - Earth: visible when its **center** clears the horizon.
   - If a stretch without Sun or without Earth touches the start or end of the period, it is shown with "≥", because it could be longer.

## Validation

- The code's lunar orientation matches astronomy-engine's `Astronomy.RotationAxis('Moon')` (differences below 10⁻¹⁰°).
- **Checked against NASA's [JPL Horizons System](https://ssd.jpl.nasa.gov/horizons/)** (observer on the lunar surface, airless apparent azimuth and elevation): at Malapert and Shackleton, on 31 daily dates from November 14 to December 14, 2026, the Sun's and Earth's azimuth and elevation agree with Horizons within 0.01°.
- According to the project notes, it was also checked against PyEphem on 80 dates between 2026 and 2030: Earth's direction within 0.03° and the Sun's within 0.3°. It has not been checked against SPICE directly.
- Computing a full year (8,760 hourly samples) takes about 160 ms.

## Code structure

Everything lives in a single file, [`index.html`](index.html):

- **`LunarEngine`** (first `<script>`): the computation engine, separate from the interface. It can also be loaded in Node (`module.exports`) for testing.
  - `frame(d)`: IAU 2009 orientation for `d` days since J2000.
  - `toBody(x, y, z, fr)`: rotation from J2000 into the Moon-fixed frame.
  - `buildEphemeris(start, hours, step)`: Sun and Earth vectors for the whole period.
  - `siteSky(eph, lat, lon)`: azimuth and elevation of the Sun and Earth for one site.
- **Interface** (second `<script>`): site selection, metric cards, timelines, the SVG horizon chart and playback.

## Run it on your computer

Download `index.html` and open it in a browser. It needs an internet connection to load astronomy-engine (from jsDelivr) and the fonts (from Google Fonts).

## Known limitations

- **The terrain is flat.** The horizon is a straight line at the height you set. Near the pole the Sun only rises and falls by a few degrees, so real terrain decides everything. With flat terrain, a crater whose floor is in permanent shadow (such as Shackleton) can show up as sunlit 100% of the time in some seasons. **The results are not meant for choosing a landing site.**
- **The sites are crater or massif centers**, not real landing sites, which are usually on rims, ridges and plateaus.
- **The horizon can only go up** (0° to 20°). It cannot represent a high peak, from which the horizon sits below 0°.
- **Shackleton coordinates:** the page uses −89.9°; the center according to the [USGS gazetteer](https://planetarynames.wr.usgs.gov/Feature/5450) is −89.67°, 129.78° E.
- **Earth link:** it only requires Earth to be above the horizon; it ignores minimum antenna elevation and ground-station availability.

## Next steps

- Real horizon profiles by azimuth, computed from the **LOLA** elevation model of the LRO orbiter (the engine already computes the azimuth for every hour, so the check would simply become `elevation > horizon[azimuth]`).
- Add real landing sites of south pole missions.
- Allow negative horizons for elevated sites.

## AI assistance

In line with the NASA Space Apps rules on Super Intelligence (SI) use, this project acknowledges the AI tools it used:

- **Code:** AI (Claude, by Anthropic) assisted with writing, debugging and reviewing the web app (HTML, CSS and JavaScript).
- **Documentation and translation:** this README and the English version of the interface were written and translated with AI assistance.
- **Demo video:** the animations were generated with AI-written code, and the narration is an AI-generated voice ([Kokoro TTS](https://huggingface.co/hexgrad/Kokoro-82M)). The video carries a visible "AI-GENERATED" label.
- **Metadata:** commits made with AI assistance include a `Co-Authored-By: Claude` line, and `index.html` carries an `ai-assistance` meta tag.

## Credits

- [astronomy-engine](https://github.com/cosinekitty/astronomy), by Don Cross (MIT license).
- Validation data: NASA's [JPL Horizons System](https://ssd.jpl.nasa.gov/horizons/).
- Rotation model: Archinal et al. (2011), *Report of the IAU Working Group on Cartographic Coordinates and Rotational Elements: 2009*, Celestial Mechanics and Dynamical Astronomy 109, 101–135.
- Typefaces: Bricolage Grotesque, IBM Plex Mono and Public Sans (Google Fonts).
