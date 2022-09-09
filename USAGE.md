## Examples
Due to deprecated features and Helix gaining more functionality, updates to this page will be sparse.

Paginated data utilizes the [Relay](https://facebook.github.io/relay) cursor spec, see https://facebook.github.io/relay/graphql/connections.htm.  
Use a query parameter to get the first *n* entries in any list. Default is always 10.


### Streams
#### Getting top live streams

**Request body**
````graphql
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
````json
{
  "streams": {
    "edges": [
      {
        "node": {
          "id": "40182652665",
          "title": "🐑CLICK NOW🐑DRAMA🐑NEWS🐑TOURNAMENT🐑PROBABLY VR🐑MASSIVE🐑MEGA JUICE🐑REACT LORD🐑GAMEPLAY GOD🐑GOLEM #1🐑WOW🐑INSANE🐑WOW🐑WOW🐑",
          "viewersCount": 52271,
          "broadcaster": {
            "displayName": "xQc"
          },
          "game": {
            "name": "Just Chatting"
          }
        }
      },
      {
        "node": {
          "id": "47116009709",
          "title": "🟦BEST BASKETBALL PLAYER IN THE WORLD🟦CLICK HERE NOW🟦#1 GAMER🟦MAFIA OVER EVERYTHING🟦",
          "viewersCount": 46716,
          "broadcaster": {
            "displayName": "KaiCenat"
          },
          "game": {
            "name": "Just Chatting"
          }
        }
      }
    ]
  }
}
````

### Game
#### Game metadata and top live streams

````graphql
query {
  game(name: "Teamfight Tactics") {
    id
    name
    streams(first: 50) {
      edges {
        node {
          id          
          viewersCount
          broadcaster {
            id
            displayName
            broadcastSettings {
              title
            }
          }
        }
      }
    }
  }
}

#### Clips and videos
You can get current top clips and videos for a game, all at once!. 
````graphql
query { 
  game(name: "Multiversus") {
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

### Users
#### Getting a single user

*Requires two calls for Helix!*
Stream will be null if the user isn't streaming.

````graphql
query {
  user(login: "Jerma985") {
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

#### Find out who follows a user
````graphql
query {
  user(login: "Amouranth") {
    followers(first: 25) {
      totalCount
      edges {
        followedAt
        node {
          login
        }
      }
    }
  }
}
````

#### Find out who a user follows
Same as above, except query *follows* instead of *followers*.

### User
#### Get subscription levels and emotes
*Not available in Helix!*
````graphql
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
````graphql
query {
  user(login: "rocketleague") {
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
*Implemented June 2019 in Helix, but only with OAuth `moderator:read` permissions*
````graphql
query {
  user(login: "HasanAbi") {
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
#### Find out some information about the stream video (if they're streaming)
*Not available in Helix!*
````graphql
query {
  user(login: "QTCinderella") {
    stream {
      averageFPS
      bitrate
      broadcasterSoftware
      codec
      height
      width
    }
  }
}
````
