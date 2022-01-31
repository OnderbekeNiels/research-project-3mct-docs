# Demo 1 feb 2O22

## Open programs

- tab with localhost:3000/users/1
- tab with graphCDN dashboard
- tab with blog post
- tab with stack overflow article
- Discord
- tab with youtube channel apollo
- foto automatic persisted queries
- vscode user-resolver
- vscode posts type definition
- vscode userId page nextjs
- docker compose

## Need to mention

- Wat is mijn onderzoek & wat is de context?
- Welke bronnen heb je gebruikt ?
- Welke bron draagde veel bij tot jouw onderzoek?
- Levenslang leren: In welke mate heb je communities rond jouw onderzoeksthema actief opgevolgd?
- Levenslang leren: Heb je gebruik gemaakt van e-learning platformen? Waren ze nuttig?
- Levenslang leren: Ben je op zoek gegaan naar specifieke communities? Heb je ze actief opgevolgd? En misschien zelfs deelgenomen?
- Medium blog post
- Comment op stackoverflow
- Vergelijking Apollo Client vs Urql

## Demo script

### Intro

- Naam
- Onderzoeksvraag vertellen: "Welke is de beste caching strategie bij GraphQL voor een grote relationele database?"
- Context: In de API wereld hoor je vaker wel eens dat GraphQL caching breekt zoals wij het kennen met Rest API's.
  Dit doordat ze highly customizable zijn in het opvragen van data & dus meer client specifiek zijn. Ook werkt GraphQL met POST requests en kan je dus geen HTTP caching uit gaan voeren.
  Ik kan jullie nu al zeggen dat er voor deze stellingen oplossingen zijn en zal deze nu demonstreren.
- Ik heb in dit onderzoek niet enkel gekeken naar pure snelheid maar ook naar developper experiences en naar cache busting mogelijkheden.

### Demo itself

- Zoals jullie kunnen zien heb ik een kleine frontend gemaakt in Next.js voor mijn test database & de testen op te gaan uitvoeren. Ik gebruikte een 10 GB SQL database dump van stack overflow hiervoor.
- Ik zal jullie de door mij best bevonden caching methodes gaan demonstreren en even vermelden wat de andere opties waren die ik gebruikte.
- In deze demo zal een pagina inladen met 4 geneste level, zijnde gegevens van een user, zijnde de posts, de comments van de posts en de users van de comments. Dit is een real-life voorbeeld van hoe zo een query zou kunnen verlopen.

#### Client-side: Apollo Client v3 InMemory cache (next niet in docker om devtools te tonen)

- Library voor GraphQL in een React omgeving
- in-memory genomalizeerde cache, dus elke type/entiteit wordt op zichzelf opgeslagen
- Werkt aan de hand van fetch policies
- Tonen zonder cache, eerste load
- Tonen snelheidsmeting in pop-up
- Tonen met cache, DevTools open en tonen geen network request
- Tonen snelheidsmeting in pop-up
- Cache busting tonen door post aan te passen.
- Oplossing tegen niet automatische cache busting -> zelf definiëren, meer info in handleiding en blog post

- Zeer snel, loading bar komt niet eens in zicht

- Een alternatief is Urql, heeft een kleinere package bundle maar is gemiddeld bij elke request 10ms seconden trager

#### Server-side: HTTP caching met Automatic Persisted Queries

- HTTP caching met de cache-control header is weldegelijk mogelijk door Automatic Persisted Queries
- Tonen schema
- Tonen directives manueel en even vermelden met Apollo directives met de directives op types etc en beperking maxAge & Scope en dat dit gedaan is met Apollo Server & TypeGraphQL

- Frontend moet je dit aanzetten, kan met Apollo Client maar bijvoorbeeld ook met Urql

- Tonen network tab bij inladen eerste keer & 3e keer dat de cache inkickt

- ! => neemt steeds de laatste fieldresolver als response cache-control header, probeer dus enkel dit te plaatsen op top-level resolvers.

bronnen eerste 2: Apollo youtube channel, Apollo Docs, Docs van TypeGraphQL, Youtube van Ben Awad

#### CDN: GraphCDN

- graphcdn serve --backend-port 4000 --service stackof-rp --path /graphql      

- CDN die cached op POSTS
- Specifiek gemaakt voor GraphQL
- Purging API tegen cache busting
- Automatische updates zoals Client

- Tonen headers & caching rules
- Rules zijn inzet baar voor types, queries & fields
- Headers instel baar en security mogelijkheden

- Tonen dashboard

bronnen: docs graph cdn & video met Max Stoiber (co-founder)

### Conclusion

- Er is niet echt een specifiek beste oplossing, dit hangt af van hoevaak je data veranderd, hoe groot je responses zijn, of snelheids de belangrijkste factor is etc.
- Ik heb een guide gemaakt en dit als Medium blogpost gepublished. Hierin behandel ik ook even caching met redis.
- Tonen metingen en guide metrics
- Levenslang leren: In welke mate heb je communities rond jouw onderzoeksthema actief opgevolgd?
  - Ik heb mezelf gesubscriped op het Apollo Youtube channel, er komen hier regelmatig gastsprekers op van verschillende bedrijven en dat is zeer leerrijk
  - Ik heb de GraphQL,Redis & GraphCDN discord gejoined voor mocht ik in de toekomst vragen hebben of mensen zou kunnen helpen
  - Ik heb gereageerd op een Stack Overflow vraag, iemand had problemen met Apollo Server en raakte daar niet aan uit. Ik kon gelukkig helpen hiermee.
- Levenslang leren: Heb je gebruik gemaakt van e-learning platformen? Waren ze nuttig?
  - Geen gebruikt, alles was te vinden op youtube en de docs van de verschillende technologiën

### Potential questions

- hoe zit het met de time passes in de xd rond automatic
