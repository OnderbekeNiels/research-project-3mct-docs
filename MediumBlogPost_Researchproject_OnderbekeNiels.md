![](https://miro.medium.com/max/1400/1*LV6YwfzFpQbTlOLC-KLQ_A.png)picture: [https://labs.kadaster.nl/demonstrators/architectuur-selfservice/GraphQLETL/](https://labs.kadaster.nl/demonstrators/architectuur-selfservice/GraphQLETL/)

# Research project: Which is the best caching strategy with GraphQL for a big relational database?

# Intro

As a final year student [web development](https://mct.be/programma/next-web-developer/) at [**Howest**](https://mct.be/), I’ve had the task to do research on a topic related to my studies. We had to formulate a research question and, in this article, I will try to give you an answer to that question.

In this research project I’ve looked to the 3 domains where you can cache data. These being server-side, client-side & at a CDN.

Often people say that you can’t cache with [**GraphQL**](https://graphql.org/) or that it breaks caching as we know it with REST API’s. Although there are multiple ways to cache data with GraphQL. So, I’ve decided to look at these methods, and give you **my own subjective conclusions**.

To do this research I’ve used a [10 GB SQL database of Stack Overflow](https://www.brentozar.com/archive/2015/10/how-to-download-the-stack-overflow-database-via-bittorrent/). This database was large enough to measure the differences between caching methods and pure select queries. I ran this database in a Docker container using the [azure-sql-edge](https://hub.docker.com/_/microsoft-azure-sql-edge) image.

All the code I’ve written to do this research can be found on my Github on this repo: [https://github.com/OnderbekeNiels/research-project-3mct](https://github.com/OnderbekeNiels/research-project-3mct).

If you have any feedback on this research or Medium post in general, leave a comment below! ✍️

> **Disclaimer:** I’ve tested these caching methods on a **small scale** and will give you my **subjective thoughts**. I did this research on my own and did this in a **non-production environment**. My goal is helping other people to find answers on their questions around the caching possibilities with GraphQL.
>
> The timing results are taken in my local develop environment being on a MacBook PRO (M1 PRO) using [Next.js](https://nextjs.org/).

# Evaluation criteria

## Speed

- How fast can the cached data be fetched? \*
- How big is the impact of the data size?
- How big is the impact of the nesting / relational tree?

## Developer experience

- How easy is it to implement / get started?
- Pros & Cons
- Costs
- Bundle sizes (client-side bundles)
- How can I control cache busting?

> \***Speed:** I’ve measured this in a react component by setting a useState hook of the current time as start time and calculated the time between start and the moment the data was fetched.

# Caching strategies

In the table below you can find the caching strategies I’ve looked at. I know that there are many other libraries in other languages available, but I’ve chosen to do my research in a **JavaScript** environment.

A well-known library I did not test on the client-side is [Relay](https://relay.dev/), due to the learning curve and time I’ve decided to focus more on other clients.

![](https://miro.medium.com/max/1400/1*xg05kufWtNj4K2av3s2XEQ.png)

# Server-side

To build a backend server I’ve used [**Express**](https://expressjs.com/) (node.js web framework) in combination with [TypeGraphQL](https://typegraphql.com/). Express is easy to get started with and lets you use your own structure. On the other hand, [TypeGraphQL](https://typegraphql.com/) is a well-known framework that makes using GraphQL with TypeScript straight forward.

## Response caching with Redis

To handle in-memory response caching on the server-side I’ve used [Redis](https://redis.io/). Redis is super flexible and is easy to set up. An alternative could be [Memcached](https://memcached.org/) for example.

**_Implementation_**

**Redis** is an in-memory database where you can **store Key Value** **pairs** and other data types but, in my case, I’ve just used Key Value storage. Due to the fact Redis is an in-memory database I thought it was the perfect tool to cache responses. I’ve used a Docker image to run a Redis server in a container. The library I’ve used to connect with Redis is called **“**[**ioredis**](https://github.com/luin/ioredis)**”.**

![](https://miro.medium.com/max/700/1*CYKGIw1zx9zCbOst9fMJjQ.png)

At first, I wanted to cache the full response of my queries but then I would work against the principles of GraphQL. Because every query can ask different fields, it would not be a good idea to cache the full response from a query because this is client specific.

My solution to this was to cache only the database response I’ve got returned by TypeORM. This way I returned the same data with the Redis cache as my database would return.

![](https://miro.medium.com/max/952/1*V449nrEjoaCnG0BFvhwLRA.png)![](https://miro.medium.com/max/848/1*KpwO-BOG9bTY2u0sAtrc-g.png)

In the above example you can see how I first try to receive data from the cache, if not present, I will fetch the data from the database. To set a max-age on the Key Value pairs you can just pass a time-to-live in seconds.

This way was very effective and made it also possible to cache field resolvers (non-scalar types).

![](https://miro.medium.com/max/952/1*KMu6EDLgT31ZQauzgrhFlw.png)

**_Evaluation_**

_Speed_

![](https://miro.medium.com/max/700/1*lGiUrdefuZbCcs6kUrK-iw.png)

As you can see in the above table, Redis is almost **50% faster** than using no caching when you are fetching data to the frontend. This percentage is calculated on the average of 4 different queries, that represent a real-world example.

Redis has het most impact when you have large data and deep nested data, because these are expensive jobs for a database. Smaller queries can be easily handled by SQL servers so the impact there is much smaller. But because I’ve stored the full response as a stringified JSON I didn’t have control over the relations.

_Developer experience_

Redis was a real pleasure to implement for response caching, although that’s not the only part that comes to play with caching. Like how do we handle cache busting?

Because Redis does not know about our relations when it stores our JSON’s as a string it’s much harder to control updates on the data. You can for example delete keys that contain a certain id. But what if the id of our updated or deleted entity is not represented in the key?

We would need to store every single entity to invalidate it when there are changes. This is a bit more complex to set up and slows down the backend while computing al these things. In client-side libraries like Apollo this can be done for you, without you needing to make a mechanism yourself.

I would recommend using Redis for large datasets that don’t change often. It will be effective, and you don’t need to worry about invalidation of your cache.

**Pros**

- Easy to implement for responses
- Usable in microservices architecture
- Fast

**Cons**

- Complex when you want to cache normalized data and keep track of their invalidation.

## HTTP Caching

GraphQL breaks HTTP caching because it only uses POST requests, right? Then you haven’t heard about **Automatic persisted queries!**

Automatic persisted queries are basically queries that are sent as a hash. If your server supports this, you can send your query as a GET request with a hash of your query in the query string of the request. You can find how this actually works on this link: [https://www.apollographql.com/docs/apollo-server/performance/apq/](https://www.apollographql.com/docs/apollo-server/performance/apq/) .

A nice thing is that this is not only available with Apollo! Urql client also supports this, so you don’t have to worry about creating this mechanism yourself.

**_Implementation_**

If you don’t exactly know how HTTP caching works, I would recommend watching this video: [https://www.youtube.com/watch?v=HiBDZgTNpXY](https://www.youtube.com/watch?v=HiBDZgTNpXY) to fully understand what is going on. By using **automatic persisted** **queries** with **GET** requests, you can set directives in your HTTP cache-control header.

Your server keeps those hashes in-memory to know which hash represents which query. You have the possibility to use another memory cache like Redis than the default from Apollo to cache those hashes. This caching method enables you to let a browser or CDN cache your API responses. This can be done in 2 certain ways when you are using TypeGraphQL on the server-side.

**_Way 1: Apollo Directives_**

The first way is by using the cacheControl directive from Apollo Server. With this directive you can define a **max-age** and **scope** to a type or a resolver. How those directives are used to calculate the cache-control header can you read on this page: [https://www.apollographql.com/docs/apollo-server/performance/caching/#setting-cache-hints](https://www.apollographql.com/docs/apollo-server/performance/caching/#setting-cache-hints)

Because Apollo Sever only lets you set the max-age and scope (public or private) directives you can’t take fully advantage of HTTP caching. You cannot set the “stale-while-revalidate” directive for example, which is important to handle cache busting. To do so, you need to use way 2.

If you want to place directives on types and fields in your classes with **TypeGraphQL** like I did, you will need to define the directive @Directive(“@cacheControl(maxAge: n)”)

You can see how to do this in the example below.

_Examples:_

In the type (also possible at a field)

![](https://miro.medium.com/max/1172/1*rbsMwTLro0vYU-eFhxF7_A.png)

In a resolver

![](https://miro.medium.com/max/1240/1*5Jb6qejRURN9a7EbMSYQHg.png)

**_Way 2: Set the response header ‘cache-control’ manually._**

Because I’m using Express, I can easily pass the response argument to my resolvers using the GraphQL context. This way I was able to set my own cache-control directives in a resolver. This way is very customizable & handy if you are familiar with HTTP caching!

How I’ve set this up:

index.ts file

![](https://miro.medium.com/max/888/1*fi-1GXNjrhs2aN310rQUdA.png)

resolver

![](https://miro.medium.com/max/1360/1*rcFanJLza1D9e0hedK1vxg.png)

**_Client-side config_**

Besides the directives on the server-side, you will have to enable sending automatic persisted queries as GET requests on the client-side as well. In this example I’ve used Apollo Client, but keep in mind other libraries like Urql also support this.

![](https://miro.medium.com/max/1400/1*RPiiAWG3Pgbv3dXtqi9QMw.png)

**_Evaluation_**

_Speed_

From my own testing, this way of caching seems to be the fastest. Because the cache data is stored locally, your browser has instant access to it. When your browser is using the local cache it’s on average **80% faster** than working without caching! I did not measure this way of caching with a CDN, so this data is only based on the cache from your browser.

![](https://miro.medium.com/max/1400/1*bpNdnTsu-w0cIDKGsQlUTg.png)

![](https://miro.medium.com/max/1400/1*WS4HKnkRlIAF4KOxJclfkg.png)

_Developer experience_

Just like Redis I would recommend using Automated Persisted Queries for large responses that don’t change that often. It’s also handy if you already use a CDN provider for a REST API and you want to add a GraphQL API.

A good basic knowledge of HTTP caching is needed to use the full potential of this caching method. You also must understand how Apollo calculates the max-age and scopes when you are placing directives on types and fields. Like I said there are multiple ways to configure this, because of that, it can be a bit hard to get started with this.

**Pros**

- Easy to implement for responses
- Super-fast
- Usable with a CDN
- Server does not store anything

**Cons**

- Can be hard to set this up with your server
- Good knowledge about HTTP caching is needed

# Client-side

To test caching in a client-side environment I’ve chosen to use [Next.js](https://nextjs.org/) as framework, it’s easy to use and has many libraries that are made for it.

I will mainly talk about Apollo Client in this section. For my research I skipped Relay because it has a longer learning curve, but I did look at [Urql](https://formidable.com/open-source/urql/). Because these Clients are very similar, I will just mention the main differences & won’t make a separate section for Urql. I will compare some parts of Apollo with Urql & share my thoughts on it with you.

You can find a good feature comparison guide on this site from Urql: [https://formidable.com/open-source/urql/docs/comparison/](https://formidable.com/open-source/urql/docs/comparison/)

## Apollo Client v3 — InMemory Cache

Important to know is that Apollo’s client is specifically made for a React environment and it uses hooks to fetch data. If you are familiar with React you will like this library for sure! On the other hand, Urql has implementations for other frameworks as well, like Vue & Svelte. So, if you don’t want to stick to React, [Urql](https://formidable.com/open-source/urql/) is a great option!

Apollo Client delivers out of the box in-memory caching. It’s easy to setup and configure. It uses a normalized cache to work with relational data and that way it can easily modify or merge the cache after a mutation. By letting you set different cache/fetch policies you can describe how every Query or Mutation must behave when it fetches data. How this exactly works can you read on these pages:

https://www.apollographql.com/docs/react/caching/overview/

**_Implementation_**

To get started you just need install the npm package [@apollo/client](https://npm.io/package/@apollo/client). Unlike [Urql](https://formidable.com/open-source/urql/) you don’t need to install “Exchanges” to use the normalized cache, which makes it easier to get started. How to use this in-memory cache is good documented on the site of Apollo. So, I recommend reading the docs before using it!

**_Evaluation_**

_Speed_

Apollo client is the second fastest caching method I’ve tried. Of the measurements I did, it was on average **69% faster** than when using no caching at all. It’s even **10% faster** than using **Urql**, while both were using in-memory normalized caches in the browser.

![](https://miro.medium.com/max/1400/1*bpNdnTsu-w0cIDKGsQlUTg.png)

![](https://miro.medium.com/max/1400/1*V4YMRO76QJ5uvgyey6LcOA.png)

_Developer experience_

In my opinion this is the best solution for GraphQL caching, client-side and even in general. It has great Chrome Dev tools that make debugging your cache super clear.

Due to its customizable in-memory normalized cache you can keep your back- and frontend in sync. After a mutation or query it automatically updates the cache when it receives the right information back (id & updated fields). Then merges the entitie(s) in the cache it already has.

The cache updates only happen automatically when its receiving fields of an entity it did not already have or while updating mutated fields. You must update the cache yourself when deleting or creating entities or when you can’t return the updated fields from an update mutation.

How do we update the cache manually then? You can configure what should happen with the cache in a callback. Read more about it here: [https://www.apollographql.com/docs/react/caching/cache-interaction/](https://www.apollographql.com/docs/react/caching/cache-interaction/)

It’s also possible to configure **polling** or **refetching** for your data in the cache, these are 2 strategies Apollo Client provides you with. With these options you can also keep your cached data in sync with your server! [https://www.apollographql.com/docs/react/data/queries/#updating-cached-query-results](https://www.apollographql.com/docs/react/data/queries/#updating-cached-query-results)

**Pros**

- Fast
- Easy to setup
- Hooks for fetching data
- Great dev tools

**Cons**

- Less flexible as Urql in terms of bringing your own implementation as an Exchange
- Relative big bundle size

_What about the bundle size?_

When we take a look at [Bundlephobia](https://bundlephobia.com/), you will notice that [@apollo/client](https://npm.io/package/@apollo/client) is a bigger package in comparison with [Urql](https://npm.io/package/urql), the other GraphQL client I’ve tested.

![](https://miro.medium.com/max/1400/1*nhoo-rbeENdGUnGxKepIgw.png)

This is a huge difference at first sight. But when I analyzed my 2 apps their bundles with the same functionality but both different clients with [@next/bundle-analyzer](https://npm.io/package/@next/bundle-analyzer). The results were way closer to each other than you would think. The actual size of the bundles doesn’t differ that much.

My take on this is because with Urql, you will need an extra package to make normalized caching work ([@urql/exchange-graphcache](https://npm.io/package/@urql/exchange-graphcache)). Before I analyzed my bundles, I wanted the caches to work the exact the same way so that I could make a valid comparison. Urql caches by default the data as documents and not as normalized entities like Apollo. This means Urql can’t update the cache by default without sending a new request to the server.

![<img alt="" class="ef es eo ex w" src="https://miro.medium.com/max/1400/1*JX34Ke61DbJZr7k_9ny-1Q.png" width="700" height="655" srcSet="https://miro.medium.com/max/552/1*JX34Ke61DbJZr7k_9ny-1Q.png 276w, https://miro.medium.com/max/1104/1*JX34Ke61DbJZr7k_9ny-1Q.png 552w, https://miro.medium.com/max/1280/1*JX34Ke61DbJZr7k_9ny-1Q.png 640w, https://miro.medium.com/max/1400/1*JX34Ke61DbJZr7k_9ny-1Q.png 700w" sizes="700px" role="presentation"/>](https://miro.medium.com/max/1400/1*JX34Ke61DbJZr7k_9ny-1Q.png)

![](https://miro.medium.com/max/804/1*_9aR6QcSnXPsd8ue8jBfOw.png)

Chrome network tab: Apollo

![](https://miro.medium.com/max/900/1*uvcMXAGbUGfIr2-iw5UCOw.png)

Chrome network tab: Urql

If you are new to the GraphQL world I would highly recommend using Apollo client because you will have to think less about how things work together. On the other hand, Urql is great option too and gives you more flexibility. But as you can see in the measurements, Urql is still a bit slower than Apollo.

In the end it’s your choice and I think you will have a great experience with both. You will have to decide which suits the best for your own project.

# CDN (Content Delivery Network)

## GraphCDN

As you already hear in the name, GraphCDN is a CDN build for GraphQL API’s. It embraces the power of GraphQL and makes caching on a CDN level easy and naturally. Just like with automatic persisted queries, some knowledge about **HTTP caching** is needed. GraphCDN sets the cache-control headers like “s-max-age” & “stale-while-revalidate” for you, you just need to declare them in an easy-to-understand form!

![](https://miro.medium.com/max/444/1*VKr9T-ni2LMszUUzuNTjTQ.png)

[https://graphcdn.io/](https://graphcdn.io/)

GraphCDN offers you wide customization for your own caching strategy! You can create caching rules for queries, types & fields. You can define what fields are needed for purging your cache and you can define cache scopes based on headers like “Authorization”.

Analytics, Performance monitoring and error logging are included when you are using GraphCDN, you can track all the traffic that happens to your API. There are nice graphs and summarizations available to track how efficient your cache configuration is!

**_Implementation_**

Setting up GraphCDN for your GraphQL API is easy, they have a great onboarding experience! You just need to fill in the URL of your GraphQL API and that’s it connection wise. After that, you need to define the caching rules you want.

You will get a URL from GraphQL that will handle your API traffic. If you want to, you can set this URL to your own domain name.

Before using GraphCDN I would recommend watching this video: [https://www.youtube.com/watch?v=EjrJtp4JaGQ&t=1061s](https://www.youtube.com/watch?v=EjrJtp4JaGQ&t=1061s) . In this video the co-founder of GraphCDN (Max Stoiber) talks about how it works & why it exists!

**_Evaluation_**

_Speed_

From a speed standpoint GraphCDN is the slowest caching method that I’ve tested in my research. But it was still **42% faster** than using no caching at all. But speed is not everything when it comes to caching.

CDN’s are built to deliver data as close to the consumers as possible. CDN’s are also solving the problem of your server having to deal with many requests at one time. They try to make the load on your server as low as possible by providing cached data. So, if you have a huge user base that’s divided around the world, GraphCDN is the way to go! Something to take in mind is that I tested this in my local environment, so that’s not the environment GraphCDN will show its strengths.

![](https://miro.medium.com/max/1400/1*bpNdnTsu-w0cIDKGsQlUTg.png)

![](https://miro.medium.com/max/1400/1*dmYAhuXlol5iVwLCrci-Bg.png)

_Developer experience_

GraphCDN provides a CLI so you can proxy requests to your local develop environment for testing your backend with your GraphCDN configuration.

GraphCDN is free up to 5 million requests per month. So, if you want to use GraphCDN in your production environment, the best option is to define your needs & compare them with the plans GraphCDN provides.

To implement cache busting / invalidate cached data after mutations, GraphCDN provides auto updates to renew your cache when an update mutation happens (if the updated data is included)! This is just like how Apollo Client handles updates. [https://docs.graphcdn.io/docs/automatic-cache-invalidation-via-mutations](https://docs.graphcdn.io/docs/automatic-cache-invalidation-via-mutations) .

They also provide a Purging API [https://docs.graphcdn.io/docs/purging-api](https://docs.graphcdn.io/docs/purging-api) . This is a secured API that automatically gets created for your schema. It enables you to send requests from your backend server to purge caches that contain a certain Type, Field or Operation name. This is super handy to keep your data fresh in the cache! I would consider this a huge plus because you can relatively easy keep control over your caching strategy! Important to mention, because this is a public API that clears your real cache, you can’t really test it with your development setup using the CLI.

Nice to know, GraphCDN also support Automatic Persisted Queries!

**Pros**

- Super easy to get started
- Effects many users
- Develop environment with CLI
- Can be combined with HTTP caching headers that are set on the server-side
- Auto invalidation
- Purging API
- Build-in analytics, performance logging & error logging

**Cons**

- Paying after 5 million requests
- Analytics website sometimes a bit buggy

# Results and conclusions

## Measurements

![](https://miro.medium.com/max/1400/1*3NA_gdBKPIwkJXyUoZ2Iwg.png)![](https://miro.medium.com/max/1400/1*8-WNO_O2xLBn8kEN5htX7Q.png)

## Recommendations

I’ve ranked the different caching methods from 1 to 5 on the criteria that are important to me. 5 being the “best” and 1 being the “worst”.

**_Overall_**

![](https://miro.medium.com/max/1400/1*5Fgx6yYdhH0Obdrn-e0ERw.png)

**Client-side**

![](https://miro.medium.com/max/1400/1*RIJ0NCkcUsXxh16bzAJmfA.png)

**Server-side**

![](https://miro.medium.com/max/1400/1*QM50EHf14mxsDOAEPrZ54Q.png)

**CDN**

![](https://miro.medium.com/max/1400/1*c0OeduvVicPaKVyC4BNj7w.png)

**Final verdict**

As you can see, there is no straight answer to the main question. Which caching strategy you need to use will depend on the data size, relations, how many times data changes in a certain period, how many users, etc…. I hope that my research could help you with finding the right strategy!
