---
name: designing-graphql-schemas
description: Designs GraphQL schema additions — types, queries, mutations, pagination, errors — following Relay conventions and Apollo's operation best practices, so the shape works for the clients that will consume it. Use when adding or changing anything in a GraphQL schema, writing an SDL sketch for frontend or mobile, reviewing a proposed query or mutation, or when the user says "definamos el schema", "qué shape tiene la mutation", "add a GraphQL endpoint", or asks what an API should look like before it is built.
---

# Designing GraphQL schemas

A schema is a contract with clients that cannot be quietly withdrawn. The shape
decided in an afternoon is the shape web and mobile build against for years, so
the goal is a surface that is obvious to consume, cheap to cache, and able to
grow without breaking anyone.

Write the SDL before the resolvers, and hand it to the client developers as the
deliverable it is.

## Evolve additively

Treat every published field as permanent. Add fields and arguments; never remove
one, rename it, or change its type or nullability in place. When something must
change shape, add the new field beside the old one and deprecate the old with
`@deprecated(reason: "...")` pointing at the replacement.

Nullability is part of the contract: making a nullable field non-null is a
breaking change for no one, but the reverse breaks every client. Start strict
where the server can guarantee a value, and nullable where it genuinely cannot.

## Mutations

**One input object, Relay style.** Every mutation takes a single `input`
argument and returns a payload type. Carry `clientMutationId` through both.

```graphql
input ShareMapsWithTeamInput { mapIds: [ID!]!, clientMutationId: String }
type ShareMapsWithTeamPayload {
  maps: [Map!]!
  userErrors: [ShareUserError!]!
  clientMutationId: String
}
```

**Name the action, don't expose a setter.** `shareMapsWithTeam` and
`unshareMapsFromTeam`, not `setMapsShared(shared: Boolean!)`. Named actions read
better in logs, analytics and client code, and they leave room for the two sides
to diverge later.

**Accept a list from day one.** A mutation that acts on one entity today gets a
bulk sibling eventually; `ids: [ID!]!` covers the single case as a list of one
and avoids shipping two endpoints for the same operation.

**Return the entities you changed.** The payload carries the updated objects so
the client's cache updates itself without a refetch. A payload of `{ ok: true }`
forces every client to re-query and guess what moved.

**Decide all-or-nothing versus partial, and say which.** For a list input, state
whether one bad element fails the whole call. Then make the payload express it:
all-or-nothing returns no entities and a populated error list; partial returns
what succeeded alongside the failures.

**Expected failures are data, not exceptions.** Business outcomes — not the
owner, not allowed, already done, wrong state — belong in a `userErrors` field
with a machine-readable `code` enum and the id of the element that failed.
Reserve top-level GraphQL errors for things the client cannot act on. Codes let
clients localize; messages alone force string matching.

**Long operations return a job handle**, not a timeout. Pair it with a query for
the result.

## Types

**Let the server decide what the viewer may do.** Expose capability fields —
`viewerCanEdit`, `viewerCanShare`, `viewerCanDelete` — instead of making clients
compare owner ids and re-derive the rules. Without them, every client
reimplements the authorization logic and they drift apart the first time a rule
changes.

**Model what the domain means, not what the table stores.** Column names,
foreign keys and nullable legacy fields are implementation; the schema is the
place to leave them behind.

**Share one type per entity.** If two screens show the same thing, they get the
same type with different fields selected — never a parallel `SearchResultFoo`.
A forked type silently loses whatever other services or future extensions add to
the original.

**Be deliberate about global ids.** The Relay Node spec wants opaque, globally
unique ids. Adopting it is a decision with consequences for federation keys and
for clients; so is declining it. Either way, write the decision down rather than
letting it happen.

## Lists and pagination

**Any list that can grow gets a connection**, with `edges`/`node`, `pageInfo`
and opaque cursors. Lists that are bounded by the domain (a handful of statuses,
the nodes of one document) can stay plain arrays — say which and why.

**Cursor, not offset.** Offsets skip or repeat items when the underlying data
changes between pages, and get slower the deeper you go.

**Order by a stable, deterministic key**, tie-broken by id. An ambiguous order
makes cursors skip rows at page boundaries — a bug that only shows up with real
data and is miserable to diagnose.

**Set a default and a maximum page size in the schema**, so no client can ask
for everything by accident.

**Filters are arguments, not separate fields.** One `items(filter: …)` beats
`sharedItems`, `recentItems` and `myItems` as three fields that diverge.

**Watch what a field costs to resolve.** A field that triggers a query per
parent needs batching; a field backed by a huge column needs an explicit
projection so a page of results doesn't load megabytes nobody asked for.

## Operations, for the clients consuming it

These come from Apollo's operation best practices and are worth stating when
handing a schema over, because they change what the schema needs to support:

- **Every operation gets a name.** Anonymous operations lose per-operation
  metrics and break when documents are combined.
- **Arguments go through variables**, never interpolated into the query string.
  Hardcoded arguments wreck cache reuse, leak values into query strings, and
  defeat persisted queries.
- **Each component asks for exactly the fields it renders.** Over-fetching costs
  network and hurts cache reuse.
- **Fragments distribute a query across the components that render it**, so each
  one declares its own data needs.
- **Global data and viewer-specific data belong in separate operations**, so the
  shared part stays cacheable.
- **List fields get paginated** rather than fetched whole.

## Guardrails

- Never remove, rename or retype a published field. Deprecate and add.
- Never return a bare `Boolean` from a mutation that changed an entity.
- Never put an authorization rule in the client's hands when a capability field
  can carry it.
- Don't invent a second type for an entity that already exists in the schema.
- Don't add a field whose semantics are still undecided — an unimplemented field
  in a published schema is worse than a later additive change.

## Before finishing

- [ ] Mutations take one `input` and return a payload with the changed entities
- [ ] Expected failures are `userErrors` with a code enum, not thrown errors
- [ ] All-or-nothing versus partial is stated for every list operation
- [ ] Growable lists are connections with cursors, a default and a max page size
- [ ] Ordering is deterministic and tie-broken
- [ ] Capability fields cover every rule a client would otherwise re-derive
- [ ] Nothing published is removed, renamed or retyped
- [ ] The SDL was handed to the client developers before the resolvers were built
