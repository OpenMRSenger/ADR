# ADR-005: Monoliet versus Microservices

## Status

Accepted

## Context

Bij het ontwerp van de communicatiemodule moest een keuze gemaakt worden tussen een monolithische applicatie en een microservices-architectuur. Deze keuze heeft invloed op de complexiteit van deployment, schaalbaarheid, communicatie tussen componenten, onderhoudbaarheid en infrastructuurvereisten.

Twee opties zijn onderzocht:

**Optie A - Monolithische architectuur**
Alle componenten (OpenMRS-integratie, notificatieverwerking, planuitvoering, authenticatie) zitten in één applicatie. Interne communicatie verloopt via directe functie-aanroepen.

**Optie B - Microservices-architectuur**
Elk domein draait als aparte service met eigen database. Services communiceren via HTTP, gRPC of message queues en kunnen onafhankelijk worden geschaald.

## Decision

Er is gekozen voor **Optie A: een modulaire monoliet**.

De communicatiemodule wordt als één applicatie opgezet, maar met duidelijke interne grenzen per domein. Elk domein (OpenMRS-integratie, notificatieverwerking, planning, authenticatie) is een aparte module met een eigen interface. Modules zijn losjes gekoppeld, ook al draaien ze in hetzelfde proces.

Redenen voor deze keuze:

- **Schaalbaarheid**: Een monoliet kan goed horizontaal schalen via meerdere instanties achter een load balancer. Veel grote systemen draaien op deze manier.
- **Minder complexiteit**: Microservices brengen netwerkproblemen, race conditions en data-inconsistentie met zich mee. Die extra complexiteit is voor dit project niet nodig.
- **Eenvoudige transacties**: Alle data-aanpassingen kunnen binnen één database-transactie. Bij microservices zou je complexe compensatielogica (zoals het saga-patroon) nodig hebben.
- **Betere prestaties**: Communicatie binnen één proces is altijd sneller dan via het netwerk. Dat is belangrijk voor real-time notificaties.
- **Eenvoudig beheer**: Één applicatie betekent één deploymentproces, één monitoring en één logverzameling.
- **Lagere kosten**: Microservices vereisen meer infrastructuur. Een monoliet kan met minder middelen draaien.
- **Toekomstige flexibiliteit**: Door de modulaire opzet is het later eenvoudig om een module los te koppelen als aparte service, als dat nodig wordt.

## Consequences

**Wordt eenvoudiger:**

- Onboarding van developers: één codebase en één deployment-pipeline. Modules helpen nieuwe developers snel wegwijs te worden.
- Lokaal ontwikkelen en testen: de volledige applicatie draait op één laptop, zonder extra containers of services.
- Foutherstel: logs en stacktraces zijn beschikbaar binnen één proces, geen gedistribueerde tracing nodig.
- Deployment en rollback: één applicatie updaten is veiliger en sneller dan meerdere services beheren.
- Schaalbaarheid: het aanpassen van replica-aantallen in Kubernetes of Docker is eenvoudig.
- Codestructuur: duidelijke module-grenzen voorkomen spaghetti-code en maken refactoring veiliger.

**Vraagt extra aandacht:**

- Grote teams: als het team sterk groeit, kan werken in één codebase lastiger worden. Dit is nu niet relevant.
- Aparte schaling per module: modules kunnen niet los van elkaar worden geschaald. Als één module veel meer resources nodig heeft, kan die later worden losgetrokken als aparte service.
- Deployment risico: een bug in één onderdeel kan het hele systeem raken. Dit wordt beperkt door goede tests, code reviews en gefaseerde uitrol.
- Modulaire discipline: de grenzen tussen modules moeten actief bewaakt worden, zodat er geen circulaire afhankelijkheden of directe koppelingen tussen interne module-onderdelen ontstaan.
