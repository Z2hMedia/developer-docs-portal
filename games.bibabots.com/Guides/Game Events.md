Game Events
======

All Biba games emit events to PubSub on the "games" topic. The events are all
defined in [games.proto](games.proto). 

## Sending Events

**Only the games should be sending events to PubSub!**

### Google Cloud Function Proxy

The URL for the proxy is `https://games.bibabots.com/v1/events`. The JSON to
send for each type of event is documented later in this README file.

## Attributes

This is a mostly-comprehensive list of all attributes that should be sent with
every event:

* `env` - if the game was compiled as a developer version ('dev'), or is a
  production game out in the wild ('prod')
* `app` - the name of the application ( ie, 'biba-collection', 'biba-dhx-2d' )
* `game` - short name of the game ( ie, 'drive', 'collectobots', etc)
* `userId` - Firebase user id
* `deviceId` - Unique device id ( aka, `udid` )
* `deviceModel` - Model of the device ( 'iPhone 8', 'Android', etc -- should be
  provided by Unity )
* `deviceLanguage` - Name of the language the game is currently set to
* `os` - OS of the device -- provided by Unity
* `timestamp` - (RFC 3339 formatted timestamp)[https://tools.ietf.org/html/rfc3339] -- your language should be
  able to generate these without any issue
* `version` - version of the game that generated this event
* `metaType` - the games meta type (explained below)

Each event also has a `type`, which specifies what kind of event it is. The
values for `type` are documented below along with each event type.

### Meta Type

In order to allow the partitioning of events from games for projects like "Biba
For Schools", we have the `metaType` field.

The default value is `biba`.

Currently, the only application which has a different value for this field are
the games included in the "Biba For Schools" bundle.

### Location

If the user has allowed us to use their device location, the following attribute should also be set:

When sending via PubSub directly, the location must be set as an attribute, like
so:

* `location` - a JSON object, with two fields: `latitude` & `longitude`

Example:
```json
"location": {
  "latitude": 49.13234,
  "longitude": -121.203423
},
```

### Duration Format

When sending the duration of a session ( or any event that tracks duration, such
as mini games or equipment use ), there are two options for how to send the
information:

* in milliseconds
* duration string format

If the value in `sessionLength` is just a number, it will be parsed as a
duration in milliseconds.

Otherwise, you can pass a duration string. A duration string is a possibly
signed sequence of decimal numbers, each with optional fraction and a unit
suffix, such as "300ms", "-1.5h" or "2h45m". Valid time units are "ns", "us" (or
"µs"), "ms", "s", "m", "h" ( for nanoseconds, microseconds, milliseconds,
seconds, minutes, hours ).

## Data

The data sent depends on the type of event. Below is the current list of events,
along with their value for the `type` attribute, and what they should send in `data**.

### Activity

When the application has collected LMV data, it is sent using this event.

**Example Data:**
```json
{                                                   
  "attributes": {                                   
    "type": "activity",
    "timestamp": "2019-02-20T10:43:54.585632648-08:00",
    "env": "production",                            
    "game": "biba-copsnrobbers",                    
    "app": "biba-collection",                       
    "userId": "1b91f32d-74d1-4ac8-8a4a-34953b5577bb",
    "deviceId": "a58f1510-1e26-4b83-80d8-b35cc2c971eb",
    "deviceModel": "Pixel 2 XL",                    
    "deviceLanguage": "en_us",                      
    "os": "Android 9 / API 28",                     
    "version": "3.25.342",                          
    "location": {                                   
      "latitude": 49.13234,
      "longitude": -121.203423                      
    },                                              
    "metaType": "biba"                              
  },                                                
  "activity": {                                     
    "lmvs": [                                       
      {                                             
        "player": "player 1",                       
        "light": "34s",                             
        "moderate": "1m3s",                                 
        "vigorous": "5m34s"                         
      },                                            
      {                                             
        "player": "player 2",
        "profile": "50960d97-1c93-4403-bf82-d07966a3a93e",
        "light": "3m20s",                           
        "moderate": "1m20s",                          
        "vigorous": "45s"                           
      }                                             
    ]                                               
  }                                                 
}      
```

### Ad Clicked

When an advertisement is clicked, this event is fired

**Example Data:**
```json
{                                                           
  "attributes": {                                                                                        
    "type": "ad_clicked",                           
    "timestamp": "2019-02-20T10:43:54.585814466-08:00",
    "env": "production",
    "game": "biba-copsnrobbers",
    "app": "biba-collection",
    "userId": "1b91f32d-74d1-4ac8-8a4a-34953b5577bb",
    "deviceId": "a58f1510-1e26-4b83-80d8-b35cc2c971eb",
    "deviceModel": "Pixel 2 XL",
    "deviceLanguage": "en_us",
    "os": "Android 9 / API 28",
    "version": "3.25.342",
    "location": {
      "latitude": 49.13234,
      "longitude": -121.203423
    },
    "metaType": "biba-for-school"
  },
  "adClicked": {
    "houseAdId": "crunch-dino-bone-cereal"
  }
}
```

### Ad Shown

When an advertisement is displayed to the user, this event is fired

**Example Data:**
```json
{
  "attributes": {
    "type": "ad_shown"
  },
  "adShown": {
    "houseAdId": "crunch-dino-bone-cereal"
  }
}
```

### AR Moment

When an AR moment is started in the game, this event is fired

**Example Data:**
```json
{
  "attributes": {
    "type": "ar_moment"
  },
  "arMoment": {
    "name": "castle-lobby"
  }
}
```

### Character Select

When the player ( or players, if there are more than one ) are allowed to select
a character to play as, this event is fired once all players have selected a character.

**Type:** `activity`

**Example Data:**
```json
{
  "attributes": {
    "type": "character_select"
  },
  "charSelect": {
    "selected": [
      {
        "playerName": "player 1",
        "profileId": "a8145b06-8a53-488b-bdf3-cd10961daa06",
        "characterName": "drac"
      },
      {
        "playerName": "player 2",
        "characterName": "bubba-ho-tep"
      }
    ]
  }
}
```

### Custom

When a game needs to send some data not already covered by the other events in
this document, this is the event that should be used.

**Warning!** This should not be used lightly -- if a custom event is used in
more than one game, a discussion should be had about turning it into a proper
event type.

**Example Data:**
```json
{
  "attributes": {
    "type": "custom"
  },
  "custom": {
    "example": "what"
  }
}
```

**Example Data:**
```json
{
  "attributes": {
    "type": "custom"
  },
  "custom": {
    "foo": "baz",
    "something": "wicked"
  }
}
```

### Cutscene

When a game shows a cutscene to the player, at the end of the cutscene ( or when
it is skipped ) this event fires.

**Example Data:**
```json
{
  "attributes": {
    "type": "cutscene"
  },
  "cutscene": {
    "name": "intro-pano"
  }
}
```

**Example Data (Skipped Cutscene):**
```json
{
  "attributes": {
    "type": "cutscene"
  },
  "cutscene": {
    "name": "intro-pano",
    "skipped": true
  }
}
```

### Equipment Select

After a user has selected all their equipment, this event is fired.

**Example Data:**
```json
{
  "attributes": {
    "type": "equip_select"
  },
  "equipSelect": {
    "selected": [
      "swing",
      "bridge"
    ]
  }
}
```

### Equipment Used

After a piece of equipment has been used, this event is fired

**Example Data:**
```json
{
  "attributes": {
    "type": "equip_used"
  },
  "equipUsed": {
    "equipment": [
      {
        "player": "player 1",
        "used": "swing"
      },
      {
        "player": "player 2",
        "profile": "4f0bce8b-fb40-4195-9f45-7055ce79fdcc",
        "used": "swing"
      }
    ],
    "presented": [
      "swing"
    ]
  }
}
```

### Round Start

Depending on the game, this event fires at different times:

* Dino Dig: from beginning a dinosaur set to completing that set, reaching the dino revelation screen
* Drive: completing a single race as a participating player, reaching the podium screen.
* Collectobots: finding all 6 biblets in a round, reaching the congrats screen.
* Treasure Hunter: continuing until one is booted from the temple, sees the treasure total score screen
* Butterfly Bounty: finding the golden butterfly (looong round, but it's all we have currently. may change in coming redux)
* Relay: participating in a relay until one times out and see the final score screen
* HT3: completing an island i.e. restoring an islands treasure
* HTTV: completing a full floor of two guests
* TT: completing any of the games by getting to their final score screens



**Example Data:**
```json
{
  "attributes": {
    "type": "round_start"
  },
  "roundStart": {
    "numPlayers": 2
  }
}
```

### Round End

**Example Data:**
```json
{
  "attributes": {
    "type": "round_end"
  },
  "roundEnd": {
    "roundLength": "5m21s"
  }
}
```

### Is Biba

When a user selects "Yes" or "No" on the "Do you see the Biba sign" screen, this
event is fired.

**Example Data (User Selected 'Yes'):**
```json
{
  "attributes": {
    "type": "is_biba"
  },
  "isBiba": {
    "isBiba": true
  }
}
```

**Example Data (User Selected 'No'):**
```json
{
  "attributes": {
    "type": "is_biba"
  },
  "isBiba": {
    "isBiba": false
  }
}
```


### Level Select

When a level is selected in games which allow players to select what level to
play, this event is fired after the level is selected.

**Example Data:**
```json
{
  "attributes": {
    "type": "level_select"
  },
  "levelSelect": {
    "selected": "island-one-section-two"
  }
}
```

### Mini Game

Upon completion of a min-game, this event is fired.

**Example Data:**
```json
{
  "attributes": {
    "type": "mini_game"
  },
  "miniGame": {
    "game": "blergh-buffet",
    "duration": "5m54s"
  }
}
```

### Purchase  

When a user completes an in-app purchase, this event is fired.

**Example Data:**
```json
{                                                   
  "attributes": {                                   
    "type": "purchase"                              
  },                                                
  "purchase": {  
    "platform": "ios",  
    "purchase": "343e5d6e-4015-4086-9a3e-b23b4d69450a",                                                  
    "price": "0.99",                               
    "code": "CAD"
  }
}
```

* `price` is set using `localizedPrice`
* `code` is set using `isoCurrencyCode`

### Screen View

Each screen in the game fires this event upon loading the screen.

**Example Data:**
```json
{
  "attributes": {
    "type": "screen_view"
  },
  "screenView": {
    "screenName": "pano",
    "previousScreenName": "main-menu"
  }
}
```

### Settings

When a user changes any settings, this event is fired

**Example Data:**
```json
{
  "attributes": {
    "type": "settings"
  },
  "settings": {
    "name": "language",
    "oldValue": "en_us",
    "newValue": "fr_ca"
  }
}
```

### Survey

When a user completes a playground survey, this event is fired

**Example Data:**
```json
{
  "attributes": {
    "type": "survey"
  },
  "survey": {
    "survey": {
      "question-id-one": "answer-one",
      "question-id-three": "answer-three",
      "question-id-two": "answer-two"
    }
  }
}
```

### Tag Scan

When a user scans a tag at a playground, this event is fired.

**Example Data:**
```json
{
  "attributes": {
    "type": "tag_scan"
  },
  "tagScan": {
    "name": "tag-id-one"
  }
}
```

If they choose to skip the tag scanning, add the "skipped" boolean:

```json
{
  "attributes": {
    "type": "tag_scan"
  },
  "tagScan": {
    "name": "tag-id-one",
    "skipped": true
  }
}
``**


### Weather

After fetching weather information from the API, this event is fired.

**Example Data**
```json
{
  "attributes": {                                                                                        
    "type": "weather"                               
  },
  "weather": {
    "description": "cloudy",
    "temp": 10.3,
    "windSpeed": 2.1
  }
}
```
