# CommBank-Server — Goal Model: Icon Field

This task extends the existing .NET backend (rSERVER) to support an optional
`Icon` field on the `Goal` model, backed by MongoDB.

## Overview

| | |
|---|---|
| **Task** | Modify an Existing .NET Backend |
| **Stack** | ASP.NET Core (.NET), MongoDB.Driver, MongoDB Atlas |
| **Goal** | Add an optional, public `Icon` (string) field to the `Goal` model without breaking existing data |

## What changed

### `Models/Goal.cs`

Added a new optional string property, `Icon`, mapped to the `icon` field in
MongoDB:

```csharp
[BsonElement("icon")]
[BsonIgnoreIfNull]
public string? Icon { get; set; }
```

- **Optional** — nullable (`string?`), so existing Goal documents without an
  `icon` field remain valid and don't error on deserialization.
- **`[BsonIgnoreIfNull]`** — MongoDB won't persist the field when it's `null`,
  keeping older documents clean.
- **Public** — exposed the same way as every other field on `Goal`, so it is
  returned automatically by the existing `GoalController` endpoints with no
  additional route changes required.

No other files needed changes — `GoalController.cs`, `GoalService.cs`, and
`IGoalsService.cs` all operate on the `Goal` model directly, so the new field
flows through to the API response automatically.

## Setup steps followed

1. Forked [`fencer-so/commbank-server`](https://github.com/fencer-so/commbank-server)
2. Created a free MongoDB Atlas cluster (M0 tier)
3. Created a database user with read/write access
4. Connected the server to the database via `appsettings.Development.json`:
   ```json
   {
     "MongoDbSettings": {
       "ConnectionString": "mongodb+srv://<user>:<password>@<cluster-url>/?retryWrites=true&w=majority",
       "DatabaseName": "commbank"
     }
   }
   ```
5. Seeded the database using the sample data from `/data`
6. Verified via Postman/Swagger that `GET /api/Goal` succeeded **without** an
   `icon` field present (baseline, pre-change)
7. Added the `Icon` field to `Models/Goal.cs` (see above)
8. Re-verified via Postman/Swagger that `GET /api/Goal` succeeded **with**
   `icon` values returned for goals that have one set
9. Saved the Swagger/Postman response body as `goal_response.json` for
   submission

## Running locally

```bash
cd CommBank-Server
dotnet build
dotnet run
```

Once running, open Swagger to explore and test all endpoints:

```
http://localhost:<port>/swagger
```

### Testing the Goal endpoint

```
GET /api/Goal
```

Example response (with icon):

```json
[
  {
    "id": "64f1a2b3c4d5e6f7a8b9c0d1",
    "name": "Vacation Fund",
    "targetAmount": 5000,
    "targetDate": "2026-12-31T00:00:00Z",
    "balance": 1200,
    "transactionIds": [],
    "tagIds": [],
    "icon": "palm-tree",
    "userId": "64f1a2b3c4d5e6f7a8b9c0aa"
  }
]
```

## Submission

The captured API response is saved as [`goal_response.json`](./goal_response.json)
in this repo, showing the `Goal` endpoint returning the new `icon` field.