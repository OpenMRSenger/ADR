# ADR-009: Strategy pattern voor messaging providers

## Status

Accepted

## Context

De rest-service moet notificaties versturen via vier externe messaging providers: SwiftSend, AsyncFlow, SecurePost en LegacyLink. Elke provider heeft een eigen API en authenticatiemethode:

- SwiftSend: eigen API-structuur
- AsyncFlow: eigen API-structuur
- SecurePost: OAuth2 client credentials
- LegacyLink: HTTP Basic Auth, alleen SMS

Nieuwe providers moeten in de toekomst eenvoudig toegevoegd kunnen worden zonder bestaande code aan te passen.

Twee opties zijn overwogen:

- Optie A: Losse if/switch logica per provider in de notificatieservice
- Optie B: Strategy pattern met een gedeelde interface en een concrete adapter per provider

## Decision

Er is gekozen voor Optie B: het strategy pattern via een port-adapter structuur.

Elke provider implementeert de MessagingProviderPort interface. Een abstracte basisklasse (AbstractRestMessagingAdapter) bevat de gedeelde HTTP-logica. Per provider is er een concrete adapter die de provider-specifieke aanroep en authenticatie afhandelt. De actieve provider wordt geselecteerd op basis van configuratie (ProviderConfig).

Redenen voor deze keuze:

- Een nieuwe provider toevoegen vereist alleen een nieuwe klasse. Bestaande adapters en de service hoeven niet aangepast te worden.
- Elke adapter is los te testen zonder afhankelijkheid van andere providers.
- De notificatieservice heeft geen kennis van provider-specifieke details. Koppeling blijft laag.
- De gedeelde basisklasse voorkomt duplicatie van HTTP-boilerplate.

## Consequences

Wordt eenvoudiger:

- Nieuwe provider toevoegen: nieuwe klasse schrijven die MessagingProviderPort implementeert.
- Testen per provider in isolatie, zonder andere adapters te beinvloeden.
- Configuratiegestuurde providerselectie zonder code-aanpassingen.

Vraagt extra aandacht:

- Extra abstractielaag. Voor een eenvoudige aanroep zijn nu meerdere klassen betrokken.
- De basisklasse vormt een gedeeld punt. Aanpassingen daar raken alle adapters.
- ProviderConfig moet correct geconfigureerd zijn. Een ontbrekende of foute providernaam geeft een runtime-fout.
