# FastAPI — OAuth2 Scopes & HTTP Basic Auth

---

## OAuth2 Scopes

The OAuth2 specification defines **scopes** as a list of strings separated by spaces. The content of each string can have any format but should not contain spaces. Scopes represent **permissions**.

**Common examples:**
- `users:read` or `users:write`
- `instagram_basic` (used by Facebook/Instagram)
- `https://www.googleapis.com/auth/drive` (used by Google)

### Full OAuth2 with Scopes Example

```python
from datetime import datetime, timedelta, timezone
from typing import Annotated
import jwt
from fastapi import Depends, FastAPI, HTTPException, Security, status
from fastapi.security import (
    OAuth2PasswordBearer,
    OAuth2PasswordRequestForm,
    SecurityScopes,  # Special class to collect required scopes
)
from jwt.exceptions import InvalidTokenError
from pwdlib import PasswordHash
from pydantic import BaseModel, ValidationError

# 🔐 Secret and JWT settings
SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# 🧪 Fake database
fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "...",
        "disabled": False,
    }
}

# 📦 Response model for token
class Token(BaseModel):
    access_token: str
    token_type: str

# 📦 Data extracted from JWT token
class TokenData(BaseModel):
    username: str | None = None
    scopes: list[str] = []  # ✅ Stores scopes from JWT

# 📦 User models
class User(BaseModel):
    username: str
    disabled: bool | None = None

class UserInDB(User):
    hashed_password: str

# 🔑 Password hashing utility
password_hash = PasswordHash.recommended()

# Dummy hash to prevent timing attacks
DUMMY_HASH = password_hash.hash("fakepassword")

# ✅ OAuth2 scheme with scopes
oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl="token",
    scopes={
        "me": "Read current user data",
        "items": "Read user items",
    },
)

app = FastAPI()

# ✅ Verify password
def verify_password(plain_password, hashed_password):
    return password_hash.verify(plain_password, hashed_password)

# ✅ Get user from DB
def get_user(db, username: str):
    if username in db:
        return UserInDB(**db[username])

# ✅ Authenticate user
def authenticate_user(db, username: str, password: str):
    user = get_user(db, username)
    if not user:
        verify_password(password, DUMMY_HASH)
        return False
    if not verify_password(password, user.hashed_password):
        return False
    return user

# ✅ Create JWT token with scopes
def create_access_token(data: dict, expires_delta: timedelta | None = None):
    to_encode = data.copy()
    expire = datetime.now(timezone.utc) + (
        expires_delta or timedelta(minutes=15)
    )
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

# 🔥 CORE DEPENDENCY — This is where scopes are checked
async def get_current_user(
    security_scopes: SecurityScopes,  # ✅ FastAPI injects required scopes here
    token: Annotated[str, Depends(oauth2_scheme)]  # ✅ Extract token from request
):
    # ✅ Build proper "WWW-Authenticate" header
    if security_scopes.scopes:
        authenticate_value = f'Bearer scope="{security_scopes.scope_str}"'
    else:
        authenticate_value = "Bearer"

    # ❌ Common exception for invalid credentials
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": authenticate_value},
    )

    try:
        # ✅ Decode JWT token
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        # ✅ Extract username
        username = payload.get("sub")
        if username is None:
            raise credentials_exception
        # ✅ Extract scopes from token
        scope: str = payload.get("scope", "")
        token_scopes = scope.split(" ")
        # ✅ Validate token data
        token_data = TokenData(username=username, scopes=token_scopes)
    except (InvalidTokenError, ValidationError):
        raise credentials_exception

    # ✅ Get user from DB
    user = get_user(fake_users_db, username=token_data.username)
    if user is None:
        raise credentials_exception

    # 🔴 IMPORTANT: Scope validation logic
    for scope in security_scopes.scopes:
        if scope not in token_data.scopes:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Not enough permissions",
                headers={"WWW-Authenticate": authenticate_value},
            )

    return user

# ✅ Dependency on top of another dependency
async def get_current_active_user(
    # ✅ Requires "me" scope
    current_user: Annotated[User, Security(get_current_user, scopes=["me"])]
):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user

# 🔑 Login endpoint
@app.post("/token")
async def login_for_access_token(
    form_data: Annotated[OAuth2PasswordRequestForm, Depends()]
) -> Token:
    # ✅ Authenticate user
    user = authenticate_user(
        fake_users_db, form_data.username, form_data.password
    )
    if not user:
        raise HTTPException(status_code=400, detail="Incorrect credentials")
    # ✅ Create token with scopes requested by user
    access_token = create_access_token(
        data={
            "sub": user.username,
            "scope": " ".join(form_data.scopes)  # ✅ Add scopes to JWT
        }
    )
    return {"access_token": access_token, "token_type": "bearer"}

# ✅ Endpoint requiring "me" scope (via dependency)
@app.get("/users/me/")
async def read_users_me(
    current_user: Annotated[User, Depends(get_current_active_user)]
):
    return current_user

# ✅ Endpoint requiring BOTH "me" scope (from dependency) and "items" scope (declared here)
@app.get("/users/me/items/")
async def read_own_items(
    current_user: Annotated[
        User,
        Security(get_current_active_user, scopes=["items"])
    ]
):
    return [{"item_id": "Foo", "owner": current_user.username}]

# ✅ No scope required — only authentication
@app.get("/status/")
async def read_system_status(
    current_user: Annotated[User, Depends(get_current_user)]
):
    return {"status": "ok"}
```

---

## HTTP Basic Auth

```python
import secrets  # ✅ Used for secure string comparison (prevents timing attacks)
from typing import Annotated
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import HTTPBasic, HTTPBasicCredentials

app = FastAPI()

# ✅ Create a "security scheme" for HTTP Basic Auth
security = HTTPBasic()

# ✅ Dependency function to validate username & password
def get_current_username(
    # ✅ FastAPI automatically extracts credentials from Authorization header
    credentials: Annotated[HTTPBasicCredentials, Depends(security)],
):
    # ✅ Convert username to bytes (required for secure comparison)
    current_username_bytes = credentials.username.encode("utf8")
    correct_username_bytes = b"stanleyjobson"

    # ✅ Secure comparison (prevents timing attacks)
    is_correct_username = secrets.compare_digest(
        current_username_bytes, correct_username_bytes
    )

    # ✅ Convert password to bytes
    current_password_bytes = credentials.password.encode("utf8")
    correct_password_bytes = b"swordfish"

    # ✅ Securely compare password
    is_correct_password = secrets.compare_digest(
        current_password_bytes, correct_password_bytes
    )

    # ❌ If credentials are wrong → raise 401 Unauthorized
    if not (is_correct_username and is_correct_password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            # ✅ This header tells the browser to show the login popup again
            headers={"WWW-Authenticate": "Basic"},
        )

    # ✅ If valid → return username
    return credentials.username

# ✅ Protected endpoint
@app.get("/users/me")
def read_current_user(
    # ✅ Uses dependency → only accessible if authentication passes
    username: Annotated[str, Depends(get_current_username)]
):
    return {"username": username}
```

### HTTP Basic Auth Flow

```
CLIENT (Browser / Postman)
        |
        | 1. Request GET /users/me
        |    (No credentials yet)
        ↓
FASTAPI SERVER
        |
        | 2. Depends(security) runs
        |    → HTTPBasic looks for Authorization header
        |
        | ❌ Not found
        ↓
Response: 401 Unauthorized
         WWW-Authenticate: Basic
        |
        ↓
BROWSER
        |
        | 3. Shows login popup (username/password)
        ↓
USER enters credentials
        |
        | 4. Browser sends request again:
        |    Authorization: Basic base64(username:password)
        ↓
FASTAPI SERVER
        |
        | 5. HTTPBasic extracts credentials
        ↓
get_current_username(credentials)
        |
        | 6. Compare username/password securely
        |
        | ✅ If correct → continue
        | ❌ If wrong   → raise 401
        ↓
Endpoint executes: read_current_user()
        |
        ↓
Response: {"username": "stanleyjobson"}
```
