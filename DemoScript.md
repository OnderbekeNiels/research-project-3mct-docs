# Demo 1 feb 2O22

## Caching strategies i will demo

1. Server-side with HTTP caching & Automatic persisted queries
2. Client-side with Apollo Client v3
3. CDN with GraphCDN

## Effect of the demo

**Goal:** show how the caching works by the 3 best caching methods

**Scenario for the 3 strategies**

1. Get all posts
2. Show request visualizer
3. Show how it's not cached
4. Show second load (that should show that it's cached)
5. Show where & how it's cached
6. Show cache busting if needed at a update of a post

### Server-side with HTTP caching & Automatic persisted queries

#### How show cache works?

- Open network tab
- Show from disk indicator & response headers
- Show directives in types & resolver

### Client-side with Apollo Client v3

#### How show cache works?

- Show fetchPolicies in for ex postId page
- Don't click refresh bcs in memory, click other page and return to show impact
- Open Apollo Dev tools and show cache
- Open Network tab & show that there is not request send

### CDN with GraphCDN

#### How show cache works?

- Show caching rules & purge api ? **Do i have to make the api public to show working of purge api or is the auto update enough?**
- Open network tab and show s-max-age & stale-while-revalidate headers
