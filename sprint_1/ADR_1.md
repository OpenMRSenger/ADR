# ADR 1 — Architectuurkeuze: Positionering communicatiemodule t.o.v. OpenMRS

## Status

Superseded (ADR1_1)

## Context

De communicatiemodule moet notificaties versturen naar patiënten van zorginstellingen die gebruikmaken van OpenMRS. Deze module moet wereldwijd inzetbaar zijn voor verschillende organisaties, elk met eigen messaging providers en configuraties.

De centrale vraag is hoe de communicatiemodule gepositioneerd wordt ten opzichte van OpenMRS:

- Wordt de module **ingebouwd in OpenMRS** als interne extensie?
- Of wordt deze ontwikkeld als een **zelfstandige Software-as-a-Service (SaaS) oplossing** die extern gekoppeld wordt?

Belangrijke eisen uit de opdracht:

- Ondersteuning voor meerdere OpenMRS-instanties (multi-tenant)
- Integratie met verschillende messaging providers (zoals SwiftSend, LegacyLink, AsyncFlow en SecurePost)
- Configureerbaarheid per organisatie
- Toekomstbestendigheid (nieuwe providers eenvoudig toevoegen)
- Beveiligde opslag van gevoelige gegevens
- Geen afhankelijkheid van lokale infrastructuur bij klanten

## Decision

De communicatiemodule wordt ontwikkeld als een **zelfstandige SaaS-oplossing**, losstaand van OpenMRS.

De module communiceert met OpenMRS via gestandaardiseerde API’s (bij voorkeur HL7 FHIR), en wordt gehost als externe dienst waarop organisaties zich kunnen abonneren.

## Positionering in architectuur

De gekozen architectuur ziet er als volgt uit:

- OpenMRS fungeert als bronsysteem (bron van patiënt- en afspraakdata)
- De communicatiemodule draait als externe service (SaaS)
- Communicatie verloopt via API’s (bijv. FHIR REST endpoints)
- De communicatiemodule stuurt berichten via externe messaging providers

```plaintext
[OpenMRS instantie(s)]
        |
        | (FHIR API / REST)
        v
[Communicatiemodule (SaaS)]
        |
        v
[Messaging Providers]
```
