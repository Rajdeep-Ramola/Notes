# Security — OAuth2, JWT & Authentication

## Simple Authentication Example

This example uses the password flow concept (not encouraged in modern systems). Basically, we send credentials to a client which uses them to get a token. In FastAPI, we implement the password flow through `OAuth2PasswordBearer`.

```python
from typing import Annotated
from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

@app.get("/items/")
async def read_items(token: Annotated[str, Depends(oauth2_scheme)]):
    return {"token": token}
```

---

## Getting the Currently Authenticated User via Dependency Injection

Each request in a web app is stateless. To ensure the next request is by the same user (for authorization and personalization), we have to verify the user at each request.

Instead of directly handling authentication in every endpoint, we can centralize it in reusable dependencies — write authentication logic once, reuse it across any number of endpoints.

```python
from typing import Annotated
from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer
from pydantic import BaseModel

app = FastAPI()

# Tells FastAPI to expect a Bearer token in the request
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None

def fake_decode_token(token):
    """
    In real applications:
    - Decode JWT token
    - Fetch user from database
    - Validate credentials
    Here we just simulate returning a user
    """
    return User(
        username=token + "fakedecoded",
        email="john@example.com",
        full_name="John Doe"
    )

async def get_current_user(
    token: Annotated[str, Depends(oauth2_scheme)]
):
    """
    Steps:
    1. FastAPI extracts token using oauth2_scheme
    2. This token is passed here
    3. We convert token -> User
    """
    user = fake_decode_token(token)
    return user

@app.get("/users/me")
async def read_users_me(
    current_user: Annotated[User, Depends(get_current_user)]
):
    return current_user
```

---

## Simple OAuth2 with Password and Bearer

**Flow:**
1. User sends username + password → `/token`
2. Server verifies credentials
3. Server returns `access_token`
4. User sends token in `Authorization` header
5. Server extracts token → gets user
6. Protected route returns user data

```python
from typing import Annotated
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel

fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "fakehashedsecret",
        "disabled": False,
    },
    "alice": {
        "username": "alice",
        "full_name": "Alice Wonderson",
        "email": "alice@example.com",
        "hashed_password": "fakehashedsecret2",
        "disabled": True,
    },
}

app = FastAPI()

def fake_hash_password(password: str):
    return "fakehashed" + password  # 🔐 Simulated password hashing (not real security)

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")  # 🔐 Extracts Bearer token from Authorization header

class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None

class UserInDB(User):
    hashed_password: str

def get_user(db, username: str):
    if username in db:
        user_dict = db[username]
        return UserInDB(**user_dict)

def fake_decode_token(token):
    user = get_user(fake_users_db, token)  # 🔐 Converts token → user (fake decoding)
    return user

async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    # 🔐 Dependency: gets token from request and resolves current user
    user = fake_decode_token(token)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Not authenticated",
            headers={"WWW-Authenticate": "Bearer"},  # 🔐 Required OAuth2 header
        )
    return user

async def get_current_active_user(
    current_user: Annotated[User, Depends(get_current_user)],
):
    # 🔐 Additional security layer: ensures user is active
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user

@app.post("/token")
async def login(form_data: Annotated[OAuth2PasswordRequestForm, Depends()]):
    # 🔐 Receives username & password via OAuth2 form data
    user_dict = fake_users_db.get(form_data.username)
    if not user_dict:
        raise HTTPException(status_code=400, detail="Incorrect username or password")
    user = UserInDB(**user_dict)
    hashed_password = fake_hash_password(form_data.password)
    if not hashed_password == user.hashed_password:
        raise HTTPException(status_code=400, detail="Incorrect username or password")
    return {
        "access_token": user.username,  # 🔐 Returns token (fake, should be JWT in real apps)
        "token_type": "bearer",         # 🔐 Required OAuth2 token type
    }

@app.get("/users/me")
async def read_users_me(
    current_user: Annotated[User, Depends(get_current_active_user)],
):
    # 🔐 Protected route: only accessible with valid & active user
    return current_user
```

---

## JWT (JSON Web Tokens)

- A standard to codify a JSON object in a long dense string without spaces.
- It is **not encrypted but is signed**, so when you receive a token that you issued, you can verify that it was you who issued it.
- We can create a token with an expiration; after expiry, the user will be required to sign in again.

**Dependencies:**
- **PyJWT** — to generate and verify JWT tokens in Python
- **Pwdlib** — Python package to handle password hashes

```python
from datetime import datetime, timedelta, timezone
from typing import Annotated
import jwt
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jwt.exceptions import InvalidTokenError
from pwdlib import PasswordHash
from pydantic import BaseModel

# 🔐 Secret key + algorithm used to SIGN and VERIFY JWT tokens
SECRET_KEY = "09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        # 🔐 Stored hashed password (NOT plaintext)
        "hashed_password": "$argon2id$v=19$m=65536,t=3,p=4$wagCPXjifgvUFBzq4hqe3w$CYaIb8sB+wtD+Vu/P4uod1+Qof8h+1g7bbDlBID48Rc",
        "disabled": False,
    }
}

class Token(BaseModel):
    access_token: str
    token_type: str

class TokenData(BaseModel):
    username: str | None = None  # 🔐 Extracted from JWT payload

class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None

class UserInDB(User):
    hashed_password: str

# 🔐 Password hashing utility (Argon2 via pwdlib)
password_hash = PasswordHash.recommended()

# 🔐 Dummy hash to mitigate timing attacks when user not found
DUMMY_HASH = password_hash.hash("dummypassword")

# 🔐 Extracts Bearer token from Authorization header
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

app = FastAPI()

def verify_password(plain_password, hashed_password):
    # 🔐 Compares plain password with stored hash (secure)
    return password_hash.verify(plain_password, hashed_password)

def get_password_hash(password):
    # 🔐 Converts plaintext → hashed password for safe storage
    return password_hash.hash(password)

def get_user(db, username: str):
    if username in db:
        user_dict = db[username]
        return UserInDB(**user_dict)

def authenticate_user(fake_db, username: str, password: str):
    user = get_user(fake_db, username)
    if not user:
        # 🔐 Prevents user enumeration attacks (timing attacks)
        verify_password(password, DUMMY_HASH)
        return False
    # 🔐 Verify input password against stored hashed password
    if not verify_password(password, user.hashed_password):
        return False
    return user

def create_access_token(data: dict, expires_delta: timedelta | None = None):
    to_encode = data.copy()
    # 🔐 Set token expiry time
    if expires_delta:
        expire = datetime.now(timezone.utc) + expires_delta
    else:
        expire = datetime.now(timezone.utc) + timedelta(minutes=15)
    to_encode.update({"exp": expire})  # 🔐 Add expiry claim to JWT
    # 🔐 Encode and sign JWT using secret key
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    # 🔐 Standard OAuth2 error when token invalid
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        # 🔐 Decode and verify JWT signature + expiry
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username = payload.get("sub")  # 🔐 Extract "subject" (username)
        if username is None:
            raise credentials_exception
        token_data = TokenData(username=username)
    except InvalidTokenError:
        # 🔐 Token invalid, expired, or tampered
        raise credentials_exception
    # 🔐 Fetch user from DB using token data
    user = get_user(fake_users_db, username=token_data.username)
    if user is None:
        raise credentials_exception
    return user

async def get_current_active_user(
    current_user: Annotated[User, Depends(get_current_user)],
):
    # 🔐 Additional security layer (business rule)
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user

@app.post("/token")
async def login_for_access_token(
    form_data: Annotated[OAuth2PasswordRequestForm, Depends()],
) -> Token:
    # 🔐 Authenticate user using username + password
    user = authenticate_user(fake_users_db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    # 🔐 Define token expiration duration
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    # 🔐 Create JWT with "sub" claim (standard = subject/user id)
    access_token = create_access_token(
        data={"sub": user.username}, expires_delta=access_token_expires
    )
    # 🔐 Return JWT in OAuth2 compliant format
    return Token(access_token=access_token, token_type="bearer")

@app.get("/users/me/")
async def read_users_me(
    current_user: Annotated[User, Depends(get_current_active_user)],
) -> User:
    # 🔐 Only accessible with valid + active JWT token
    return current_user

@app.get("/users/me/items/")
async def read_own_items(
    current_user: Annotated[User, Depends(get_current_active_user)],
):
    # 🔐 User-specific data protected by authentication
    return [{"item_id": "Foo", "owner": current_user.username}]
```

---

## Advanced Usage with Scopes

OAuth2 has a notion of `scopes` which can be used to add a specific set of permissions to a JWT token. We can give this token to a user directly or a third party, to interact with the API with a set of restrictions.
