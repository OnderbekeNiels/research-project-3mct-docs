# Logs Researchproject

## Onderzoek

### Stappenplan

- [x] Onderzoeken welke caching mogelijkheden er zijn.
- [x] Opzetten JS backend die connect via ORM met db
- [x] Testen Backend caching met directives in Apollo GQL
- [x] Testen Backend caching met Redis cache
- [x] Meten resultaten backend
- [x] Opmaken frontend applicatie in Next.js
- [x] Frontend testen met normale fetch
- [x] Frontend testen met Apollo client en cache mogelijkheden
- [x] Frontend testen met Persisted Queries van Apollo client - backend
- [x] Kijken hoe http caching kan gebruikt worden met Persisted Queries (GET)
- [x] Meetresultaten bijhouden
- [x] GraphCDN testen
- [x] Mutations toevoegen aan app en kijken wat het efficiëntste is om het complete plaatje te hebben **!wip!**
- [x] CUD actions bekijken kwa dev exprience Persisted Queries
- [x] CUD actions bekijken kwa dev exprience GraphCDN
- [x] Bekijken of hetzelfde kan met URQL en hoe dat verhoud met Apollo Client
- [ ] Vergelijking opmaken van de verschillende strategiën.
- [x] Grafiek opmaken zoals versus een beetje met aanbevelingen cache strategie per nesting en data size
- [x] Grafiek met % sneller maken
- [ ] Medium blog post maken en reageren op stackoverflow
- [ ] Stappen setup beschrijven in eindoordeel (vb directive maken, dit aanzetten etc...)

### Metrics

#### Speed
-	Hoe snel kan de data worden opgehaald?
-	Wat voor impact heeft de datasize op het voordeel van caching?
-	Wat voor impact heeft de nesting tree op het voordeel van caching?
#### Dev experience
-	Hoe makkelijk is het te implementeren? 
-	Voor & nadelen?
-	Kosten?
-	Bundle sizes?
#### Valid / freshnes data
-	Hoe kan de data fresh gehouden worden, is dit makkelijk op te zetten?
-	Wat bij Mutations? (CUD) (Cache busting)


### Meet strategie

- Meten op level van nesting (max 4?)
- Meten op data grootte
- Meten op rows op top level

## Caching mogelijkheden

| **Caching GraphQL**                                            |                                                        |                    |
| -------------------------------------------------------------- | ------------------------------------------------------ | ------------------ |
| **Server side**                                                | **CDN**                                                | **Client side**    |
| → Apollo directives caching + Persisted queries = HTTP Caching | → GraphCDN                                             | → Apollo Client v3 |
| → Redis response caching                                       | → HTTP caching with self written cache control headers | → Relay            |
|                                                                |                                                        | → URQL             |

## Logs

### Backend

#### Apollo Server - Directives caching

##### Bevindingen

- Code-first approach bestaat nog niet met Apollo Server, enkel de SQL-first approach. => found solution, het kan wel.
- Apollo garphql express is een express versie van de apollo gql server package, type-graphql zorgt dat een schema wordt gemaakt volgens classes, hierdoor is het met type-graphql lastiger directives zoals bv max age toe te voegen.

###### Postitief

###### Negatief

- [solved : no reason] Database is zeer traag in relation queries, 10gb versie iets sneller maar niet superveel, data komt wel door via graphql maar toch timeout error ook
- type-graphql voorziet niet bepaald een out of the box aanpak om cahcecontrol directive te plaatsen, de voorbeeld repo is zeer onduidelijk: https://github.com/MichalLytek/type-graphql/tree/master/examples/apollo-cache de source van directives etc is deep nested. **UPDATE => https://typegraphql.com/docs/directives.html** goed bekijken
- **Database**: heel traag door de grote data, opgelost door indexen toe te voegen op de fk die gebruikt worden in resolvers. Nu veel sneller.
- Je kan **geen** extra cache hints plaatsen dan maxAge & Scope, lastig dus bij mutations en freshnes. Dit kan echter wel met het doorgeven van het res object en zelf cache-control headers te schrijven.

#### Redis cache

##### Bevindingen

- Redis cache heeft bij de kleine data range niet veel impact en bij het serializeren van de cahce zijn er problemen met **date** type en graphql

###### Postitief

###### Negatief

- **Redis** caching in **GQL** wordt al snel heel complex bij geneste data, zoals bij een delete of mutation, enkel als het id in de key naam zit kan je deze eventueel flushen maar vanaf je bvb een post delete, die in een posts list gecached is van een user, kan je deze niet wissen. Je zou al per post dan moeten gaan cachen en zelfs dan bevat nie elke post persé de gevraagde data. Ik zou redis cache enkel aan raden bij statiche data die niet veel veranderd.

#### Algemeen

- Code-first approach bestaat nog niet met Apollo Server, enkel de SQL-first approach.
- Code-first might work met In-memory cache setup, aangezien dit enkel de responses cached.
- Memcached/Redis setup is een optie.
- Apollo garphql express is een express versie van de apollo gql server package, type-graphql zorgt dat een schema wordt gemaakt volgs classes, hierdoor is het met type-graphql lastiger directives zoals bv max age toe te voegen. In een string file zonder type-graphql gaat dit handiger
- **Redis cache** heeft bij de kleine data range niet veel impact en bij het serializeren van de cahce zijn er problemen met date type en graphql
- Na lang zoeken de resolve field onder de knie gekregen dankzij deze bron: https://frontendmasters.com/courses/advanced-graphql/nested-resolvers-solution/
- [solved : no reason] Database is zeer traag in relation queries, 10gb versie iets sneller maar niet superveel, data komt wel door via graphql maar toch timeout error ook
- type-graphql voorziet niet bepaald een out of the box aanpak om cahcecontrol directive te plaatsen, de voorbeeld repo is zeer onduidelijk: https://github.com/MichalLytek/type-graphql/tree/master/examples/apollo-cache de source van directives etc is deep nested. **UPDATE => https://typegraphql.com/docs/directives.html** goed bekijken
- Bij **Redis** kan je enkel je database response gaan cachen, niet de gql response aangezien elke request query van een gebruiker anders kan zijn.
- **Database**: heel traag door de grote data, opgelost door indexen toe te voegen op de fk die gebruikt worden in resolvers. Nu veel sneller.
- **Redis** wordt al snel heel complex bij geneste data, zoals bij een delete of mutation, enkel als het id in de key naam zit kan je deze eventueel flushen maar vanaf je bvb een post delete, die in een posts list gecached is van een user, kan je deze niet wissen. Je zou al per post dan moeten gaan cachen en zelfs dan bevat nie elke post persé de gevraagde data. Ik zou redis cache enkel aan raden bij statiche data die niet veel veranderd.
- **Apollo Server**, je kan geen extra cache hints plaatsen dan maxAge & Scope, lastig dus bij mutations en freshnes. Dit kan echter wel met het doorgeven van het res object en zelf cache-control headers te schrijven.

### Frontend

#### Apollo Client v3 - Client InMemory cache

##### Bevindingen

- Apollo client is makkelijk op te zetten in react
- Opletten dat cache niet te veel geheugen inneemt hiermee
- 4 options kwa cache control, zeer makkelijk in gebruik
- **Juistheid van data**: er moet aandachtig gekeken worden naar de fetch policy bij mutations, soms kan je erop vertrouwen dat de clien het juist voor jou doet, maar bij deletes moet je zelf de cache updaten en bij een create in sommige gevallen bij geneste data een refetch gaan doen.
- Cached volgens \_\_typename & id => houd per geneste data ook bij & merged wanneer er nieuwe bij komt
- In apollo client moet de query string direct mee gegeven worden aan gql, en niet eerst via variabele, anders werkt de cache niet
- Bij een **mutation** moet het return \_\_typename & id aanwegzig zijn om de client cache te updaten

###### Positief

- Zeer snel
- Makkelijk op te zetten
- Handig met useQuery, useLazyQuery & useMutation method, hooks in general.
- Goed dev tool in chrome beschikbaar
- Bij de hooks om data te fetchen heb je direct een error, loading & data destructuring die het heel developper friendly maken.
- Wanneer de cache gebruikt word zie je je data kwasie direct zonder zelf de loader te zien.

###### Negatief

- Niet atlijd duidelijke documentatie, je moet vaak dingen van op 2 verschillende locaties samen gooien.

#### Apollo Client v3 - Persisted Queries

##### Bevindingen

- Makkelijk op te zetten
- Controleer baar in network tab
- Stuurt niet altijd een GET req ? Soms bij reload enkel posts?
- Werkend gekregen met persisted queries, directive toevoegen werkt. default moet hoger of 0 zijn in index.js server file

###### Positief


###### Negatief

- Wanneer je "stale-while-revalidate" gebruikt zal je nieuwe data pas gebruikt worden na 2x reloaden, dit omdat de revalidate response opgeslagen wordt in de cache en de volgende keer pas getoont zal worden, je zit dus een stapje achter
- Niet veel controle over mutations die gebreuren. Om voor fresh content te zorgen moet er atlijd een request naar backend gestuurd worden met bvb "no-cache" of "stale-while-revalidate" headers, dit is meer voor static content die niet veel veranderd


### CDN

#### GraphCDN

##### Bevindingen

- Portforwarded server met ip werkt niet, moet gehost worden op een dn. Error afkomstig van cloudflare.
- Er is een dev omgeving mogelijk om bovenstaand probleem op te lossen.
- Wanneer je op een type caching plaatst, dan zal een query die die specifieke type bevat ook cachen, bv UsersAll waar je ook de posts ophaald. Iets om rekening mee te houden. => je kan ook puur op query's caching plaatsen, **problem solved**

###### Positief

- Snel
- Makkelijk op te zetten
- Analytics out of the box
- Je kan serverside http caching combineren
- Auto invalidation wanneer je de juiste response stuurt na een mutation => werkt bij Creates and updates, kan manueel gedaan worden met **purging api** ! **zeer handig**

###### Negatief

- Site wat buggy? Error count klopt maar geen errors getoond.


#### Algemeen

- Opzet baar zonder GraphCDN, persisted queries maken het mogelijk met GET te werken en deze op eender welke cdn te gaan cachen.

