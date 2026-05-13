# ADR-005 — Architectuurbesluit: Monoliet versus Microservices

## Status

Proposed

## Context

Bij het ontwerp van de communicatiemodule moet een keuze gemaakt worden voor de architectuurparadigma: een monolithische applicatie of een microservices-architectuur. Deze keuze beïnvloedt:

- de complexiteit van deployment en operationalisering;
- de mogelijkheden voor horizontale en verticale schaling;
- de kommunikatie tussen componenten (inter-process vs netwerk);
- de onderhoudbaarheid en evolutiebarheid van het systeem;
- de vereisten aan infrastructuur en DevOps-tooling;
- de latentie en betrouwbaarheid van communicatie tussen onderdelen.

De twee opties zijn onderzocht:

**Optie A — Monolithische architectuur**
Alle functionele componenten (OpenMRS-integratie, notificatieverwerking, plan uitvoering, authenticatie) zijn in één enkele applicatie geïntegreerd. Deployment gebeurt als één eenheid. Interne communicatie verloopt via in-process function calls.

**Optie B — Microservices-architectuur**
Elk functioneel domein (OpenMRS-integratie, notificatieverwerking, plan uitvoering) draait als aparte service met eigen datastore. Diensten communiceren via HTTP/REST, gRPC of message queues. Elke service kan onafhankelijk geschaald en gedeployd worden.

## Decision

Gekozen wordt voor **Optie A: Monolithische architectuur**.

De communicatiemodule wordt als een enkele applicatie opgezet met duidelijke interne lagenarchitectuur (API-laag, business-logic-laag, data-access-laag). Dit biedt een solid fundament voor schaalbaarheid met minimale operationele complexiteit in de huidige fase.

De argumentatie ten opzichte van de alternatieven:

- **Schaalbaarheid via horizontale replicatie**: Een monoliet kan zeer effectief geschaald worden door meerdere instanties in te zetten achter een load balancer. Dit is één van de meest bewezen en stabiele schaalpatronen in productieomgevingen. Veel van 's werelds grootste applicaties runnen op monolithische architecturen.
- **Voorkoming van gedistribueerde systeemcomplexiteit**: Microservices introduceren netwerklatentie, tijdelijke storingen, race conditions en data-inconsistentie. Dit verhoogt operationele complexiteit exponentieel. Voor de huidige vereisten zijn deze extra lasten niet gerechtvaardigd.
- **Eenvoudige transactionele integriteit**: Alle gegevensmutaties kunnen binnen één database-transactie plaatsvinden. In een microservices-omgeving zou complex compensatielogica (saga-patroon) nodig zijn, wat fouten en inconsistenties kan veroorzaken.
- **Prestatie en latentie**: In-process communicatie is altijd sneller dan netwerkaanroepen. Dit is cruciaal voor real-time notificatieverwerking.
- **Operationeel beheer**: Één applicatie betekent één deploymentproces, één monitoringpijplijn, één logverzameling. Dit vermindert de drempel voor organisaties om het systeem op te zetten en bij te houden.
- **Kostefficiëntie**: Microservices vereisen meer infrastructuur (load balancers, API gateways, distributed tracing, etc.). Een monoliet kan op beperktere middelen draaien.
- **Toekomstige flexibiliteit**: Als schaling later echt een knelpunt wordt (niet waarschijnlijk voor een communicatiemodule die hoofdzakelijk I/O-gebonden is), kunnen afzonderlijke diensten uit de monoliet worden geextraheerd zonder architectuur volledig opnieuw te moeten opzetten.

## Consequences

**Wordt eenvoudiger:**

- Onboarding van developers: één codebase, één deployment-pipeline, gedeelde technische stack.
- Lokale development en testen: volledige applicatie draait op één laptop; geen containers of lokale services nodig.
- Foutherstel: stacktraces en logging zijn lokaal binnen één proces; geen gedistribueerde logtracing nodig.
- Deployment en rollback: één applicatie updaten is veel veiliger en sneller dan meerdere diensten.
- Schaalbaarheid in de praktijk: aanpassing van replica-aantallen en resource-limieten in Kubernetes/Docker is eenvoudig en directe workt.

**Wordt moeilijker of vraagt extra aandacht:**

- Grote teams: als het team groeit naar tientallen developers, kan conflictmanagement in één codebase lastiger worden. Dit kan later aan de orde staan, maar is niet relevant voor huidige schaalgrootte.
- Onafhankelijke schaling van subsystemen: als in de verre toekomst OpenMRS-polling veel meer resources nodig heeft dan notificatieverwerking, kan dit niet zonder beide te schalen. Dit kan opgelost worden door toekomstige opsplitsing.
- Deployment risk: een bug in een onderdeel kan het hele systeem beïnvloeden. Dit wordt beperkt door goede testing, code review en staged rollouts.
- Interne module-ontkoppeling: omdat alles in één proces draait, is het verleidelijk om directe afhankelijkheden tussen modules toe te staan. Dit vereist discipline in code-organisatie (duidelijke package boundaries, geen circulaire imports).
