# API reference

The Express API runs on `http://localhost:3001` by default. All resource responses use JSON.

## Base behavior

| Item | Current implementation |
| --- | --- |
| Root | `GET /` returns text `"This is home route"`. |
| JSON | Express parses JSON request bodies. |
| CORS | `cors()` is enabled for any origin. |
| Security headers | Helmet is enabled. |
| Logging | Morgan uses the `common` format. |
| Authentication | The client sends a Cognito bearer token, but the server does not validate or authorize it. |
| Success shape | Most writes/reads return `{ "success": true, "data": ... }`. |
| Error shape | Most controllers return `{ "message": "..." }` with status `500`. |

The server does not implement pagination, explicit validation, standardized 400/404 responses, or an OpenAPI specification.

## Projects

### `GET /projects`

Returns every project.

Response `200`:

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Apollo",
      "description": "A space exploration project.",
      "startDate": "2023-01-01T00:00:00.000Z",
      "endDate": "2023-12-31T00:00:00.000Z"
    }
  ]
}
```

### `POST /projects`

Creates a project.

Request body:

```json
{
  "name": "Apollo",
  "description": "A space exploration project.",
  "startDate": "2023-01-01T00:00:00.000Z",
  "endDate": "2023-12-31T00:00:00.000Z"
}
```

`name` is required by Prisma. `description`, `startDate`, and `endDate` are optional.

Response `201`:

```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "Apollo",
    "description": "A space exploration project.",
    "startDate": "2023-01-01T00:00:00.000Z",
    "endDate": "2023-12-31T00:00:00.000Z"
  }
}
```

## Tasks

### `GET /tasks/project/:id`

Returns tasks for a project. Each task includes `author`, `assignee`, `comments`, and `attachments`.

Example:

```bash
curl "http://localhost:3001/tasks/project/1"
```

### `GET /tasks/user/:id`

Returns tasks where `authorUserId` or `assignedUserId` equals the numeric user ID. Each task includes `author` and `assignee`.

Example:

```bash
curl "http://localhost:3001/tasks/user/1"
```

### `POST /tasks`

Creates a task.

Request body:

```json
{
  "title": "Prepare launch checklist",
  "description": "Review launch dependencies.",
  "status": "To Do",
  "priority": "High",
  "tags": "launch,operations",
  "startDate": "2026-01-10T00:00:00.000Z",
  "dueDate": "2026-01-20T00:00:00.000Z",
  "points": 5,
  "projectId": 1,
  "authorUserId": 1,
  "assignedUserId": 2
}
```

Required by the Prisma task model:

- `title`
- `projectId`
- `authorUserId`

`assignedUserId` may be omitted or `null`. Status and priority are unvalidated strings.

Response `201`: `{ "success": true, "data": <task> }`.

### `PATCH /tasks/:id/status`

Updates only the task status.

Request body:

```json
{
  "status": "Completed"
}
```

Response `200`: `{ "success": true, "data": <updated task> }`.

Known client status values are `To Do`, `Work In Progress`, `Under Review`, and `Completed`; the database does not constrain them.

## Search

### `GET /search?query=:query`

Case behavior follows PostgreSQL/Prisma `contains`; the implementation does not escape or validate the value beyond casting it to a string.

Returns matching tasks by title/description, projects by name/description, and users by username:

```json
{
  "success": true,
  "data": {
    "tasks": [],
    "projects": [],
    "users": []
  }
}
```

Example:

```bash
curl --get --data-urlencode "query=apollo" "http://localhost:3001/search"
```

The client triggers this endpoint after the search term reaches at least three characters.

## Users

### `GET /users`

Returns all database users.

### `POST /users`

Creates a database user.

Request body:

```json
{
  "username": "AliceJones",
  "cognitoId": "Cognito-sub-value",
  "profilePictureUrl": "p1.jpeg",
  "teamId": 1
}
```

`username` and `cognitoId` are required by Prisma and unique. The controller defaults:

- `profilePictureUrl`: `i1.jpg`
- `teamId`: `1`

Response `201`:

```json
{
  "success": true,
  "data": { "id": 1 },
  "message": "User Created Successfully"
}
```

The response example is abbreviated; the actual `data` object contains the created Prisma user.

### `GET /users/:cognitoId`

Finds a database user by the unique Cognito ID field.

Example:

```bash
curl "http://localhost:3001/users/123e4567-e89b-12d3-a456-426614174001"
```

Response `200`:

```json
{
  "success": true,
  "data": { "id": 1 }
}
```

The actual data contains the matching user fields. If no user exists, Prisma returns `null` and the controller still returns HTTP `200`.

## Teams

### `GET /teams`

Returns teams with a nested `productOwner` and `projectManager` object derived from numeric IDs.

Example shape:

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "teamName": "Quantum Innovations",
      "productOwnerUserId": 11,
      "projectManagerUserId": 2,
      "productOwner": {
        "username": "example",
        "profilePictureUrl": "p11.jpeg"
      },
      "projectManager": {
        "username": "example",
        "profilePictureUrl": "p2.jpeg"
      }
    }
  ]
}
```

If an owner ID is missing or unresolved, nested fields may contain `null`/`undefined` values as described by the controller.

## Error handling

Database/controller exceptions are caught and converted to `500` responses:

```json
{
  "message": "Error retrieving projects: <database error>"
}
```

The exact message includes the underlying error. This is not a stable public error contract.

There is no global Express error handler. Invalid route IDs, missing fields, missing records, and Prisma constraint failures generally depend on each controller and may return `500` or, in some cases, `200` with `null`.
