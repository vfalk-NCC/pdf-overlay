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

## ⚡ Visa ritningen i 3D (linjer)

Trimble Connect kan inte visa en PDF-bild i 3D-vyn: Workspace API har
ingen texturkanal, och IFC-texturer verkar inte följa med i Trimbles
konvertering. En PDF utskriven från CAD består däremot av riktiga linjer.
**⚡ Visa linjer** i Steg 4 plockar ut dem och ritar dem direkt i 3D-vyn
med `markup.addLineMarkups`, genom samma kalibrering som guiden (dina
referenspunkter).

- **Justering:** linjerna och guiden följer Steg 4-reglagen live när
  "Uppdatera live" är ikryssat.
- **Detalj:** dubbletter tas bort och raka följdsegment slås ihop. Vid
  fler linjer än taket (Låg 5 000, Normal 15 000, Hög 40 000) visas de
  längsta.
- **Spara:** när du låser följer linjerna med overlayen, och korten får
  knapparna "⚡ Linjer" och "Dölj linjer".
- **Begränsningar:** kräver en vektor-PDF (inte skannad). Fyllningar och
  riktig PDF-text följer inte med. Linjerna försvinner när vyn laddas om.
- **Permanent:** **⬇ Ladda ner DXF** / **☁ Ladda upp DXF** i Steg 5 ger
  samma linjer som DXF i världskoordinater (meter). Trimble Connect visar
  DXF i 3D.

Linjeutdragningen är skriven för pdf.js 3.11.174 (den version som laddas),
som packar vägdata som `[OPS-koder, koordinater, minMax]`. Bara vägar som
faktiskt ritas (stroke/fill) tas med, inte osynliga klippramar.

## 📐 Vrid in DWG/modell

Flyttar och vrider en inläst modell (DWG, DXF eller IFC) direkt i vyn med
`viewer.placeModel`.

1. Öppna DWG:n i 3D-vyn och klicka **📐 Vrid in DWG/modell**. Välj
   modellen (ritningar listas först).
2. Klicka två punktpar: först en punkt **på DWG:n**, till exempel ett
   axelkryss, sedan **samma punkt i modellen**. Välj punkter långt ifrån
   varandra.
3. Appen räknar ut vridning och förflyttning, så att punkternas mittpunkt
   sammanfaller, och flyttar DWG:n direkt. Den visar också kvarvarande
   avvikelse per par.
   - **Tillåt skalning:** för DWG:er i fel enhet, till exempel mm i stället
     för m. Appen varnar när avstånden mellan punkterna inte stämmer.
   - **Flytta till modellens höjd:** lyfter DWG:n till målpunkternas höjd.
4. **Finjustera och rotera (Steg 3).** Fungerar även utan punktparen.
   Allt läggs på direkt.
   - **Förskjutning** X/Y/Z i meter.
   - **Rotation X / Y / Z** (lutning framåt och bakåt, åt sidan, vridning
     i planet): reglage (±180°) och sifferfält för exakta grader. Vridpunkt
     är punkt 1 från paren, annars modellens origo. Med **📍 Välj
     vridpunkt i 3D** väljer du en egen.
   - **Rotera runt en egen linje:** **✏️ Rita linje** och klicka två
     punkter i 3D, till exempel längs en väggs fot. Linjen ritas ut i
     orange, och reglaget (±180°, snabbknappar −90/0/+90/180) fäller
     DWG:n runt den. Bra för att resa en sektion eller fasad.
   - **Inbakning:** när du väljer en ny vridpunkt eller ritar en ny linje
     bakas nuvarande läge in som ny bas och vinklarna nollställs. Du
     bygger alltså vidare från där DWG:n står.
   - **Positionsinställningar:** värdena visar position, rotation X/Y/Z
     (R = Rz·Ry·Rx) och skala.

**Inpassningen följer med:** `placeModel` gäller bara den aktuella
visningen. Släcker och tänder man modellen laddar Trimble om den med sin
egen placering. Därför:
- **Automatisk påläggning:** varje gång en ritning läses in, och när
  tillägget öppnas, lägger tillägget automatiskt på den sparade
  inpassningen. Det gäller även nya revideringar, eftersom fil-id:t är
  detsamma. Tillägget måste vara öppet.
- **Delad:** inpassningen sparas i `projects/<projekt-id>/dwg_placements.json`
  i 4D-data, så kollegor får ritningen på rätt plats. Den skrivs först när
  man slutat justera i 3 sekunder, eftersom GitHub begränsar antalet
  skrivningar. Utan token sparas den bara i webbläsaren.
För att spara permanent för alla i projektet: för över de visade värdena
(position, rotation kring Z, skala) till Trimble Connects
**Positionsinställningar**. **↺ Återställ originalplacering** ångrar.

Antagande att verifiera i Trimble: `ModelPlacement` tolkas som
*värld = position + skala · R · lokal*, med `position` i mm och R från
`refDirection`/`axis`. Hamnar DWG:n fel trots att avvikelsen visas som
0 mm, skicka 🐞 Rådata-loggen (loggar `placeModel` och modellistan).

## 🏷️ Ritningsregister

Trimble Connect saknar ett fält för egna namn eller kommentarer på
ritningar. **🏷️ Ritningsregister** listar projektets modeller (DWG/DXF
som standard) med ett fritextfält efter ritningsnumret, till exempel
"Sektion A-A vägg 3, gäller DP2".

- **Revideringar:** texten kopplas till filen i Trimble Connect, inte till
  en version, och ligger kvar när ritningen revideras. Filen i sig ändras
  inte.
- **Var namnet syns:** i "📐 Vrid in DWG/modell" som
  "5082648 – Sektion A-A vägg 3". Med **Visa namnen i 3D** visas det som
  en skarp textetikett över varje inläst ritning.
- **Sök och visa:** sök på nummer eller text. "Visa i 3D" läser in
  ritningen, och "Zooma" flyttar kameran till en inläst ritning.
- **Delning:** namnen sparas i `projects/<projekt-id>/drawing_labels.json`
  i det privata repot `vfalk-NCC/4D-data`, via `github-storage.js`
  (samma som 4D-planering). Alla i projektet ser dem. Tokenen från
  4D-planering eller anteckningsappen används automatiskt eftersom
  tilläggen ligger på samma adress. Utan token sparas namnen bara i
  webbläsaren.
- **Fixade ritningar syns alltid:** ritningar du har namngett eller passat
  in listas överst, även när Trimble inte listar dem. Trimble ser bara
  filer i mappar som öppnats i Modeller-panelen. Märkningen visar
  🏷️ (text), 📐 inpassad och "ej i Trimbles lista". **Bara mina fixade**
  filtrerar fram dem.
- **Inläsning:** "Visa i 3D" läser in ritningen och lägger på den sparade
  inpassningen direkt. Hittar Trimble inte filen, öppna dess mapp i
  Modeller-panelen. "✕" glömmer en sparad inpassning.
- **Lagring:** inpassningarna sparas i webbläsaren, namnen delat.

`github-storage.js` är en identisk kopia av filen i 4D-planering.

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

- **Trimble Connect-filer:** "Välj från Trimble Connect" och
  uppladdningarna använder Trimbles Core API 2.0 på samma sätt som
  ritningssnabbtitten, där det är bekräftat att fungera:
  - Åtkomst via `extension.requestPermission("accesstoken")`. Token kommer
    som eventet `extension.accessToken`.
  - Regional server utifrån `project.location` (EU = `app21`).
  - Filträdet läses från projektets rotmapp.
  - Uppladdningar hamnar i mappen "PDF-overlay".
  - Fältet "Trimble API-värd" skriver över servern vid behov.
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
