# Mozillians.org Repositioning Architechture

## Mozillians.org Services

### Technologies (Common)

#### GraphQL

We stated a discussion around wether we want to test [GraphQL](https://graphql.org/) as our API
query language. Could be a great fit and we plan to give it a shot.

#### Node, Python, Rust

We want to use whatever technologie fits our needs. If it makes sense to write a small service in
in Node or Rust we encourage it.


### New Front-End

A new front-end to render the new pages:

- profile view
- (edit profile section)
  TBD if we do this _inline_ or as separate page
- search result page
- org chart

#### Technologies

- We plan to implement the new front-end using [Vue](https://vuejs.org).
- We use components throughout accross all pages
- We plan to use global CSS instead of using components because we embrace the
  [cascade](https://developer.mozilla.org/en-US/docs/Web/CSS/Cascade)
- Server side rendering is put on hold for now

#### Questions

Will we support IE11?

#### Soft Goals

- Encourage community engagement by having a simple leightweight dev setup
  - Ideally we have mocks for all APIs
- Have a non-JS fallback.
  - Either server side rendering or
  - Super basic html fallback


### Mozillians.org

Is still in charge of:

- Authentication
- Authorization
- Spam cron job
- Group managment
  - Membership will live in profile v2 data though
- Business logic (vouching, access groups, …)
- Signup?

New things:
- API for the new front-end
  - GraphQL?!
- Integration off all other new services

#### Flows

##### Signup
