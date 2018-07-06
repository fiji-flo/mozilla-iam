# Mozillians.org Repositioning Architecture

## Mozillians.org Services

### Technologies (Common)

#### GraphQL

We stated a discussion around whether we want to test [GraphQL](https://graphql.org/) as our API
query language. Could be a great fit and we plan to give it a shot.

#### Node, Python, Rust

We want to use whatever technology fits our needs. If it makes sense to write a small service in
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
- We use components throughout across all pages
- We plan to use global CSS instead of using components because we embrace the
  [cascade](https://developer.mozilla.org/en-US/docs/Web/CSS/Cascade)
- Server side rendering is put on hold for now

#### Questions

Will we support IE11?

#### Soft Goals

- Encourage community engagement by having a simple lightweight dev setup
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

New things:
- API for the new front-end
  - GraphQL?!
- Integration off all other new services

#### Views

This will outline all changes to views. Which parts come from Django and which parts come from the
new front-end.

#### Group Management

How we'll handle groups in phase 1.

#### New APIs

- single profile view
- profile (section) edit
- search
- search auto-complete / suggestions
- orgchart

#### Scenarios

##### Viewing a Profile

- user visits https://mozillians.org/en-US/u/fiji/
- hits Django
- gets view with new front-end
- new front-end requests profile data from API (Django)
- retrieving data from [search service](#search-service) and ([orgchart service](#orgchart-service))
- responds according to permission level
- new front-end renders profile view

##### Editing a Profile

- user is on the edit profile view
- user clicks on ✎ in a section of their profile
- hits Django
- gets view with new front-end
- new front-end requests edit profile section data from API (Django)
- new front-end renders edit profile section view
- user changes value
- user clicks on save button
- changed fields get sent to API
- resulting partial profile update gets sent to [profile publisher](#profile-publisher)
- on success user gets redirected to edit profile view

##### Searching

- user enters something in the search bar on any page
- while typing the new front-end queries the suggestion API for suggestions
- Django queries search service according to users permissions for suggestions
- front-end displays suggestions
- user hits enter / clicks on search / clicks on a suggested search
- font-end sends search query to search API
- Django queries search service according to users permissions
- new-front end renders search results and updates the search bar to reflect the proper search syntax

##### …


### Search Service

The search service will be built on top of elastic search. It should provide the following APIs:

- query for an individual profile by username / user id and return the according profile data
- a search that exposes most of ES syntax that returns results in one of two formats:
  - list of usernames / user ids
  - list of minimal profile data (everything needed to render a card)
- suggestion / auto-complete for search

The search service will only be accessible by Mozillians, profile publisher and integrity checker.
Ideally we store raw profile v2 data and just add some prefixed properties if necessary.

#### Privacy Levels

The search service will be responsible of filtering fields according to privacy settings. We have
two proposals on how we can achieve this.

1. We use one index per privacy level. This enables simple search but we need to do the filtering on
   every insertion of data.
2. We use a single index and include privacy filters in the search query and do filter fields in the
   result set. This enables simple insertion but requires more work during search time.
