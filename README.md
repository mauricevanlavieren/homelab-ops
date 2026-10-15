# Server1 Platform — leeromgeving

**Documentatiestatus: 15 oktober 2026.** Dit is de historische configuratie- en leerrepository voor Server1. Het bestaan van manifests bewijst niet dat deze momenteel live draaien. **Server1 is niet live gecontroleerd** bij deze documentatieronde.

## Start hier

- [Centrale AI-overdracht](https://github.com/mauricevanlavieren/homelab-architecture/blob/main/docs/AI-HANDOVER.md) — vier repositories, architectuur, afspraken, veiligheidsregels en open risico's.
- [AI-instructies](AGENTS.md) — repo-specifieke grenzen.
- [Kubernetes](kubernetes/) — bestaande configuratie; inhoud niet gewijzigd bij deze documentatieronde.
- [Demo-app](demo-app/) en [learning](learning/) — oefeningen en voorbeelden.
- [Architectuurdocumentatie](docs/architecture/) — bestaande documenten.

## Grenzen

Server1 is de **leer-/experimenteeromgeving** en staat los van het operationele Server2-platform. Server1 niet via SSH, kubectl of deployment benaderen zonder expliciete nieuwe opdracht. Niet aannemen dat deze repository een complete reproduceerbare hostconfiguratie of actuele inventaris is. Gebruik geen standaard Kubernetes-context zonder deze te controleren. Geen secrets of private sleutels committen. Voor terminalwerk één commando per stap.

## Overdracht

Bij vervolgwerk eerst branch, HEAD, open PR's, Git-status (indien lokaal beschikbaar), relevante bestanden en de juiste clustercontext controleren. Leg vast wat geverifieerd is en wat onbekend blijft. Wijzigingen aan live systemen, workflows of bestaande manifests vereisen aparte toestemming. Historische bestanden blijven bewaard.
