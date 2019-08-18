## Concept

On the top level, what WCA Live does is:
- import competition WCIF
- let's the user work on it by adding/modifying round results
- save WCIF back

Additionally it provides a public interface for viewing competitions and results.

Following the [WCIF vision](https://docs.google.com/document/d/1Kalp1k24ga9N8DRYTk4NPeCZJ4jw8WBMU5N7zeuklm0)
this results platform is not meant to be an all-in-one solution for running competitions,
but rather a part in the puzzle dedicated to entering results and displaying them to the world.

## Technology

- MongoDB
- Node.js
- React
- GraphQL (specifically Apollo)
- Material UI

## Database structure

We use MongoDB to store imported WCIF. We don't destructure it into many collections,
but keep as is, which has several benefits:
- ability to easily cache whole competition (keep in memory), see `server/competition-loader.js` for more details
- we operate on the plain, common WCIF format, instead of specific data models
- no logic related to loading WCIF into/from collections
- it takes one query to get all data necessary to serve any supported GraphQL query

The database schema looks like so:

#### `competitions`

```json
{
  "title": "competition",
  "type": "object",
  "properties": {
    "_id": {
      "type": "objectId"
    },
    "wcif": {
      "type": "object",
      "description": "Competition WCIF."
    },
    "importedById": {
      "type": "objectId",
      "description": "Id of the user who imported the competition from the WCA website."
    },
    "managerWcaUserIds": {
      "type": "array",
      "description": "Organizers, delegates and dataentries.",
      "items": {
        "type": "number",
        "description": "User ID in the WCA database."
      }
    },
    "synchronizedAt": {
      "type": "Date",
      "description": "Time of last synchronization."
    }
  }
}
```

#### `users`

```json
{
  "title": "user",
  "type": "object",
  "properties": {
    "_id": {
      "type": "objectId"
    },
    "wcaUserId": {
      "type": "number",
      "description": "User ID in the WCA database."
    },
    "wcaId": {
      "type": "string"
    },
    "name": {
      "type": "string"
    },
    "avatar": {
      "type": "object",
      "properties": {
        "url": {
          "type": "string"
        },
        "thumbUrl": {
          "type": "string"
        }
      }
    }
  }
}
```

Naturally the only part of WCIF that the app modifies is `Round#results`.
For every `Result` we store some additional app-specific data:

```json
{
  "title": "result",
  "description": "Additional data on WCIF#Result.",
  "type": "object",
  "properties": {
    "recordTags": {
      "type": "object",
      "description": "Regional record tags.",
      "properties": {
        "single": {
          "enum": ["WR", "CR", "NR", "PB", null]
        },
        "average": {
          "enum": ["WR", "CR", "NR", "PB", null]
        }
      }
    },
    "updatedAt": {
      "type": "date"
    }
  }
}
```

## Wording

- WCIF (WCA Competition Interchange Format) is a specification of competition data in JSON format.
It's designed as a way for many applications to exchange data in a standardized manner.
In practice WCIF usually refers the actual competition JSON including
competitors, events/rounds data, schedule, etc. You can find the full specification
[here](https://docs.google.com/document/d/1lE70gW2rA2J2WlXeSl6N2mLnJ52ibTP_VsIwrAVdl2I/edit).