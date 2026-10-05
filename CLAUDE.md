# Detektor

Låtsasskanner för fingeravtryck, byggd som installerbar PWA på samma sätt som `Kalkylator`.
Ett stort fingeravtryck mitt på skärmen och en bred knapp, Skanna. Knappen är i hemlighet tre
knappar: var på bredden man trycker avgör vilket svar som visas när skanningen är klar.

Ännu inget git-repo och inte publicerad. Tänkt att läggas på GitHub Pages som `Kalkylator`.

## Stack

Ren HTML/CSS/JavaScript utan ramverk, byggsteg eller `package.json`. Det ska förbli så.

| Fil | Innehåll |
|---|---|
| `index.html` | Hela appen: CSS, markup och skript i en fil. |
| `sw.js` | Service worker: cache-first för alla GET, faller tillbaka på `index.html` offline. |
| `manifest.webmanifest` | PWA-manifest (standalone, porträtt). |
| `*.png` | Appikoner. |

## Köra lokalt

Service workern kräver http, så öppna inte `index.html` direkt från disk:

```
npx serve .
```

Service workern cachar hårt. Syns inte en ändring: avregistrera den i DevTools
(Application → Service workers) eller testa i ett privat fönster.

## Ny version

Varje ändring som ska nå installerade appar kräver båda stegen, med samma nummer:

1. Höj `CACHE` i `sw.js` (`detektor-vN`).
2. Höj `APP_VERSION` i `index.html` (`'N · ÅÅÅÅ-MM-DD'`). Visas under Version i dolda läget.

Nya filer som ska fungera offline läggs också till i `ASSETS` i `sw.js`.

## Så fungerar appen

- Skriptet är en IIFE i ES5-stil (`var`, `function`, inga pilfunktioner). Håll den stilen i
  `index.html`. `sw.js` använder modern syntax.
- Storlekar anges i `cqw` mot `.phone` (container query), så layouten skalar med bredden.
- Tillståndet ligger i `data-state` på `.phone`: `idle`, `scanning` eller `result`. CSS styr
  laser, glöd och resultatfärg (`--res`) utifrån det.
- Fingeravtrycket ritas av `buildPrint` som nästlade valv beskurna av en oval. Det finns i två
  exemplar: `#fpBase` (grått, färgas med resultatfärgen) och `#fpHot` (rött, syns bara i ett
  band kring lasern via `clip-path`).
- `frame` driver skanningen med `requestAnimationFrame`: laserns läge, det röda bandet och
  procenträknaren. `SWEEP_MS` är tiden för en tur upp och ner.
- Knappen lyssnar på `pointerdown`, inte `click`, så att den fungerar även när ett annat finger
  vilar på fingeravtrycket. `zoneAt` delar bredden i tre lika delar: `left`, `mid`, `right`.
- På dator startar tangenterna 1, 2 och 3 en skanning för respektive zon.

### Dolt läge

- Långtryck (650 ms) på rubriken högst upp öppnar inställningsbladet. Det går inte att öppna
  under en pågående skanning.
- Där väljs svarstext och färg (`COLORS`) för var och en av de tre zonerna samt skanningstid
  i sekunder (1–15).
- Inställningarna ligger i `cfg` och sparas i `localStorage` under `detektor_cfg`.

Inget i det synliga gränssnittet får avslöja att knappen har tre zoner: samma tryckeffekt
oavsett var man trycker.

## Test

Det finns inga automatiska tester. Testa för hand i webbläsaren, helst även på telefon som
installerad app, eftersom långtryck, vibration och safe-area bara märks där.
