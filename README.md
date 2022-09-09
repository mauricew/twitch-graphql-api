Twitch GraphQL API
======================

## History
In 2017, Twitch was working on modernizing their technology stack, which included moving the frontend web site from EmberJS to React. This new site (code named "Twilight") was available as a public beta for a few months, and observations in that time showed that backend API access was being revamped as well.  
Previously, the same or similar Kraken (v5 and below) endpoints were used, and even though the beginnings of Helix (the "New Twitch API") were being made available to developers, this new site jumped straight over these and went for a brand new technology: GraphQL.

The engineering team at Twitch has discussed this a few times at developer events:

* [Twitch's GraphQL Transformation](https://eventil.com/talks/xDSQ3z-tony-ghita-twitch-s-graphql-transformation/)
* [Under the Hood of the New Twitch API - TwitchCon Developer Day 2017](https://www.slideshare.net/twitchdev/under-the-hood-of-the-new-twitch-api-twitchcon-developer-day-2017) (mostly a discussion of Helix but has some GQL references)

## Disclaimer
Please note that this API is currently not officialy sanctioned to be used directly by end consumers. Please refer to the [official Twitch Developers site](https://dev.twitch.tv) to see documentation on recommended uses of the API.  
This documentation and its maintainer has no affiliation or connection to Twitch Interactive, Inc., who owns the aformentioned API and its data.

## Making requests
Requests are made as a POST in JSON format to **https://gql.twitch.tv/gql**.  
~~You may also note that https://api.twitch.tv/gql works the exact same way~~, although it was taken down some time in 2021.

### Authorization
From the day the updated Twitch site went out of beta, it has required a *Client-ID* or a bearer token in the *Authorization* header.  
This being an unsupported API means that 1) You are using this at your own risk, and 2) Twitch can detect custom developer token usage tied back to your account. However, you can use the Client ID of a logged out user just fine.  

### Responses
Responses will always have a status code of 200, even when there are errors. The body will be a either a JSON object if one call is made, or array if several are.

* **data** an object containing the data requested,
* **errors** an array if there was an error with the request,
* **extensions** an object containing metadata about the request. Always includes time taken. If a custom query is passed, it will have the original operation name, and any variables passed.  

Even though custom queries are outside of the scope of this document, they make up a large amount of unique functionality for rendering specific views in Twilight.

## Usage
[**API usage has moved to its own page.**](USAGE.md)

## `/integrity`
A new development has just come onto the scene as of September 2022 - the integrity endpoint. It takes in an empty-body POST request that requires at least a Client ID, and outputs a custom-signed JWT with the following metadata fields:
* Client IP address that made the request
* A random device ID
* Your user ID, if an Authorization header was passed
* A boolean field called "is_bad_bot"

Not much is known about what this token is used for, or what "bot" means in this context. The output token's value is passed to the gql calls with the header name `Client-Integrity`.

## Consequences
At some points in time, there were access restrictions added to make it harder for third party developers to use.

### 2018
Sometime towards the end of 2018, there were limitations placed on the API:
* GraphQL schema introspection no longer worked without Client-ID or Authorization header.
* Client IDs were restricted, so almost any token created via the Glass dev management portal will probably no longer work.

According to a few tweets, it was made known via private channels that the use of this API when tied back to you may lead to consequences.

### 2021
Possibly due to the "brand safety score" debacle, it appears that schema introspection has been disabled, as it simply returns a zero byte body.

