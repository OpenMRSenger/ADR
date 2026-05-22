# ADR-006: Outbox-patroon in de omod versus direct versturen

## Status

Accepted

## Context

Wanneer er een afspraak wordt aangemaakt of gewijzigd in OpenMRS, moet de omod (de aangepaste OpenMRS-module) een notificatie versturen via de SaaS-provider. De vraag is hoe betrouwbaar dat moet gebeuren.

Twee opties zijn onderzocht:

**Optie A - Direct versturen**
De omod roept direct de SaaS-provider aan zodra een afspraakevent plaatsvindt. Als de aanroep slaagt, is het bericht verstuurd. Als het mislukt, is het bericht kwijt.

**Optie B - Outbox-patroon binnen de omod**
De omod schrijft het te versturen bericht naar een outbox-tabel in de OpenMRS-database. Een achtergrondproces binnen de omod leest berichten uit die tabel en verstuurt ze via de SaaS-provider. Na een succesvolle verzending wordt het bericht als verstuurd gemarkeerd. Bij een fout wordt het later opnieuw geprobeerd.

## Decision

Er is gekozen voor **Optie B: het outbox-patroon binnen de omod**.

Wanneer een afspraakevent plaatsvindt in OpenMRS, slaat de omod het bericht op in een outbox-tabel die door de omod wordt beheerd in de OpenMRS-database. Een achtergrondtaak binnen de omod controleert de tabel periodiek en verstuurt openstaande berichten naar de SaaS-provider. Mislukte berichten worden na een wachttijd automatisch opnieuw geprobeerd. Succesvolle berichten worden gemarkeerd als verstuurd.

Redenen voor deze keuze:

- **Betrouwbaarheid**: Als de SaaS-provider tijdelijk niet bereikbaar is, gaat het bericht niet verloren. Het blijft in de outbox staan totdat het succesvol is verstuurd.
- **Consistentie**: Het opslaan van het bericht en het verwerken van het afspraakevent gebeuren in dezelfde database-transactie binnen OpenMRS. Er is geen situatie waarbij een event is verwerkt maar het bericht nooit wordt verstuurd.
- **Zichtbaarheid**: De outbox-tabel geeft inzicht in welke berichten verstuurd zijn, welke nog wachten en welke zijn mislukt. Dat maakt foutopsporing eenvoudiger.
- **Retry-logica**: Mislukte berichten worden automatisch opnieuw geprobeerd zonder handmatige tussenkomst.

## Consequences

**Wordt eenvoudiger:**

- Betrouwbaarheid van notificaties: berichten gaan niet verloren bij een tijdelijke storing bij de SaaS-provider.
- Foutopsporing: de outbox-tabel in OpenMRS laat precies zien wat er met elk bericht is gebeurd.
- Ontkoppeling van event en verzending: de afspraakverwerking in OpenMRS hoeft niet te wachten op de SaaS-provider.

**Vraagt extra aandacht:**

- Meer complexiteit in de omod: de omod beheert nu een eigen tabel en een achtergrondtaak. Dit is meer werk dan een directe aanroep.
- Lichte vertraging: berichten worden verstuurd zodra de achtergrondtaak ze oppakt, niet direct op het moment van het event. In de praktijk is dit verwaarloosbaar.
- Dubbele verzending: als de achtergrondtaak een bericht verstuurt maar de status niet kan bijwerken (door een crash), kan het bericht nogmaals worden aangeboden aan de SaaS-provider. Dit risico wordt beperkt door een uniek bericht-ID mee te sturen.
