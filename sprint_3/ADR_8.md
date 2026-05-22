# ADR-008: AOP voor event-interceptie in de OMOD

## Status

Accepted

## Context

De OMOD moet afspraakevents opvangen op het moment dat een afspraak wordt aangemaakt of gewijzigd in OpenMRS of Bahmni. OpenMRS biedt hiervoor meerdere mechanismes:

- Optie A: Spring AOP om methodes van de Bahmni AppointmentsService te intercepteren
- Optie B: Het native OpenMRS event-systeem (via de Event module en luisteraars)

Bahmni's AppointmentsService is het centrale punt voor het aanmaken en wijzigen van afspraken. De OMOD mag de broncode van Bahmni niet aanpassen.

## Decision

Er is gekozen voor Optie A: Spring AOP via AppointmentServiceAdvice.

De OMOD registreert een Spring AOP advice op de AppointmentsService bean. Elke aanroep van de relevante methodes (aanmaken, wijzigen) wordt onderschept, ongeacht wie de aanroep initieert. Na de methodeaanroep schrijft de advice een record naar de outbox-tabel.

Redenen voor deze keuze:

- Bahmni stuurt niet altijd OpenMRS-events bij afspraakwijzigingen. De AppointmentsService bean is het enige betrouwbare interceptiepunt.
- AOP werkt ongeacht hoe de service wordt aangeroepen (REST, interne code, andere modules).
- Geen aanpassingen in Bahmni-broncode nodig. De OMOD blijft een losstaande extensie.
- Het OpenMRS event-systeem vereist dat het Bahmni Appointments-pakket zelf events publiceert, wat niet gegarandeerd het geval is.

## Consequences

Wordt eenvoudiger:

- Alle afspraakwijzigingen worden betrouwbaar onderschept, ook bij aanroepen via interne Bahmni-logica.
- De OMOD hoeft geen Bahmni-code aan te passen of te forken.

Vraagt extra aandacht:

- De advice koppelt aan de interface van AppointmentsService. Als Bahmni de signatuur van relevante methodes wijzigt, moet de advice bijgewerkt worden.
- AOP-configuratie is minder zichtbaar dan directe code. Nieuwe teamleden moeten weten dat de interceptie via Spring AOP verloopt.
- Foutafhandeling in de advice moet voorkomen dat een exception in de outbox-schrijfstap de originele Bahmni-operatie stoort.
