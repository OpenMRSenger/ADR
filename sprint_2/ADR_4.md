# ADR-004 — Integratiemethode: hoe koppelt de module aan OpenMRS? (Heroverweging)

## Status

Accepted

## Context

Na verder onderzoek en discussie is de initiële keuze voor Optie A (FHIR REST polling) in ADR-003 heroverwogen. De drie integratieopties zijn opnieuw geëvalueerd gegeven de specifieke vereisten voor real-time afspraakverwerking en de capaciteiten van de aangesloten OpenMRS-instanties.

## Decision

Gekozen wordt voor **Optie C: Webhook / push vanuit OpenMRS** in plaats van de eerder gekozen Optie A.

De communicatiemodule ontvangt real-time HTTP POST-verzoeken van een aangepaste OpenMRS-module wanneer afspraken worden aangemaakt of gewijzigd. Per organisatie wordt een webhook-endpoint URL en authenticatietoken geconfigureerd in de aangepaste OpenMRS-module. De communicatiemodule verwerkt inkomende events onmiddellijk en stuurt notificaties uit op basis van de actuele afspraakstatus en timing.

De argumentatie ten opzichte van de alternatieven:

- Optie C biedt volledig real-time verwerking van afspraakwijzigingen. Dit elimineert de vertraging die ontstaat bij polling-benaderingen en zorgt ervoor dat annuleringen, uitstel of status-wijzigingen onmiddellijk verwerkt worden.
- Optie C is veel efficiënter qua resource-gebruik: de OpenMRS FHIR API ontvangt geen repetitieve polling-verzoeken, wat de belasting aanzienlijk verlaagt, vooral bij meerdere organisaties.
- Optie A vereist persistente polling met risico op vertragingen en verhoogde API-belasting. Event-driven is inherent efficiënter.
- Optie B vereist aanvullende modules (Atom Feed, Event module) en event-driven verwerking via feeds, maar is minder flexibel dan directe webhooks gericht op de specifieke communicatiemodule.
- Hoewel een aangepaste OpenMRS-module vereist is, is dit eenmalig per instantie en kan aan iedere organisatie ter beschikking gesteld worden als referentie-implementatie. Dit is acceptabel gezien de aanzienlijke voordelen in real-time responsiviteit en efficiëntie.
- De webhook-benadering kan gemakkelijk uitgebreid worden voor andere typen events (patiëntgegevens-wijzigingen, clinicale data) in de toekomst.

## Consequences

**Wordt eenvoudiger:**

- Real-time verwerking van afspraken: wijzigingen worden onmiddellijk verwerkt, geen polling-latentie.
- Reduceerde belasting op OpenMRS FHIR API: geen periodieke polling-verzoeken, alleen event-gebaseerde communicatie.
- Nauwkeurigheid van notificaties: annuleringen en wijzigingen worden verzonden vóórdat de gebruiker andere actie onderneemt.
- Schaalbaarheid zonder prestatie-impact: hoe meer organisaties, des te minder API-load (tegengesteld aan polling).
- Flexibiliteit voor toekomstige uitbreiding: webhooks kunnen gemakkelijk andere event-types ondersteunen.

**Wordt moeilijker of vraagt extra aandacht:**

- Installatie en configuratie van de aangepaste OpenMRS-module: elke aangesloten organisatie moet de webhook-module installeren en het webhook-endpoint configureren. Dit vereist wat meer setup dan puur FHIR polling.
- Downtime-resiliëntie: als de communicatiemodule offline gaat, kunnen OpenMRS-fouten ontstaan wanneer webhooks niet bereikbaar zijn. Dit vereist robuuste foutafhandeling aan de OpenMRS-kant (retry-logica, deadletter queues).
- Authenticatie en beveiliging: webhooks moeten beveiligd zijn tegen onbevoegde aanroepen. Tokens en HTTPS zijn essentieel en moeten zorgvuldig beheerd worden.
- Vereist twee-richtingscommunicatie: de communicatiemodule moet actief luisteren naar inkomende POST-verzoeken en beschikbaar zijn op het internet. Dit kan extra firewallconfiguratie vereisen.
- Dependency on custom module versioning: OpenMRS-versie-updates kunnen invloed hebben op de webhook-module; versiebeheer en compatibiliteit moeten zorgvuldig worden beheerd.
