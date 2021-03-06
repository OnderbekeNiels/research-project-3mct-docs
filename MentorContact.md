# Contact momenten mentor

## Week 01 - 12/01/2022

### Vragen

- Zijn de parameters die ik zal meten goed of heeft u nog suggesties? (Response time client, server & (CPU & RAM)?)
- Is caching met local-storage testen relevant of hou ik het bij de client-side frameworks
- Is dit een relevant nesting level (4 levels diep)?
- Stel dat ik redis cache de beste oplossing vind, moet ik hiervan een full demo voorzien, met alle CRUD actions?

### Need to mention

- Caching methodes die ik zal testen per domein.
- Tonen frontend
- Tonen poging 1 apollo cache directives en bevindingen
- Tonen begin redis cache & uitwerking hiervan
- Tonen eerste metingen met redis cache
- Relevantie eerste tests aanhalen door oplossing met indexen op database

### Feedback

- CPU & RAM tonen kan interssant zijn tijdens de demo met reallife tabs open, disclaimer geven ook omdat dit niet super representatief is
- 4 Levels diep is voldoende
- Eens kijken hoe je data fresh houd, maar volledige crud niet nodig per onderdeel
- Gebruik maken van algemene logger (vb. firebase performance logger) om data te verzamelen

## Week 02 - 19/01/2022

### Volgorde persenteren

1. Docs repository tonen en toevoegen
2. Logger Firebase traag dus eigen implementatie om te meten
3. Directives + Persisted queries
4. Apollo Client InMemory Cache
5. GraphCDN
6. CUD actions bedenkingen (redis, ...)
7. Meetresultaten
8. Vragen

### Vragen

- Moet ik een normale cdn opzetten om dit te tonen of is het aantonen dat ik http caching heb kunnen implementeren voldoende? Als ik dan vb ook kan aantonen dat de s-headers configureerbaar zijn?
- Ik heb nu vooral gekeken hoe ik deze cachings kan opzetten, maar nog niet hoe ik ze efficient kan gebruiken we input/output, moet ik dit voor elke gevonden manier voor enkel voor de manieren die ik zou aanraden als dev?
- Is het testen van **Relay** nog revelant aangezien deze vrij complex is en ook een client uiteindelijk, waarbij de memory speed de factor is bij het inladen? Ook bestaat er **URQL** is een lichtere bundle size heeft en de core dingen van apollo client kan zoals een normalized memory cache.

### Need to mention

- Client caching werkt
- Persisted Queries werkt
- HTTP caching werkt
- Mogelijkheid eigen cache-control headers te zetten
- Overlopen metrics om te bepalen wat de beste strategie is
- Tonen GraphCDN dashboard
- Met de data die ik nu al heb, kan ik al een antwoord hebben op mijn onderzoeksvraag. Alleen is dit geen vanzelf sprekend antwoord omdat het afhangt van de data.

- **Life long learning**: joined discord GraphQl & Redis + subscribed on Apollo YT channel

### Feedback

- Relay skippen, URQL nog eens bekijken omdat deze een veel kleinere bundle size heeft
- Afweging bundle size met voordeel in cache snelheid
- Grafiek opmaken zoals versus een beetje met aanbevelingen cache strategie per nesting en data size
- Grafiek met % sneller maken
- Medium blog post maken en reageren op stackoverflow
- Stappen setup beschrijven in eindoordeel (vb directive maken, dit aanzetten etc...)

## Week 03 - 26/01/2022

### Vragen

- Is mijn vooropgestelde demo goed of wil je nog andere dingen aanbod zien komen tijdens de presentatie?
- Voor de demo van graphCDN, moet ik de purge API demonsteren of is gewoon de code eens tonen voldoende? Want anders moet ik deze online plaatsen, wat geen probleem is voor de duidelijkheid.
- Is een demo in dezelfde stategie??n goed of mag er wat meer dynamiek zijn? zoals delete, andere pagina's etc? Of zet ik deze gewoon online voor de leerkrachten?
- Indienen? Moet ik van elke strategie een branch zip indienen of enkel de beste die ik gevonden had? Aangezien ik elke stratie op een andere branch behandelde?
- Door dockerized prod build is apollo dev tools niet beschikbaar, raad ik docker dan aan in de installatie & gebruikers handleiding of vermeld ik dit gewoon even?
- GraphCDN in dev handleiding of in prod met docker? of beide?
- Ik merkte kleine snelheids improvements (20ms) na dockerizen backend in zelfde netwerk, moet ik metingen aanpassen of dit ergens vermelden?
- Are there some last tips or things I really need to focus on in the presentation?

### Need to mention

- Urql mesurements en bundle results
- Dockerization
- Blogpost & stackoverflow post
- To do's last week

- **Life long learning**: wrote medium blog post https://medium.com/@niels.onderbeke.no/research-project-which-is-the-best-caching-strategy-with-graphql-for-a-big-relational-database-56fedb773b97

### Feedback