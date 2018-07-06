# Mozillians.org Repositioning Architecture

* [Mozillians.org Services](#mozilliansorg-services)
  + [Technologies (Common)](#technologies-common)
  + [New Front-End](#new-front-end)
  + [Mozillians.org](#mozilliansorg)
  + [Search Service](#search-service)
  + [Profile Publisher](#profile-publisher)
  + [Orgchart Service](#orgchart-service)
* [Integrity Checker](#integrity-checker)
* [CIS requirements](#cis-requirements)
  + [Profile Updates from Mozillians](#profile-updates-from-mozillians)
  + [Profile Updates from external Sources](#profile-updates-from-external-sources)


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

#### Profile Updates

The search service will have an endpoint to receive profile updates. This endpoint will insert or
update the profile it received.

#### Privacy Levels

The search service will be responsible of filtering fields according to privacy settings. We have
two proposals on how we can achieve this.

1. We use one index per privacy level. This enables simple search but we need to do the filtering on
   every insertion of data.
2. We use a single index and include privacy filters in the search query and do filter fields in the
   result set. This enables simple insertion but requires more work during search time.


### Profile Publisher

The profile publisher is the central broker for dealing with profile updates. Its main
responsibilities are:

- receive user created updates from Mozillians and send them to CIS
  - the request/respond cycle will wait until the profile was successfully updated in CIS or an
    error/timeout occurred
- receive update events from CIS and send profile updates to [search service](#search-service)
  and [orgchart service](#orgchart-service)

The profile publisher will be implemented in an async mindset. Since it's mainly IO bound this
will allow us to scale vertically for a long time.


### Orgchart Service

The orgchart service will serve any information necessary to render (parts of) the organizational
structure of Mozilla.

It's to be determined what technology will back this. We will focus on having a stable API.

It exposes the following data profile v2 properties:
- `user_id`
- `first_name`
- `last_name`
- `business_title`
- `team`
- `company`
- `office_location`


## Integrity Checker

In order to verify the integrity of the profile data in [search service](#search-service) and
[orgchart service](#orgchart-service) we need a service that does either:

- continuously checks random profiles
- periodically checks all profiles

This can be done by having a separate endpoint that provides a signature / checksum of profiles.

If any integrity errors occur at any time this should raise an alarm. In order to resolve the issue
we re-populate all data in the downstream services.


## CIS requirements

### Profile Updates from Mozillians

Currently this is done via [CIS](https://github.com/mozilla-iam/cis) as a library in Mozillians.
We like to change this to one of the following options:

1. Have a separate service and an endpoint to publish profile updates.
2. Have a lightweight async client for this.
3. Validate a sign within [profile publisher](#profile-publisher) and invoke the lambda directly.

#### Requirements

Since we agreed on treating CIS / the Identity Vault as our single source of truth we need to know
whether the profile update was successful. To achieve this we need to get a Kinesis sequence number
as a response and an endpoint to poll for a status of the update. Alternatively, we provide a web
hook that is invoked and sends the status. To achieve a fluent UX this should happen within the
hundreds of milliseconds.


### Profile Updates from external Sources

Whenever a profile update from HRIS or LDAP or anything else happens CIS is responsible to invoke a
web hook.
