# PDF-overlay – Trimble Connect-tillägg

Kalibrera en PDF-sida (plan, sektion, fasad) mot 3D-modellen med
referenspunkter, justera läge/rotation/skala/transparens, och lås
placeringen som en riktig, texturerad modell i projektet.

## Arkitektur i korthet

Trimble Connects Workspace API kan **inte** rendera egna texturer/bilder
direkt i 3D-vyn (kontrollerat mot dokumentationen — se motivering nedan).
Därför är detta tillägg byggt i två steg:

1. **Kalibrering & förhandsvisning** (snabbt, allt lokalt i denna panel):
   välj PDF, välj sida, klicka 2–3 referenspunkter (matchande punkt i PDF:en
   + motsvarande punkt i 3D-vyn), justera fritt. En **guide** (fyra
   hörnpunkter + diagonaler, ritade som linjer) visas i 3D-vyn så du kan
   verifiera läge/rotation/skala — det är inte den riktiga PDF-bilden, bara
   en ram som visar var den kommer hamna.
2. **Lås**: genererar en textbaserad IFC-fil med PDF-sidan inbränd som
   textur i rätt läge, och försöker ladda upp den till projektet. Trimble
   konverterar IFC-filer automatiskt till visningsbar geometri, precis som
   för vanliga modellfiler — resultatet blir en äkta texturerad yta, inte en
   overlay-illusion.

**Varför inte rendera texturen direkt?** Jag undersökte `viewer.addTrimBimModel`
(Trimbles enda geometri-med-textur-kanal i Workspace API) men TrimBIM (.trb)
är ett stängt, proprietärt binärformat utan publik specifikation — det går
inte att skapa en sådan fil i webbläsaren. IFC är däremot ett öppet
textformat som går att generera i ren JavaScript, inklusive texturmappning,
så det är vägen vi går istället.

## Så här används det

1. Klicka **"+ Ny PDF-overlay"**.
2. **Steg 1:** välj en PDF från Trimble Connects projektfiler, eller ladda
   upp en lokal fil.
3. **Steg 2:** välj sida via miniatyrerna.
4. **Steg 3:** välj 2 eller 3 referenspunkter:
   - **2 punkter** — antar ett plant, horisontellt plan (för planritningar).
     Ange nivå (Z-höjd) manuellt.
   - **3 punkter** — löser ett fritt plan i 3D, inklusive lutning (för
     sektioner/fasader).
   För varje punkt: klicka först på PDF-sidan, klicka sedan på motsvarande
   punkt i den riktiga 3D-vyn.
5. **Steg 4:** justera extra offset, rotation, skala och transparens.
   Klicka "Uppdatera guide i 3D-vyn" för att se ramen flytta sig.
6. **Steg 5:** klicka **"Lås & generera"**. Ladda gärna ner IFC- och
   kalibreringsfilerna direkt för att testa manuellt, oavsett om
   auto-uppladdningen lyckas.

## Installation

Samma mönster som de andra tilläggen:
1. Ladda upp `index.html` och `manifest.json` till en egen hosting/repo.
2. Uppdatera `"url"`/`"icon"` i `manifest.json`.
3. **Project Settings → Apps & Capabilities → Add Custom**, klistra in
   raw-länken till `manifest.json`.

## Kända begränsningar & experimentella delar

Detta är det mest osäkra tillägget hittills eftersom det rör sig utanför
Workspace API:ts dokumenterade yta. Testa metodiskt och skicka gärna
"🐞 Rådata"-loggen när något inte fungerar, så justerar vi tillsammans —
precis som med tidigare tillägg.

- **Filuppladdning/-listning mot Trimble Connect (`listProjectPdfFiles`,
  `downloadTcFile`, `uploadTcFile`) är en kvalificerad gissning.** Den
  exakta REST-strukturen för projektfiler är inte del av den publika
  Workspace-API-dokumentationen. Koden gissar en URL baserat på
  `document.referrer` (iframens värdadress) och Trimbles kända
  `/tc/api/2.0/...`-mönster. Fungerar det inte, ladda upp lokalt istället
  (fungerar garanterat) och skicka felmeddelandet från 🐞 Rådata så justerar
  vi endpointen.
- **IFC-texturering (`IfcImageTexture` med en `data:`-URI) är obeprövad.**
  Vissa IFC-importerare stödjer bara texturer via externa URL:er, inte
  inbäddade data-URI:er. Om geometrin (den platta ytan, rätt placerad)
  dyker upp men utan bild, är det troligen orsaken — nästa steg blir då att
  ladda upp bilden separat och referera dess riktiga nedladdnings-URL
  istället.
- **Guide i 3D-vyn ≠ den riktiga PDF-bilden.** Fram tills du låser visas
  bara en ram (linjer), inte pixlarna. Det är en medveten avvägning eftersom
  plattformen inte tillåter livetexturer.
- **Referenspunkter kräver klick på synlig geometri/punktmoln** i 3D-vyn,
  precis som i extrude-verktyget — samma `viewer.onPicked`-begränsning.
- Session-only för guiden (linjerna); den låsta IFC-modellen är dock en
  riktig, permanent projektfil när uppladdningen lyckas.
- Ingen automatisk "återställ overlays vid projektöppning" är inbyggd ännu
  i denna första version — kalibreringen sparas som JSON (lokalt
  nedladdningsbar + försök till uppladdning), men inläsning/skanning av
  befintliga `.calibration.json`-filer vid uppstart är inte implementerat.
  Säg till om du vill att vi bygger det som nästa steg, när
  filuppladdningen är verifierad och fungerar.
