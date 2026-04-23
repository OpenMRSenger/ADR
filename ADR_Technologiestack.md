# ADR 2 — Technologie stack

## Status

Accepted

## Context

De communicatiemodule wordt ontwikkeld als een zelfstandige SaaS-oplossing die notificaties verstuurt namens meerdere OpenMRS-instanties. De module moet schaalbaar, veilig, uitbreidbaar en betrouwbaar zijn.

Belangrijke eisen uit de opdracht:

- Ondersteuning voor meerdere organisaties (multi-tenant)
- Integratie met externe messaging providers
- Verwerking van afspraakdata (FHIR)
- Betrouwbare aflevering van notificaties (inclusief retries en logging)
- Beveiligde opslag van gevoelige gegevens (zoals API-keys en tokens)
- Toekomstbestendigheid en uitbreidbaarheid
- Onderhoudbaar door het team

Daarnaast moet de gekozen technologie stack aansluiten bij de kennis en vaardigheden van het team, zodat de implementatie haalbaar blijft binnen de projecttijd.

## Decision

De communicatiemodule wordt gebouwd met de volgende technologie stack:

| Component          | Keuze                   |
| ------------------ | ----------------------- |
| Programmeertaal    | Java                    |
| Framework          | Spring Boot             |
| API-standaard      | REST + HL7 FHIR         |
| Messaging queue    | RabbitMQ                |
| Database           | PostgreSQL              |
| Monitoring         | Prometheus + Grafana    |
| Containerisatie    | Docker                  |
| Secrets management | (bijv.) HashiCorp Vault |

## Argumentatie per keuze

### Java

Java is een veelgebruikte programmeertaal in enterprise- en healthcare-omgevingen.

**Voordelen:**

- Sterke typeveiligheid
- Grote community en veel libraries
- Veel gebruikt binnen zorgsystemen
- Goede ondersteuning voor schaalbare backend systemen

---

### Spring Boot

Spring Boot wordt gebruikt als backend framework.

**Voordelen:**

- Snelle ontwikkeling van REST API’s
- Ingebouwde ondersteuning voor dependency injection
- Goede integratie met security, messaging en databases
- Veelgebruikte standaard in enterprise applicaties

---

### REST + HL7 FHIR

De module communiceert via REST API’s en gebruikt de HL7 FHIR-standaard voor zorgdata.

**Voordelen:**

- FHIR is de standaard voor healthcare interoperability
- Directe aansluiting op OpenMRS
- Gestandaardiseerde resources zoals `Appointment`, `Patient`, `Location`
- Toekomstbestendig en breed ondersteund

---

### RabbitMQ

RabbitMQ wordt gebruikt als messaging queue voor asynchrone verwerking.

**Voordelen:**

- Betrouwbare message delivery
- Ondersteuning voor retries en queueing
- Ontkoppeling tussen componenten (bijv. scheduling en verzending)
- Geschikt voor event-driven architectuur

---

### PostgreSQL

PostgreSQL wordt gebruikt als relationele database.

**Voordelen:**

- Betrouwbaar en stabiel
- Ondersteuning voor complexe queries
- Geschikt voor gestructureerde data zoals afspraken en logs
- Open source

---

### Prometheus + Grafana

Voor monitoring en observability.

**Voordelen:**

- Inzicht in performance en systeemstatus
- Visualisatie van metrics (bijv. aantal verzonden berichten)
- Ondersteuning voor alerts bij fouten of downtime

---

### Docker

Voor containerisatie van de applicatie.

**Voordelen:**

- Consistente deployment-omgeving
- Eenvoudig schaalbaar
- Geschikt voor cloud deployment (Kubernetes-ready)

---

### Secrets management (bijv. HashiCorp Vault)

Voor veilige opslag van gevoelige gegevens.

**Voordelen:**

- Geen opslag van secrets in code of config files
- Centrale en veilige toegang tot credentials
- Ondersteunt encryptie en toegangsbeheer

---

## Alternatives

### Alternatief 1: Node.js (JavaScript/TypeScript)

#### Voordelen:

- Snelle ontwikkeling
- Veel libraries beschikbaar
- Goede ondersteuning voor async processing

#### Nadelen:

- Minder gebruikelijk in healthcare-omgevingen
- Minder sterke typeveiligheid (tenzij TypeScript)
- Team heeft mogelijk minder ervaring

#### Reden van afwijzing:

Hoewel Node.js geschikt is voor snelle ontwikkeling, sluit Java beter aan bij enterprise en healthcare standaarden, en biedt het meer robuustheid voor dit type systeem.

---

### Alternatief 2: Apache Kafka (in plaats van RabbitMQ)

#### Voordelen:

- Zeer schaalbaar
- Geschikt voor high-throughput event streaming

#### Nadelen:

- Complexer om te beheren
- Overkill voor huidige use case
- Hogere leercurve voor het team

#### Reden van afwijzing:

Voor deze applicatie is RabbitMQ voldoende en eenvoudiger te implementeren. Kafka biedt meer schaal, maar voegt onnodige complexiteit toe.

---

### Alternatief 3: NoSQL database (bijv. MongoDB)

#### Voordelen:

- Flexibel datamodel
- Geschikt voor ongestructureerde data

#### Nadelen:

- Minder geschikt voor relationele data (zoals afspraken en relaties tussen patiënten)
- Minder sterke consistentie

#### Reden van afwijzing:

De data in deze applicatie is sterk gestructureerd en relationeel, waardoor PostgreSQL beter past.

---

### Alternatief 4: Monolithische architectuur zonder messaging queue

#### Voordelen:

- Simpeler te implementeren
- Minder infrastructuur nodig

#### Nadelen:

- Slechter schaalbaar
- Geen asynchrone verwerking
- Moeilijker om retries en foutafhandeling te implementeren

#### Reden van afwijzing:

De eisen rondom betrouwbaarheid en schaalbaarheid maken een asynchrone architectuur met messaging noodzakelijk.

---

## Consequences

### Positieve gevolgen:

- Robuuste en schaalbare architectuur
- Goede aansluiting op healthcare standaarden (FHIR)
- Betrouwbare verwerking van notificaties
- Eenvoudig uitbreidbaar met nieuwe providers

### Negatieve gevolgen:

- Hogere complexiteit (meerdere componenten)
- Leercurve voor sommige tools (bijv. RabbitMQ, monitoring tools)
- Meer infrastructuur nodig voor deployment

## Conclusie

De gekozen technologie stack biedt een goede balans tussen betrouwbaarheid, schaalbaarheid en ontwikkelbaarheid. De combinatie van Java, Spring Boot, RabbitMQ en PostgreSQL sluit goed aan bij de eisen van de opdracht en maakt het mogelijk om een toekomstbestendige SaaS communicatiemodule te realiseren.
