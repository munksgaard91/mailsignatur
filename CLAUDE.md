# Mailsignatur (Best One A/S)

Internt værktøj, hvor medarbejdere hos Best One A/S laver deres Gmail-signatur. Man udfylder et par felter, ser signaturen live og kopierer den direkte ind i Gmail.

Værktøjet erstatter den gamle arbejdsgang, hvor man copy-pastede HTML-koden ind i W3Schools' HTML-editor, rettede sine oplysninger direkte i koden og kopierede resultatet over i Gmail.

Ejer: Rasmus Munksgaard (RMU), web-afdelingen.

## Hosting

* GitHub Pages fra repoet `munksgaard91/mailsignatur` (main, roden).
* Domæne: `mailsignatur.leksione.dk` via `CNAME`-filen i roden.
* DNS ligger hos Curanet: CNAME-record `mailsignatur` peger på `munksgaard91.github.io`.
* Enforce HTTPS har ikke været tilgængelig på leksione.dk-subdomænerne. DNS skal ikke flyttes til Cloudflare.
* Siden har `noindex, nofollow`, da den ligger på et offentligt domæne.

## Filer

* `index.html`: hele værktøjet i én fil (HTML, CSS og JS). Ingen build, ingen afhængigheder ud over Google Fonts (Montserrat).
* `CNAME`: domænet til GitHub Pages.
* `CLAUDE.md`: denne fil.

## Sådan virker det

Felter i denne rækkefølge:

1. **Initialer**, fx `rmu`. Renses for store bogstaver, mellemrum og specialtegn.
2. **Navn**
3. **Titel** (placeholder: "Konsulent")
4. **Mail**: udfyldes automatisk som `initialer@best-one.dk`. Følger initialerne, så længe feltet er tomt eller stadig matcher de forrige initialer. Retter brugeren selv mailen, røres den ikke.
5. **Mobil** (valgfri): accepterer 8 cifre, også med +45 og mellemrum. Vises som `25 94 00 00` og linker som `tel:+4525940000`. Tomt felt fjerner hele mobilrækken.
6. **Feriesignatur**: toggle med Fra og Til og med. Indsætter en grøn OBS-boks med datoerne, fx "fra d. 22. december til og med d. 30. december".

Øvrig adfærd:

* **Profilbillede** findes ud fra initialerne: `https://best-one.dk/mailsignatur/<initialer>.jpg`. Siden tester med `new Image()`, om billedet findes. Findes det ikke, bruges `bo.jpg` (Best One logoet), og der vises en lille note under initialfeltet.
* **Kopiér signatur** kopierer signaturen som formateret HTML via `ClipboardItem` (text/html og text/plain). Fallback: markerer previewet og bruger `execCommand('copy')`. Knappen venter, hvis profilbilledet stadig hentes.
* **Validering**: mail skal ende på `@best-one.dk`, mobil skal være 8 cifre, ferie skal have begge datoer i rigtig rækkefølge. Fejl vises som røde advarsler under feltet, og kopiering blokeres.
* **localStorage** (`bo-mailsignatur`) husker felterne i browseren.
* **Udløbet ferie**: er Til og med passeret, slås feriesignaturen fra ved næste besøg.
* **Ryd alle felter** nulstiller alt og sletter det gemte.

## Signaturens HTML

Bygges i `buildSignature()`. Regler, der skal overholdes, fordi det er mailklienter (Gmail, Outlook, mobil), der skal vise den:

* Kun tabeller og inline styles. Ingen klasser, ingen `<style>`, ingen `!important`.
* `role="presentation"` på layout-tabeller.
* Font: `Montserrat,Verdana,Arial,sans-serif`. Montserrat findes sjældent hos modtageren.
* Alle billeder har `width` og `height` som attributter og i style. Outlook bruger attributterne.
* Alle links har farve inline, ellers farver Gmail dem blå.
* Ferieboksen bruger både `bgcolor` og `background-color`, så den også virker i Outlook.
* Outputtet minificeres (whitespace mellem tags fjernes). Signaturen fylder ca. 6.200 tegn, Gmails grænse er 10.000.
* Brugerinput escapes altid med `esc()`.

Faste oplysninger ligger som konstanter øverst i scriptet: `ASSETS`, `PHOTO_EXT`, `FALLBACK_PHOTO`, `SITE`, `MAIN_PHONE`, `ADDRESS`. Ret dem ét sted.

Farver: blå `#225581` (navn og best-one.dk), grå `#888888` (tekst), grøn `#43C890` (ferieboks), streg `#eaeaea`.

## Billeder på FTP

Alle ligger i `https://best-one.dk/mailsignatur/` og er i 2x opløsning, så de er skarpe på retina-skærme. Koden låser visningsstørrelsen.

| Fil | Filstørrelse | Vises i |
|---|---|---|
| `partners-logo.png` | 1000 × 143 | 500 × 72 |
| `best-logo.png` | 500 × 87 | 180 × 31 |
| `mail-icon.png`, `phone-icon.png`, `mobile-icon.png`, `pin-icon.png`, `link-icon.png` | 60 × 60 | 20 × 20 |
| `bo.jpg` og `<initialer>.jpg` | 750 × 750 (min. 270 × 270) | 135 × 135 |

Vigtigt:

* Visningsstørrelsen skal have samme proportioner som filen. Partnerlogoet blev før vist i 500 × 60 og så mast og sløret ud.
* Gmail cacher billeder i signaturer via sin proxy (googleusercontent.com). Udskiftes en fil med samme navn, kan den gamle version blive hængende. Brug et nyt filnavn, ret koden, og få folk til at indsætte signaturen igen.
* Slet ikke gamle filer på FTP med det samme. Signaturer, der allerede er sat ind i Gmail, peger stadig på dem.
* `border-radius` på profilbilledet virker ikke i Outlook til Windows. Runde PNG'er med gennemsigtig baggrund ville løse det.

## Bevidste fravalg

Ting, der er fjernet efter ønske. Tilføj dem ikke igen uden at spørge:

* Ingen topbar/header. Siden starter direkte med formularen.
* Ingen "Venlig hilsen" i previewet.
* Ingen "Kopiér HTML-kode"-knap. Kun én knap: Kopiér signatur.
* Ingen tegntæller.
* Ingen "Kopiér udfyldt link" og ingen URL-parametre.
* Ingen hjælpetekster under felterne (kun note om manglende billede og røde advarsler).
* Ingen skærmlæser-attributter (`aria-describedby`, `aria-live`, `aria-label`) i værktøjets UI.

## Stil og tekst

* Best One-branding: Montserrat og `#225581`. Nyere interne værktøjer skal matche stilen på kvalitetstjek.leksione.dk.
* Favicon er et hvidt "b" på blå cirkel, lagt ind som SVG data-URI i `<head>`.
* Al tekst i UI og signatur er på dansk.
* Brug ikke tankestreger (– eller —) i tekster.

## Test

Der er ingen testsuite i repoet. Ændringer er testet med jsdom, hvor `window.Image` mockes til at lykkes eller fejle. Tjek mindst:

* Initialer finder `<initialer>.jpg`, ukendte initialer falder tilbage til `bo.jpg`.
* Mail følger initialerne, men ikke efter manuel rettelse.
* Mobil med +45 formateres og linker korrekt.
* Ferietekst, datovalidering og at telefonnummeret i ferieboksen er et `tel:`-link.
* HTML i navnefeltet bliver escaped.
* Udløbet ferie slås fra ved indlæsning, og Ryd alle felter virker.

Test altid det endelige resultat ved at indsætte signaturen i Gmail og sende en testmail.

## Åbne punkter

* Skal `info@best-one.dk` i ferieboksen også være et klikbart `mailto:`-link ligesom telefonnummeret?
* SVG-favicon vises ikke i alle Safari-versioner. Overvej en PNG-fallback.
