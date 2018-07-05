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
- search autocomplete / suggestions
- orgchart

#### Scenarios

##### Viewing a Profile

- user visits https://mozillians.org/en-US/u/fiji/
- hits django
- gets view with new front-end
- new front-end requests profile data from API (django)
- retrieving data from [search service](#search-service) and ([orgchart service](#orgchart-service))
- repsonds according to permission level
- new front-end renders profile view

##### Editing a Profile

- user is on the edit profile view
- user clicks on ✎ in a section of their profile
- hits django
- gets view with new front-end
- new front-end requests edit profile section data from API (django)
- new front-end renders edit profile section view
- user changes value
- user clicks on save button
- changed fields get sent to API
- resulting partial profile update gets sent to [profile publisher](#profile-publisher)
- on success user gets redirected to edit profile view

##### Searching

- user enters something in the searchbar on any page
- while typing the new front-end queries the suggestion api for suggestions
- django queries search service according to users permissions for suggestions
- front-end displays suggestions
- user hits enter / clicks on search / clicks on a suggested search
- font-end sends search query to search API
- django queries search service according to users permissions
- new-front end renders search results and updates the searchbar to reflect the proper search syntax
