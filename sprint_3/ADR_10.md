# ADR-010: Webhook authenticatie via API key

## Status

Accepted

## Context

De rest-service ontvangt inkomende HTTP POST-verzoeken (webhooks) van de OMOD wanneer afspraken worden aangemaakt of gewijzigd. Deze endpoint moet beveiligd zijn zodat alleen de gekoppelde OMOD-instanties events mogen insturen.

Vier opties zijn overwogen:

- Optie A: API key via Authorization header
- Optie B: HMAC-signature over de payload
- Optie C: Mutual TLS (wederzijdse certificaatauthenticatie)
- Optie D: JWT-token (kortstondige tokens via een identity provider)

## Decision

Er is gekozen voor Optie A: API key authenticatie via de Authorization header.

De rest-service vergelijkt de meegestuurde key met een geconfigureerde waarde (via omgevingsvariabele). Verzoeken zonder geldige key krijgen een 401-response. De key wordt geconfigureerd in zowel de OMOD (als Global Property) als de rest-service (als omgevingsvariabele).

Redenen voor deze keuze:

- Eenvoudig te implementeren en te configureren. Geen extra infrastructuur nodig.
- Past bij de bestaande configuratiestijl van de OMOD (Global Properties) en de rest-service (omgevingsvariabelen).
- Voldoende beveiliging voor een intern-gebruik scenario waarbij HTTPS de transportlaag beveiligt.
- HMAC voegt integriteitscontrole per bericht toe, maar vereist gedeelde signing-logica aan beide kanten en maakt debugging complexer.
- Mutual TLS vereist certificaatbeheer per organisatie, wat de operationele overhead flink verhoogt.
- JWT vereist een identity provider of token-uitgifte mechanisme, wat overkill is voor dit gebruik.

## Consequences

Wordt eenvoudiger:

- Configuratie: één gedeeld secret in de OMOD en de rest-service is voldoende.
- Debugging: de Authorization header is direct zichtbaar in logs (mits bewust gelogd).
- Geen certificaten of externe systemen nodig voor authenticatie.

Vraagt extra aandacht:

- Bij lekken van de API key heeft een aanvaller directe toegang. Key rotatie moet dan snel uitgevoerd kunnen worden via een configuratiewijziging.
- Geen integriteitscontrole op berichtniveau. Een aanvaller met de key kan willekeurige payloads insturen. Dit risico wordt beperkt door inputvalidatie in de webhook-controller.
- HTTPS is verplicht. Zonder TLS wordt de key in plaintext verstuurd.
