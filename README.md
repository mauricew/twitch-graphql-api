Twitch GraphQL API
======================

When Twitch updated its frontend in 2017, it secretly showed signs of a new method of accessing the backend.
For the majority of the site, instead of using the kraken (v5 and below) or even the helix ("new") REST API endpoints, it uses a completely separate API utilizing GraphQL.

GraphQL is very flexible since you can control exactly what fields are returned, and it can be linked to associations (e.g. Game -> Stream -> User) without having to make multiple calls. This document will be sparse and only show examples, and you may want to use resources online to find out more about querying GraphQL APIs.

## IMPORTANT DISCLAIMER
Please note that this API is currently not officialy sanctioned to be used by end consumers. Please refer to the [official Twitch Developers site](https://dev.twitch.tv) to see documentation on recommended uses of the API.

This documentation and its maintainer has no affiliation or connection to Twitch Interactive, Inc., who owns the aformentioned API and its data.

## Making requests
The single endpoint for accessing the GraphQL API is **https://gql.twitch.tv/gql**, proxied through Fastly (also https://api.twitch.tv/gql, and the entire subdomain is proxied through Akamai). Requests are made using POST.
From the day the updated Twitch site went out of beta, it has required a *Client-ID* or a bearer token in the *Authorization* header, so use it at your own risk.

The GraphQL schema is public and can be accessed via introspection. To test out the API, I recommend [Insomnia](https://insomnia.rest) which has first-class support and autocomplete for GraphQL.

## Responses
All responses will have a status code of 200, even when there are errors. The body will be a either a JSON object if one call is made, or array if several are.

* **data** an object containing the data requested,
* **errors** an array if there was an error with the request,
* **extensions** an object containing metadata about the request. Always includes time taken. If a custom query is passed, it will have the original operation name, and any variables passed. (Custom queries are outside the scope of this document)

## Helix unimplemented examples

### User
#### Get subscriptions and emotes
````
query {	
  user(login: "day9tv") {
    subscriptionProducts {
      id
      name
      displayName
      price
      tier
      hasAdFree
      hasFastChat
      hasSubOnlyChat
      hasSubonlyVideoArchive
      interval {
        duration
        unit
      }
      emoteSetID
      emotes {
        id
        state
        text
        token
      }
    }
  }
}
````
#### Get cheer badges and cheermotes
**ProTip** Badges can actually go up to 5 million!
````
query {
  user(login: "overwatchleague") {
    cheer {
      availableBadges {
        title
        description
        clickURL
        imageURL
      }
      emotes {
        id
        prefix
        type
        tiers {
          bits
          images {
            id
            theme
            isAnimated
            dpiScale
            url
          }
        }
      }
    }
  }
}
````
#### View chat mods
````
query {
  user(login: "forsen") {
     mods {
      edges {
        node {
          login
        }
      }
    }
  }
}
````
### Games/Directory listing
#### List top games *and* their top streamers
You can also do communities or creative-specific communitiies.
````
query {
  directories(filterBy: GAMES) {
    edges {
      node {
        id
        name
        broadcastersCount
        viewersCount
        streams {
          edges {
            node {
              broadcaster {
                login
              }
              viewersCount
            }
          }
        }
      }
    }
  }
}
````


## Other common Tasks

### Game
#### Clips and videos
You can get current viewers, total followers, and top clips/videos.

````
query { 
  game(name: "IRL") {
    name
    followersCount
    viewersCount
    clips(criteria: { period: LAST_MONTH }) {
      edges {
        node {
          id
          title
          viewCount
          createdAt
          durationSeconds
          curator {
            login
          }
          broadcaster {
            login
          }
        }
      }
    }
    videos(sort: VIEWS) {
      edges {
        node {
          id
          creator {
            login
          }
          title
          viewCount
          createdAt
          lengthSeconds
          broadcastType
        }
      }
    }
  }
}
````
### User
#### Getting a single user

**Request body**
````
query {
  user(login: "thijshs") {
    id
    login
    displayName
    description
    createdAt
    roles {
      isPartner
    }
    stream {
      id
      title
      type
      viewersCount
      createdAt
      game {
        name				
      }
    }
  }
}
````
**Response body data**
````
"user": {
  "id": "57025612",
  "login": "thijshs",
  "displayName": "ThijsHS",
  "description": "Hello, i am Thijs,21 years old and a gamer from The Netherlands. I am a professional Hearthstone player. I'm currently streaming 4-5 days a week. I hope you have fun with my stream!",
  "createdAt": "2014-02-17T20:50:47.402453Z",
  "roles": {
    "isPartner": true
  },
  "stream": {
    "id": "26320132304",
    "title": "Thijs - Don Han'Cho OTK Warrior!",
    "type": "live",
    "viewersCount": 14917,
    "createdAt": "2017-09-23T07:55:26Z",
    "game": {
      "name": "Hearthstone"
    }
  }
}
````
#### Find out who follows a user
Paginated data utilizes the [Relay](https://facebook.github.io/relay) cursor spec, see https://facebook.github.io/relay/graphql/connections.htm.  
Use a query parameter to get the first *n* (in this case, 25) followers in the list. Default is always 10.

**Request body**
````
query {
  user(login: "imaqtpie") {
    followers(first: 25) {
      totalCount
      edges {
        followedAt
        node {
          displayName
        }
      }
    }
  }
}
````
**Response body data**
````
"user": {
  "followers": {
    "totalCount": 1904355,
    "edges": [
      {
        "followedAt": "2011-09-25T02:38:03Z",
        "node": {
          "displayName": "diumlol"
        }
      },
      {
        "followedAt": "2011-09-29T20:59:25Z",
        "node": {
          "displayName": "toxichail"
        }
      },
      {
        "followedAt": "2011-10-07T01:02:10Z",
        "node": {
          "displayName": "Shad0wSlyth"
        }
      },
      {
        "followedAt": "2011-12-08T22:36:10Z",
        "node": {
          "displayName": "TSjoolder"
        }
      },
      ...
    ]
  }
}
````
#### Find out who a user follows
Same as above, except query *follows* instead of *followers*.

### Stream
#### Getting top live streams

**Request body**
````
query {
  streams(first: 25) {
    edges {
      node {
        id
        title
        viewersCount
        broadcaster {
          displayName
        }
        game {
          name
        }
      }
    }		
  }
}
````
**Response body data**
````
"streams": {
  "edges": [
    {
      "node": {
        "id": "26321639056",
        "title": "[RU] Virtus.Pro -vs- Na'Vi | ESL One Hamburg 2017 | CIS | Adekvat & Smile",
        "viewersCount": 66940,
        "broadcaster": {
          "displayName": "esl_ruhub_dota2"
        },
        "game": {
          "name": "Dota 2"
        }
      }
    },
    {
      "node": {
        "id": "26320948464",
        "title": "Mythic Dungeon Invitational - EU Group Stage",
        "viewersCount": 36023,
        "broadcaster": {
          "displayName": "Warcraft"
        },
        "game": {
          "name": "World of Warcraft"
        }
      }
    },
    ...
  ]
}
````
### Game
#### Game metadata and top live streams
**Request body**  
(this shows that querying can be case insensitive)
````
query {
  game(name: "PlayerUnknown's Battlegrounds") {
    id
    name
    streams(first: 50) {
      edges {
        node {
          id
          title
          viewersCount
          broadcaster {
            id
            displayName
          }
        }
      }
    }
  }
}
````
**Response body data**
````
"game": {
  "id": "493057",
  "name": "PLAYERUNKNOWN'S BATTLEGROUNDS",
  "streams": {
    "edges": [
      {
        "node": {
          "id": "26321086528",
          "title": "nice title Tim thanks guys",
          "viewersCount": 17600,
          "broadcaster": {
            "id": "36769016",
            "displayName": "TimTheTatman"
          }
        }
      },
      {
        "node": {
          "id": "26322034224",
          "title": "🔴90% ( ͝° ͜ʖ͡°) [http://pubg-roll.ru/ - Рулетка PUBG]",
          "viewersCount": 6778,
          "broadcaster": {
            "id": "27115707",
            "displayName": "Arthas"
          }
        }
      }
    ]
  }
}
````
