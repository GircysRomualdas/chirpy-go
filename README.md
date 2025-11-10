# Chirpy

Chirpy is a JSON REST API built with Go, allowing users to create accounts, authenticate, and post short text messages. 

This is the starter code used in Boot.dev's [Learn HTTP Servers in Go](https://www.boot.dev/courses/learn-http-servers-golang) course.

---

## Requirements

- Go 1.20+
- PostgreSQL

---

## Installation

1. Clone the Repository

2. Install dependencies:
```bash
go mod download
```

3. Install PostgreSQL:  
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```
4. Configure PostgreSQL:

```bash
sudo service postgresql start
sudo -u postgres psql
```

Inside the psql shell:
```sql
CREATE DATABASE chirpy;
ALTER USER postgres PASSWORD 'postgres';
\q
```

5. Create a `.env` file in the project root:
```bash
DATABASE_URL=postgres://postgres:postgres@localhost:5432/chirpy?sslmode=disable
JWT_SECRET=<jwt_secret>
POLKA_KEY=<polka_api_key>
```

For `<jwt_secret>`, generate a random string key with:
```bash
openssl rand -base64 64
```

For `<polka_api_key>`, you can use any random string. Example (for Boot.Dev endpoint test):
```
f271c81ff7084ee5b99a5091b42d486e
```

6. Run migrations:
```bash
goose -dir ./sql/schema postgres "postgres://postgres:postgres@localhost:5432/chirpy?sslmode=disable" up
```

---

## Usage

Run chirpy:
```bash
go run .
```

The API will start on http://localhost:8080

### Example Session

#### Health check:
```bash
curl http://localhost:8080/api/healthz/
```

Output:
```bash
OK
```

#### Reset the server:
```bash
curl -X POST http://localhost:8080/admin/reset
```

Output:
```text
Hits reset to 0 and all users deleted
```

#### Create a user:
```bash
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"email":"walt@breakingbad.com","password":"123456"}'
```

Output:
```json
{
  "id":"1dc6c975-f3f2-411e-a87a-572eff782a45",
  "created_at":"2025-11-10T15:30:44.293773Z",
  "updated_at":"2025-11-10T15:30:44.293773Z",
  "email":"walt@breakingbad.com",
  "is_chirpy_red":false
}
```

#### Login:
```bash
curl -X POST http://localhost:8080/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"walt@breakingbad.com","password":"123456"}'
```

Output:
```json
{
  "id":"e9bebb70-ac45-4966-a9e6-3a13a9b6a373",
  "created_at":"2025-11-10T16:54:00.501032Z",
  "updated_at":"2025-11-10T16:54:00.501032Z",
  "email":"walt@breakingbad.com",
  "is_chirpy_red":false,
  "token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJjaGlycHkiLCJzdWIiOiJlOWJlYmI3MC1hYzQ1LTQ5NjYtYTllNi0zYTEzYTliNmEzNzMiLCJleHAiOjE3NjI3OTAwNTQsImlhdCI6MTc2Mjc4NjQ1NH0.BzNd4R4mqIjOywcSiO5W4bnE57lToBXD2AOopjBDD_s",
  "refresh_token":"564b5d32f2fd9d3cefff898851df7a6496f25a1d91ffa7fe1fa351b03f4148e1"
}
```

Store the token and user id in a bash variables `USER_TOKEN`, `USER_REFRESH_TOKEN` and `USER_ID` to reuse it. Example (use token, refresh token and id from user login):
```bash
USER_TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJjaGlycHkiLCJzdWIiOiJlOWJlYmI3MC1hYzQ1LTQ5NjYtYTllNi0zYTEzYTliNmEzNzMiLCJleHAiOjE3NjI3OTAwNTQsImlhdCI6MTc2Mjc4NjQ1NH0.BzNd4R4mqIjOywcSiO5W4bnE57lToBXD2AOopjBDD_s"
USER_ID="e9bebb70-ac45-4966-a9e6-3a13a9b6a373"
USER_REFRESH_TOKEN="564b5d32f2fd9d3cefff898851df7a6496f25a1d91ffa7fe1fa351b03f4148e1"
```

#### Post chirp:
First post:
```bash
curl -X POST http://localhost:8080/api/chirps \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${USER_TOKEN}" \
  -d '{"body":"I am the one who knocks!"}'
```

Output:
```json
{
  "id":"da6f7102-95c2-4b75-9d7a-26d9a89d67cf",
  "created_at":"2025-11-10T16:58:17.449208Z",
  "updated_at":"2025-11-10T16:58:17.449208Z",
  "body":"I am the one who knocks!",
  "user_id":"e9bebb70-ac45-4966-a9e6-3a13a9b6a373"
}
```

Second post:

```bash
curl -X POST http://localhost:8080/api/chirps \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${USER_TOKEN}" \
  -d '{"body":"We need to cook"}'
```

Output:
```json
{
  "id":"955d289f-c13f-49fe-b0b2-0670730ba56b",
  "created_at":"2025-11-10T16:59:55.020054Z",
  "updated_at":"2025-11-10T16:59:55.020054Z",
  "body":"We need to cook",
  "user_id":"e9bebb70-ac45-4966-a9e6-3a13a9b6a373"
}
```

#### Update user info:
```bash
curl -X PUT http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${USER_TOKEN}" \
  -d '{"email":"walter@breakingbad.com","password":"losPollosHermanos"}'
```

Output:
```json
{
  "id":"e9bebb70-ac45-4966-a9e6-3a13a9b6a373",
  "created_at":"2025-11-10T16:54:00.501032Z",
  "updated_at":"2025-11-10T16:54:00.501032Z",
  "email":"walter@breakingbad.com",
  "is_chirpy_red":false
}
```


#### Retrieve all chirps (sorted `desc` or `asc`):
```bash
curl http://localhost:8080/api/chirps?sort=desc
```

Output:
```json
[
  {
    "id":"955d289f-c13f-49fe-b0b2-0670730ba56b",
    "created_at":"2025-11-10T16:59:55.020054Z",
    "updated_at":"2025-11-10T16:59:55.020054Z",
    "body":"We need to cook",
    "user_id":"e9bebb70-ac45-4966-a9e6-3a13a9b6a373"
  },
  {
    "id":"da6f7102-95c2-4b75-9d7a-26d9a89d67cf",
    "created_at":"2025-11-10T16:58:17.449208Z",
    "updated_at":"2025-11-10T16:58:17.449208Z",
    "body":"I am the one who knocks!",
    "user_id":"e9bebb70-ac45-4966-a9e6-3a13a9b6a373"
  }
]
```

Store the chirp id in a bash variable `CHIRP_ID` to reuse it. Example (use chirp id from retrieve all chirps):
```bash
CHIRP_ID="955d289f-c13f-49fe-b0b2-0670730ba56b"
```

#### Get chirp by ID:
```bash
curl http://localhost:8080/api/chirps/${CHIRP_ID}
```

Output:
```json
{
  "id":"955d289f-c13f-49fe-b0b2-0670730ba56b",
  "created_at":"2025-11-10T16:59:55.020054Z",
  "updated_at":"2025-11-10T16:59:55.020054Z",
  "body":"We need to cook",
  "user_id":"e9bebb70-ac45-4966-a9e6-3a13a9b6a373"
}
```

#### Delete chirp:
```bash
curl -X DELETE http://localhost:8080/api/chirps/${CHIRP_ID} \
  -H "Authorization: Bearer ${USER_TOKEN}"
```

Retrieve all chirps
```bash
curl http://localhost:8080/api/chirps?sort=desc
```

Output:
```json
[
  {
    "id":"da6f7102-95c2-4b75-9d7a-26d9a89d67cf",
    "created_at":"2025-11-10T16:58:17.449208Z",
    "updated_at":"2025-11-10T16:58:17.449208Z",
    "body":"I am the one who knocks!",
    "user_id":"e9bebb70-ac45-4966-a9e6-3a13a9b6a373"
  }
]
```

#### Filter chirps by user:
```bash
curl http://localhost:8080/api/chirps?author_id=${USER_ID}
```

Output:
```json
[
  {
    "id":"da6f7102-95c2-4b75-9d7a-26d9a89d67cf",
    "created_at":"2025-11-10T16:58:17.449208Z",
    "updated_at":"2025-11-10T16:58:17.449208Z",
    "body":"I am the one who knocks!",
    "user_id":"e9bebb70-ac45-4966-a9e6-3a13a9b6a373"
  }
]
```

#### Polka webhook:
Boot.Dev webhook endpoint test with api key `f271c81ff7084ee5b99a5091b42d486e` to upgrade user to chirpy red.
```bash
curl -X POST http://localhost:8080/api/polka/webhooks \
  -H "Content-Type: application/json" \
  -H "Authorization: ApiKey f271c81ff7084ee5b99a5091b42d486e" \
  -d '{"event":"user.upgraded","data":{"user_id":"'${USER_ID}'"}}'
```

To see user chirp red run update user info:
```bash
curl -X PUT http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${USER_TOKEN}" \
  -d '{"email":"walter@breakingbad.com","password":"losPollosHermanos"}'
```

Output:
```json
{
  "id":"e9bebb70-ac45-4966-a9e6-3a13a9b6a373",
  "created_at":"2025-11-10T16:54:00.501032Z",
  "updated_at":"2025-11-10T16:54:00.501032Z",
  "email":"walter@breakingbad.com",
  "is_chirpy_red":true
}
```

#### Refresh access token:
```bash
curl -X POST http://localhost:8080/api/refresh \
  -H "Authorization: Bearer ${USER_REFRESH_TOKEN}"
```

Output:
```json
{
  "token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJjaGlycHkiLCJzdWIiOiJlOWJlYmI3MC1hYzQ1LTQ5NjYtYTllNi0zYTEzYTliNmEzNzMiLCJleHAiOjE3NjI3OTE2NTMsImlhdCI6MTc2Mjc4ODA1M30.jvbhQzYqUGMiI5PZ6s7VH7l-poepLzp5YUTzfkQWeAw"
}
```

Set new refresh token:
```bash
USER_REFRESH_TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJjaGlycHkiLCJzdWIiOiIxZGM2Yzk3NS1mM2YyLTQxMWUtYTg3YS01NzJlZmY3ODJhNDUiLCJleHAiOjE3NjI3ODkxMjcsImlhdCI6MTc2Mjc4NTUyN30.Pu7RcLzDVEUEl8mLTIHzRC2bRY9r0bedU-4t65kfkKk"
```

#### Revoke refresh token:
```bash
curl -X POST http://localhost:8080/api/revoke \
  -H "Authorization: Bearer ${USER_REFRESH_TOKEN}"
```

Run refresh access token:
```bash
curl -X POST http://localhost:8080/api/refresh \
  -H "Authorization: Bearer ${USER_REFRESH_TOKEN}"
```

Output:
```bash
{
  "error":"Refresh token revoked"
}
```

---

## Endpoints

| Method | Endpoint | Description |
|--------|-----------|-------------|
| GET | `/api/healthz/` | Health check |
| POST | `/api/users` | Create a new user |
| PUT | `/api/users` | Update user info (requires JWT auth) |
| POST | `/api/login` | Log in and receive access + refresh tokens |
| POST | `/api/refresh` | Refresh access token using refresh token |
| POST | `/api/revoke` | Revoke a refresh token |
| POST | `/api/chirps` | Create a new chirp (requires JWT auth) |
| GET | `/api/chirps` | List all chirps (supports `author_id` and `sort=asc|desc` query params) |
| GET | `/api/chirps/{chirpID}` | Get a single chirp by ID |
| DELETE | `/api/chirps/{chirpID}` | Delete chirp (author only, requires JWT auth) |
| POST | `/api/polka/webhooks` | Handle Polka payment webhooks for upgrades |
| GET | `/admin/metrics` | View site visit metrics |
| POST | `/admin/reset` | Reset all users and metrics |
