---
title: "Verkenning van modelconfiguratie"
---

# Modelconfiguratie (verkenning)

Dit document legt **ideeën** vast voor toekomstige modelconfiguratie. Het is geen
verzonden specificatie. Voor het huidige gedrag, zie:

- [Modellen](/concepts/models)
- [Modeloverschakeling](/concepts/model-failover)
- [OAuth + profielen](/concepts/oauth)

## Motivatie

Operators willen:

- Meerdere authenticatieprofielen per provider (persoonlijk vs werk).
- Eenvoudige `/model`-selectie met voorspelbare fallbacks.
- Duidelijke scheiding tussen tekstmodellen en modellen met beeldcapaciteiten.

## Mogelijke richting (op hoofdlijnen)

- Houd modelselectie eenvoudig: `provider/model` met optionele aliassen.
- Laat providers meerdere authenticatieprofielen hebben, met een expliciete volgorde.
- Gebruik een globale fallbacklijst zodat alle sessies consistent failoveren.
- Overschrijf beeldroutering alleen wanneer dit expliciet is geconfigureerd.

## Open vragen

- Moet profielrotatie per provider of per model zijn?
- Hoe moet de UI profielselectie voor een sessie tonen?
- Wat is het veiligste migratiepad vanaf legacy config-sleutels?


