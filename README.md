# Keycloak Sample App

A simple web application demonstrating **OpenID Connect (OIDC)** authentication with Keycloak.

![Keycloak](https://img.shields.io/badge/Keycloak-26.x-blue)
![License](https://img.shields.io/badge/License-MIT-green)

---

## 🔄 Authentication Flow

When a user clicks "Login", this is what happens:

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Your App   │     │  Keycloak   │     │    User     │
│ :3000       │     │   :8080     │     │  (Browser)  │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │
       │ 1. User clicks    │                   │
       │    "Login"        │◄──────────────────│
       │                   │                   │
       │ 2. Redirect to    │                   │
       │    Keycloak       │──────────────────>│
       │                   │                   │
       │                   │ 3. Show login     │
       │                   │    page           │
       │                   │──────────────────>│
       │                   │                   │
       │                   │ 4. User enters    │
       │                   │    credentials    │
       │                   │◄──────────────────│
       │                   │                   │
       │ 5. Redirect back  │                   │
       │    with auth code │                   │
       │◄──────────────────│                   │
       │                   │                   │
       │ 6. Exchange code  │                   │
       │    for tokens     │                   │
       │──────────────────>│                   │
       │                   │                   │
       │ 7. Return tokens  │                   │
       │    (access_token, │                   │
       │     id_token,     │                   │
       │     refresh_token)│                   │
       │◄──────────────────│                   │
       │                   │                   │
       │ 8. User is now    │                   │
       │    authenticated! │──────────────────>│
       └───────────────────┴───────────────────┘
```

This is the **OAuth 2.0 Authorization Code Flow with PKCE**.

---

## 📋 Requirements

- **Node.js** 18+ (for running the dev server)
- **Keycloak** running on `http://localhost:8080`
- **npx** (comes with Node.js)

---

## 🚀 How to Run

### Step 1: Start Keycloak

Make sure Keycloak is running on port 8080 with bootstrap admin credentials:

```bash
# From Keycloak source directory
KC_BOOTSTRAP_ADMIN_USERNAME=admin \
KC_BOOTSTRAP_ADMIN_PASSWORD=admin \
java -Dkc.config.built=true -jar quarkus/server/target/lib/quarkus-run.jar start-dev
```

### Step 2: Setup Keycloak (Realm, Client, User)

#### Get Admin Token

```bash
TOKEN=$(curl -s -X POST "http://localhost:8080/realms/master/protocol/openid-connect/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=admin" \
  -d "password=admin" \
  -d "grant_type=password" \
  -d "client_id=admin-cli" | jq -r '.access_token')

echo "Token: ${TOKEN:0:50}..."
```

#### Create Realm

```bash
curl -s -X POST "http://localhost:8080/admin/realms" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "realm": "demo",
    "enabled": true,
    "registrationAllowed": true,
    "resetPasswordAllowed": true
  }'

echo "✅ Realm 'demo' created!"
```

#### Create Client

```bash
curl -s -X POST "http://localhost:8080/admin/realms/demo/clients" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "clientId": "sample-app",
    "name": "Sample Web App",
    "enabled": true,
    "publicClient": true,
    "directAccessGrantsEnabled": true,
    "standardFlowEnabled": true,
    "redirectUris": ["http://localhost:3000/*"],
    "webOrigins": ["http://localhost:3000"],
    "attributes": {
      "pkce.code.challenge.method": "S256"
    }
  }'

echo "✅ Client 'sample-app' created!"
```

#### Create Test User

```bash
curl -s -X POST "http://localhost:8080/admin/realms/demo/users" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "email": "test@example.com",
    "firstName": "Test",
    "lastName": "User",
    "enabled": true,
    "emailVerified": true,
    "credentials": [{
      "type": "password",
      "value": "test123",
      "temporary": false
    }]
  }'

echo "✅ User 'testuser' created with password 'test123'!"
```

### Step 3: Run the Sample App

```bash
cd keycloak-sample-app
npx serve -l 3000
```

### Step 4: Open in Browser

Navigate to **http://localhost:3000** and click **"Login with Keycloak"**

**Test Credentials:**
- Username: `testuser`
- Password: `test123`

---

## 📁 Project Structure

```
keycloak-sample-app/
├── index.html           # Main HTML (login UI, user info display)
├── styles.css           # All CSS styles
├── keycloak.js          # Keycloak JS adapter (ES module)
├── silent-check-sso.html # Silent SSO check page
├── package.json         # npm config
└── README.md            # This file
```

---

## 🔗 Useful URLs

| URL | Description |
|-----|-------------|
| http://localhost:3000 | Sample App |
| http://localhost:8080/admin | Keycloak Admin Console |
| http://localhost:8080/admin/master/console/#/demo | Demo Realm Admin |
| http://localhost:8080/realms/demo/.well-known/openid-configuration | OIDC Discovery |
| http://localhost:8080/realms/demo/protocol/openid-connect/token | Token Endpoint |
| http://localhost:8080/realms/demo/protocol/openid-connect/userinfo | UserInfo Endpoint |

---

## 🔧 Configuration

The app is configured to connect to:

| Setting | Value |
|---------|-------|
| Keycloak URL | `http://localhost:8080` |
| Realm | `demo` |
| Client ID | `sample-app` |
| PKCE | Enabled (S256) |

To modify these settings, edit the Keycloak configuration in `index.html`:

```javascript
keycloak = new Keycloak({
    url: 'http://localhost:8080',
    realm: 'demo',
    clientId: 'sample-app'
});
```

---

## 📚 Features Demonstrated

- ✅ **OAuth 2.0 Authorization Code Flow** with PKCE
- ✅ **OpenID Connect** authentication
- ✅ **JWT Access Token** inspection
- ✅ **Token Refresh** functionality
- ✅ **UserInfo Endpoint** API call
- ✅ **Silent SSO Check** for session persistence
- ✅ **Logout** with redirect

---

## 🛠️ Troubleshooting

### "Invalid username or password"
- Make sure the user was created in the `demo` realm (not `master`)
- Verify credentials: `testuser` / `test123`

### "Keycloak unavailable"
- Ensure Keycloak is running on `http://localhost:8080`
- Check if the `demo` realm exists

### CORS errors
- Make sure the client's `webOrigins` includes `http://localhost:3000`
- Verify `redirectUris` includes `http://localhost:3000/*`

### Token expired
- Click "Refresh Token" button
- Or logout and login again

---

## 📄 License

MIT

