# ADR-003 — Integratiemethode: hoe koppelt de module aan OpenMRS?

## Status

Superseded (ADR_Q)

## Context

De communicatiemodule moet afspraakdata ophalen uit één of meerdere OpenMRS-instanties om tijdig notificaties te versturen (24 uur en 1 uur vóór de afspraak). OpenMRS biedt meerdere mechanismen om data te ontsluiten. De keuze voor een integratiemethode heeft directe gevolgen voor:

- de installatielast bij OpenMRS-beheerders van aangesloten organisaties;
- het gedrag van het systeem bij downtime van de communicatiemodule of van OpenMRS;
- de schaalbaarheid naar meerdere OpenMRS-instanties wereldwijd;
- de aansluiting op de HL7/FHIR-standaard die de opdracht voorschrijft.

De volgende drie opties zijn onderzocht:

**Optie A — FHIR REST polling**
De communicatiemodule bevraagt periodiek de FHIR2 REST API van OpenMRS:
`GET /ws/fhir2/R4/Appointment?date=ge<datum>&status=booked`
De module slaat de timestamp van de laatste succesvolle poll op en hervat altijd vanaf dat punt.

**Optie B — Atom Feed consumeren**
OpenMRS publiceert elke aanmaaak, wijziging of verwijdering van objecten (inclusief afspraken) als entry in een Atom Feed. De communicatiemodule pollt de feed en verwerkt nieuwe entries event-driven. Vereist installatie van de Atom Feed module én de Event module op elke OpenMRS-instantie. Afspraken worden niet altijd standaard gepubliceerd en kunnen extra configuratie vereisen.

**Optie C — Webhook / push vanuit OpenMRS**
OpenMRS stuurt een HTTP POST naar de communicatiemodule zodra een afspraak aangemaakt of gewijzigd wordt. OpenMRS heeft geen standaard webhook-mechanisme; dit vereist een zelf ontwikkelde OpenMRS-module per instantie. Bij downtime van de communicatiemodule genereert OpenMRS fouten aan de kant van de zorginstelling.

## Decision

Gekozen wordt voor **Optie A: FHIR REST polling**.

De communicatiemodule bevraagt de FHIR2 API van elke geconfigureerde OpenMRS-instantie op regelmatige intervallen. Per organisatie wordt een base URL, authenticatie en een polling-interval opgeslagen in de beveiligde configuratie van de module. De timestamp van de laatste succesvolle poll wordt persistent bijgehouden zodat bij herstart altijd vanaf het juiste punt hervat wordt en geen afspraken gemist worden. Bij het ophalen worden uitsluitend toekomstige afspraken met status `booked` verwerkt; reeds aangevangen afspraken worden gefilterd conform de functionele eis.

De argumentatie ten opzichte van de alternatieven:

- Optie A vereist geen extra modules op OpenMRS-kant. Elke OpenMRS-instantie vanaf versie 2.7.x met de FHIR2 module (standaard aanwezig in O3) is direct bruikbaar. Dit verlaagt de installatiedrempel voor organisaties wereldwijd aanzienlijk.
- Optie B is event-driven en efficiënter, maar vereist de Atom Feed module en Event module als extra afhankelijkheden. Dit vergroot de installatielast en introduceert aanvullende configuratierisico's, met name voor afspraken die niet standaard in de feed verschijnen.
- Optie C biedt realtime push maar vereist een zelf ontwikkelde OpenMRS-module per instantie en koppelt OpenMRS strak aan de communicatiemodule. Dit staat haaks op de eis dat de communicatiemodule zelfstandig moet kunnen functioneren en los van andere systemen moet draaien.
- Optie A sluit het sterkst aan op de FHIR/HL7-eis uit de opdracht doordat de FHIR Appointment resource centraal staat als gegevensbron.
- Meerdere OpenMRS-instanties worden ondersteund door per organisatie een eigen polling-configuratie bij te houden; elke instantie is een zelfstandige entry met eigen base URL en credentials.

## Consequences

**Wordt eenvoudiger:**

- Onboarding van nieuwe OpenMRS-organisaties: alleen base URL en authenticatiegegevens hoeven geconfigureerd te worden, geen extra OpenMRS-modules vereist.
- Foutherstel bij downtime: de module hervat polling vanaf de opgeslagen timestamp; er gaan geen events verloren.
- Schaalbaarheid naar meerdere instanties: elke instantie is een onafhankelijke polling-configuratie.
- Naleving van de FHIR-standaard: de integratie verloopt volledig via gestandaardiseerde FHIR R4 resources en endpoints.
- Testbaarheid: de FHIR API is goed gedocumenteerd en eenvoudig te mocken in integratietests.

**Wordt moeilijker of vraagt extra aandacht:**

- Realtime reageren op annuleringen: omdat de module pollt in plaats van pushed events ontvangt, bestaat een kleine vertraging tussen annulering in OpenMRS en verwerking in de communicatiemodule. Dit moet worden opgelost door het polling-interval kort genoeg te kiezen en bij verwerking altijd de actuele appointmentstatus te controleren via een aanvullende FHIR-aanroep vóór het versturen van de notificatie.
- Load op de OpenMRS FHIR API: bij veel aangesloten organisaties of korte polling-intervallen kunnen de API-aanroepen merkbare belasting veroorzaken. Dit vraagt om verstandige keuze van het polling-interval (aanbevolen: vijf tot tien minuten) en eventueel rate limiting aan de kant van de communicatiemodule.
- Geen garantie op wijzigingsmeldingen buiten het polling-venster: als een afspraak aangemaakt én direct daarna geannuleerd wordt vóór de eerstvolgende poll, wordt de annulering correct meegenomen omdat de module de actuele status ophaalt, niet een snapshot.
