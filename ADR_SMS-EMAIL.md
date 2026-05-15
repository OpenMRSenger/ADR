# SMS and/or EMAIL

## Status

proposed

## Context

Binnen OpenMRS hebben wij patiëntgegevens nodig om notificaties te sturen. OpenMRS slaat standaard wel telefoonnummers op, maar heeft geen makkelijke optie voor e-mailadressen.

Daarnaast moeten we volgens de opdracht 4 specifieke providers ondersteunen:

- SwiftSend, SecurePost en AsyncFlow: Kunnen zowel SMS als E-mail sturen.

- LegacyLink: Kan alleen SMS sturen.

Bovendien staat er heel duidelijk in functionele eis 1: "Als patiënt wil ik een bericht op mijn telefoon ontvangen."

## Decision

Wat is de verandering die we voorstellen?
Wij besluiten om alleen SMS te gebruiken en geen e-mail in te bouwen. Onze applicatie zoekt het telefoonnummer van de patiënt op in OpenMRS en verstuurt het bericht (de afspraakgegevens) standaard als SMS via één van de 4 geconfigureerde providers.

## Consequences

Wat wordt makkelijker of moeilijker door deze keuze?

Wat wordt makkelijker (Voordelen):

- Geen aanpassingen in OpenMRS: Omdat we alleen telefoonnummers gebruiken, hoeven beheerders OpenMRS niet aan te passen om e-mailadressen te kunnen opslaan. Het werkt meteen.

- Gelijke werking: Omdat provider LegacyLink toch alleen maar SMS aankan, is het veel makkelijker om alle 4 de providers op dezelfde manier te behandelen (alleen telefoonnummer + tekstbericht).

- Direct volgens de eisen: We doen precies wat de opdracht vraagt ("bericht op de telefoon") zonder het onnodig ingewikkeld te maken.

- Emails kunnen mogelijk worden gefilterd naar de spambox. Dit zou betekenen dat patienten geen melding krijgen en mogelijk hun afspraak missen

Wat wordt moeilijker (Nadelen):

- Geen reserve-optie: Als een patiënt een fout of geen 06-nummer heeft ingevuld, krijgt deze geen bericht. We hebben geen e-mail als 'plan B'.

- Korte berichten: Een SMS heeft vaak een limiet van 160 tekens. We moeten de tekst met afspraakgegevens dus erg kort en krachtig houden in vergelijking met een e-mail.
