## Initialize Keycloak 24.0.2 with Node.js 18.18.2

To install Keycloak, go to the Keycloak official website [here](https://www.keycloak.org/getting-started/getting-started-zip) and create an admin user with a secure password. Make sure you can always remember it; this is an important step.

Then follow these steps to configure your Keycloak:

- Create the `company-services` realm.
- Disable the required action `Verify Profile`.
- Create the `movies-app` client. If Client authentication in Capability config is enabled, then:
  - Set `"directAccessGrantsEnabled": true`.
  - Set `"publicClient": true`. (It is default enabled in version 24.0.0 and above).
  - Set Valid redirect URIs to `http://localhost:3000/*`.
- Create the client role `MOVIES_USER` for the `movies-app` client.
- Create the client role `MOVIES_ADMIN` for the `movies-app` client.
- Create the `USERS` group.
- Create the `ADMINS` group.
- Add the `USERS` group as the realm default group.
- Assign the `MOVIES_USER` client role to the `USERS` group.
- Assign the `MOVIES_USER` and `MOVIES_ADMIN` client roles to the `ADMINS` group.
- Create the `user` user.
- Assign the `USERS` group to the user.
- Create the `admin` user.
- Assign the `ADMINS` group to the admin.

OR

## Setup Administrator Panel

- Realm: `company-services`.
- Client: `movies-app`.
- Client Roles: `MOVIES_MANAGER` and `USER`.
- Two users (During creation of users, must add `email`, `firstname`, `lastname`). It is an important step.
  - `admin`: with roles `MANAGE_MOVIES` and `USER`.
  - `user`: only with role `USER`.

### Movies API's

```bash
curl http://localhost:9080/api/movies

curl http://localhost:9080/api/movies/{imdbId}

curl -H "Authorization: Bearer YourAuthToken" http://localhost:9080/api/userextras/me

curl -X POST -H "Authorization: Bearer YourAuthToken" -H "Content-Type: application/json" -d '{"avatar":"newAvatarUrl"}' http://localhost:9080/api/userextras/me

curl -X POST -H "Authorization: Bearer YourAuthToken" -H "Content-Type: application/json" -d '{"title":"New Movie", "director":"Director Name"}' http://localhost:9080/api/movies

curl -X DELETE -H "Authorization: Bearer YourAuthToken" http://localhost:9080/api/movies/{imdbId}

curl -X POST -H "Authorization: Bearer YourAuthToken" -H "Content-Type: application/json" -d '{"text":"Great movie!"}' http://localhost:9080/api/movies/{imdbId}/comments
```

Unsecured API endpoints

```javascript
// curl http://localhost:9080/api/movies
app.get('/api/movies', (req, res) => {
  res.json({ message: 'List of movies' });
});

// curl http://localhost:9080/api/movies/{imdbId}
app.get('/api/movies/:imdbId', (req, res) => {
  res.json({ imdbId: req.params.imdbId, info: 'Details about the movie' });
});
```

Secured API endpoints

```javascript
// 1st API - User Login
curl -s -X POST "http://localhost:8080/realms/company-services/protocol/openid-connect/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=user" \
  -d "password=user" \
  -d "grant_type=password" \
  -d "client_id=movies-app" \
  -d "client_secret=YourClintSecret"

// Then execute 2nd curl
curl -H "Authorization: Bearer {AccessToken}" http://localhost:9080/api/userextras/me

OR

// 1st API - Admin Login
curl -s -X POST "http://localhost:8080/realms/company-services/protocol/openid-connect/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=admin" \
  -d "password=admin" \
  -d "grant_type=password" \
  -d "client_id=movies-app" \
  -d "client_secret=YourClintSecret" | jq -r .access_token

// Then execute 2nd curl
curl -H "Authorization: Bearer {AccessToken}" http://localhost:9080/api/userextras/me
```

```javascript
app.get('/api/userextras/me', keycloak.protect(['MOVIES_MANAGER','

USER']), (req, res) => {
  res.json({ profile: 'User extras profile' });
});

// Secured API endpoints
/*
// 1st API - User Login
curl -s -X POST "http://localhost:8080/realms/company-services/protocol/openid-connect/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=user" \
  -d "password=user" \
  -d "grant_type=password" \
  -d "client_id=movies-app" \
  -d "client_secret=YourClintSecret"

// Then execute 2nd curl
curl -X POST -H "Authorization: Bearer {AccessToken}" -H "Content-Type: application/json" -d '{"avatar":"newAvatarUrl"}' http://localhost:9080/api/userextras/me

OR

// 1st API - Admin Login
curl -s -X POST "http://localhost:8080/realms/company-services/protocol/openid-connect/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=admin" \
  -d "password=admin" \
  -d "grant_type=password" \
  -d "client_id=movies-app"

// Then execute 2nd curl
curl -X POST -H "Authorization: Bearer {AccessToken}" -H "Content-Type: application/json" -d '{"avatar":"newAvatarUrl"}' http://localhost:9080/api/userextras/me
*/
app.post('/api/userextras/me', keycloak.protect(['MOVIES_MANAGER','USER']), (req, res) => {
  res.json({ updated: true, avatar: req.body.avatar });
});

// Secured API endpoints only for admin
/*
// 1st API - User Login
curl -s -X POST "http://localhost:8080/realms/company-services/protocol/openid-connect/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=user" \
  -d "password=user" \
  -d "grant_type=password" \
  -d "client_id=movies-app" \
  -d "client_secret=YourClintSecret"

// Then execute 2nd curl
curl -X POST -H "Authorization: Bearer {AccessToken}" -H "Content-Type: application/json" -d '{"title":"New Movie", "director":"Director Name"}' http://localhost:9080/api/movies

OR

// 1st API - Admin Login
curl -s -X POST "http://localhost:8080/realms/company-services/protocol/openid-connect/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=admin" \
  -d "password=admin" \
  -d "grant_type=password" \
  -d "client_id=movies-app" \
  -d "client_secret=YourClintSecret"

// Then execute 2nd curl
curl -X POST -H "Authorization: Bearer {AccessToken}" -H "Content-Type: application/json" -d '{"title":"New Movie", "director":"Director Name"}' http://localhost:9080/api/movies
*/
// Use keycloak.protect('realm:MOVIES_MANAGER') or keycloak.protect(['MOVIES_MANAGER'])
app.post('/api/movies', keycloak.protect('realm:MOVIES_MANAGER'), (req, res) => {
  console.log('User authorized', req.kauth.grant.access_token.content);
  res.status(201).json(req.body);
});

// Secured API endpoints only for admin
/*
// 1st API - User Login
curl -s -X POST "http://localhost:8080/realms/company-services/protocol/openid-connect/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=user" \
  -d "password=user" \
  -d "grant_type=password" \
  -d "client_id=movies-app" \
  -d "client_secret=YourClintSecret"


// Then execute 2nd curl
curl -X DELETE -H "Authorization: Bearer {AccessToken}" http://localhost:9080/api/movies/{imdbId}
OR

// 1st API - Admin Login
curl -s -X POST "http://localhost:8080/realms/company-services/protocol/openid-connect/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=admin" \
  -d "password=admin" \
  -d "grant_type=password" \
  -d "client_id=movies-app" \
  -d "client_secret=YourClintSecret"

// Then execute 2nd curl
curl -X DELETE -

H "Authorization: Bearer {AccessToken}" http://localhost:9080/api/movies/{imdbId}
*/
app.delete('/api/movies/:imdbId', keycloak.protect('realm:MOVIES_MANAGER'), (req, res) => {
  res.status(204).send();
});

// Secured API endpoints both admin, user
/*
// 1st API - User Login
curl -s -X POST "http://localhost:8080/realms/company-services/protocol/openid-connect/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=user" \
  -d "password=user" \
  -d "grant_type=password" \
  -d "client_id=movies-app" \
  -d "client_secret=YourClintSecret"

// Then execute 2nd curl
curl -X DELETE -H "Authorization: Bearer {AccessToken}" http://localhost:9080/api/movies/{imdbId}
OR

// 1st API - Admin Login
curl -s -X POST "http://localhost:8080/realms/company-services/protocol/openid-connect/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=admin" \
  -d "password=admin" \
  -d "grant_type=password" \
  -d "client_id=movies-app" \
  -d "client_id=movies-app" \
  -d "client_secret=YourClintSecret"

// Then execute 2nd curl
curl -X POST -H "Authorization: Bearer {AccessToken}" -H "Content-Type: application/json" -d '{"text":"Great movie!"}' http://localhost:9080/api/movies/{imdbId}/comments
*/
app.post('/api/movies/:imdbId/comments', keycloak.protect(['MOVIES_MANAGER','USER']), (req, res) => {
  res.status(201).json({ imdbId: req.params.imdbId, comment: req.body.text });
});

app.listen(port, () => {
  console.log(`Server Started at ${port}`);
});


Here's the formatted README file:

---

## Initialize Keycloak 24.0.2 with Node.js 18.18.2

To install Keycloak, go to the Keycloak official website [here](https://www.keycloak.org/getting-started/getting-started-zip) and create an admin user with a secure password. Make sure you can always remember it; this is an important step.

Then follow these steps to configure your Keycloak:

- Create the `company-services` realm.
- Disable the required action `Verify Profile`.
- Create the `movies-app` client. If Client authentication in Capability config is enabled, then:
  - Set `"directAccessGrantsEnabled": true`.
  - Set `"publicClient": true`. (It is default enabled in version 24.0.0 and above).
  - Set Valid redirect URIs to `http://localhost:3000/*`.
- Create the client role `MOVIES_USER` for the `movies-app` client.
- Create the client role `MOVIES_ADMIN` for the `movies-app` client.
- Create the `USERS` group.
- Create the `ADMINS` group.
- Add the `USERS` group as the realm default group.
- Assign the `MOVIES_USER` client role to the `USERS` group.
- Assign the `MOVIES_USER` and `MOVIES_ADMIN` client roles to the `ADMINS` group.
- Create the `user` user.
- Assign the `USERS` group to the user.
- Create the `admin` user.
- Assign the `ADMINS` group to the admin.

OR

## Setup Administrator Panel

- Realm: `company-services`.
- Client: `movies-app`.
- Client Roles: `MOVIES_MANAGER` and `USER`.
- Two users (During creation of users, must add `email`, `firstname`, `lastname`). It is an important step.
  - `admin`: with roles `MANAGE_MOVIES` and `USER`.
  - `user`: only with role `USER`.

---

### Movies API's

```bash
curl http://localhost:9080/api/movies

curl http://localhost:9080/api/movies/{imdbId}

curl -H "Authorization: Bearer YourAuthToken" http://localhost:9080/api/userextras/me

curl -X POST -H "Authorization: Bearer YourAuthToken" -H "Content-Type: application/json" -d '{"avatar":"newAvatarUrl"}' http://localhost:9080/api/userextras/me

curl -X POST -H "Authorization: Bearer YourAuthToken" -H "Content-Type: application/json" -d '{"title":"New Movie", "director":"Director Name"}' http://localhost:9080/api/movies

curl -X DELETE -H "Authorization: Bearer YourAuthToken" http://localhost:9080/api/movies/{imdbId}

curl -X POST -H "Authorization: Bearer YourAuthToken" -H "Content-Type: application/json" -d '{"text":"Great movie!"}' http://localhost:9080/api/movies/{imdbId}/comments
```

---

Unsecured API endpoints

```javascript
// curl http://localhost:9080/api/movies
app.get('/api/movies', (req, res) => {
  res.json({ message: 'List of movies' });
});

// curl http://localhost:9080/api/movies/{imdbId}
app.get('/api/movies/:imdbId', (req, res) => {
  res.json({ imdbId: req.params.imdbId, info: 'Details about the movie' });
});
```

---

Secured API endpoints

```javascript
// 1st API - User Login
// Then execute 2nd curl
// OR
// 1st API - Admin Login
// Then execute 2nd curl
app.get('/api/userextras/me', keycloak.protect(['MOVIES_MANAGER','USER']), (req, res) => {
  res.json({ profile: 'User extras profile' });
});

// Secured API endpoints
// OR
// Secured API endpoints only for admin
// OR
// Secured API endpoints only for admin
// OR
// Secured API endpoints both admin, user
app.post('/api/movies/:imdbId/comments', keycloak.protect(['MOVIES_MANAGER','USER']), (req, res) => {
  res.status(201).json({ imdbId: req.params.imdbId, comment: req.body.text });
});

app.listen(port, () => {
  console.log(`Server Started at ${port}`);
});
```

---

This README provides instructions on setting up and configuring Keycloak with Node.js, as well as details on using the Movies API.