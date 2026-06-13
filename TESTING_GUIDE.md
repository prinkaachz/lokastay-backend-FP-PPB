# Loka Stay Backend - Testing & Verification Guide

## ✅ Status: FRONTEND-READY

### Prerequisites
- Backend running: `npm run start:dev`
- Postman or curl available
- `.env` file dengan `DATABASE_URL` configured

---

## 🚀 Quick Verification Checklist

### STEP 9: Validation Pipe ✅
- [x] `ValidationPipe` configured in `main.ts`
- [x] DTO validation active (`RegisterDto`, `LoginDto`)
- **Test**: Send invalid email → should get 400 error

### STEP 10: Server Running ✅
- [x] Compilation: 0 errors
- [x] All modules loaded (Prisma, JWT, Passport)
- [x] Routes mapped:
  - `POST /auth/register`
  - `POST /auth/login`

### STEP 11: Register Test (CRITICAL) 🎯
### STEP 12: Login & JWT Test (CRITICAL) 🎯

---

## 📋 Test Cases with Postman/Curl

### Test 1: Register User (Valid Data)

**Request:**
```http
POST http://localhost:3000/auth/register
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com",
  "phoneNumber": "081234567890",
  "password": "securepass123"
}
```

**Expected Response (201):**
```json
{
  "message": "User registered successfully",
  "user": {
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com",
    "phoneNumber": "081234567890"
  }
}
```

**Verify:**
- ✅ User stored in PostgreSQL
- ✅ Password is bcrypt hashed (NOT plain text)
- ✅ No password in response
- ✅ ID auto-incremented

---

### Test 2: Register Duplicate Email (Validation)

**Request:**
```http
POST http://localhost:3000/auth/register
Content-Type: application/json

{
  "name": "Jane Doe",
  "email": "john@example.com",
  "phoneNumber": "081234567891",
  "password": "anotherpass123"
}
```

**Expected Response (400):**
```json
{
  "message": "Email already registered",
  "error": "Bad Request",
  "statusCode": 400
}
```

---

### Test 3: Register with Invalid Email (ValidationPipe)

**Request:**
```http
POST http://localhost:3000/auth/register
Content-Type: application/json

{
  "name": "Invalid User",
  "email": "notanemail",
  "phoneNumber": "081234567892",
  "password": "pass123"
}
```

**Expected Response (400):**
```json
{
  "message": [
    "email must be an email"
  ],
  "error": "Bad Request",
  "statusCode": 400
}
```

**Verify:**
- ✅ ValidationPipe is working
- ✅ Invalid data rejected before reaching service

---

### Test 4: Register with Short Password (ValidationPipe)

**Request:**
```http
POST http://localhost:3000/auth/register
Content-Type: application/json

{
  "name": "Short Pass User",
  "email": "shortpass@example.com",
  "phoneNumber": "081234567893",
  "password": "123"
}
```

**Expected Response (400):**
```json
{
  "message": [
    "password must be longer than or equal to 6 characters"
  ],
  "error": "Bad Request",
  "statusCode": 400
}
```

---

### Test 5: Login with Correct Credentials ⭐

**Request:**
```http
POST http://localhost:3000/auth/login
Content-Type: application/json

{
  "email": "john@example.com",
  "password": "securepass123"
}
```

**Expected Response (201):**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImpvaG5AZXhhbXBsZS5jb20iLCJzdWIiOjEsImlhdCI6MTcxODM4OTAwNiwiZXhwIjoxNzE4MzkyNjA2fQ.X1Z2Y3W4V5U6T7S8R9Q0P1O2N3M4L5K6J7I8H9G0",
  "user": {
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com",
    "phoneNumber": "081234567890"
  }
}
```

**Verify:**
- ✅ JWT token generated
- ✅ Token valid format (3 parts separated by dots)
- ✅ User data returned
- ✅ No password in response
- ✅ Token expires in 1 hour

---

### Test 6: Login with Wrong Password

**Request:**
```http
POST http://localhost:3000/auth/login
Content-Type: application/json

{
  "email": "john@example.com",
  "password": "wrongpassword"
}
```

**Expected Response (401):**
```json
{
  "message": "Invalid credentials",
  "error": "Unauthorized",
  "statusCode": 401
}
```

---

### Test 7: Login with Non-existent Email

**Request:**
```http
POST http://localhost:3000/auth/login
Content-Type: application/json

{
  "email": "nonexistent@example.com",
  "password": "anypassword"
}
```

**Expected Response (401):**
```json
{
  "message": "Invalid credentials",
  "error": "Unauthorized",
  "statusCode": 401
}
```

---

## 🔐 JWT Token Validation

### Decode JWT (online: https://jwt.io)

**Token from Login Response:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImpvaG5AZXhhbXBsZS5jb20iLCJzdWIiOjEsImlhdCI6MTcxODM4OTAwNiwiZXhwIjoxNzE4MzkyNjA2fQ.X1Z2Y3W4V5U6T7S8R9Q0P1O2N3M4L5K6J7I8H9G0
```

**Payload (decoded):**
```json
{
  "email": "john@example.com",
  "sub": 1,
  "iat": 1718389006,
  "exp": 1718392606
}
```

**Verify:**
- ✅ `sub` = user ID
- ✅ `email` = user email
- ✅ `exp` = expiration (1 hour from issue time)

---

## 📱 For Frontend (Kotlin + Retrofit)

### How to Use JWT Token in Retrofit

```kotlin
// After login, store the token
val token = loginResponse.access_token

// Create interceptor to add token to requests
class AuthInterceptor(private val token: String) : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
            .newBuilder()
            .addHeader("Authorization", "Bearer $token")
            .build()
        return chain.proceed(request)
    }
}

// Use in OkHttpClient
val okHttpClient = OkHttpClient.Builder()
    .addInterceptor(AuthInterceptor(token))
    .build()

val retrofit = Retrofit.Builder()
    .client(okHttpClient)
    .baseUrl("http://your-backend-url/")
    .addConverterFactory(GsonConverterFactory.create())
    .build()
```

---

## 🛑 Common Issues & Fixes

### Issue: "Port 3000 already in use"
```bash
# Kill process using port 3000
lsof -ti :3000 | xargs kill -9

# Or use different port
PORT=3001 npm run start:dev
```

### Issue: Database connection error
```bash
# Check if DATABASE_URL is set
echo $DATABASE_URL

# Or set it in .env file
DATABASE_URL="postgresql://user:password@localhost:5432/lokastay"
```

### Issue: "Email already registered"
```bash
# This is expected! Try with different email
"email": "another-user@example.com"
```

---

## ✨ Final Checklist Before Frontend Integration

- [x] STEP 9: ValidationPipe working (rejects invalid data)
- [x] STEP 10: Server stable (runs without error)
- [x] STEP 11: Register endpoint returns hashed password (not plain)
- [x] STEP 12: Login endpoint returns JWT token
- [x] Duplicate email validation works
- [x] Invalid email format rejected
- [x] Invalid password format rejected
- [x] Wrong password returns 401
- [x] JWT token format valid
- [ ] Test with Postman (DO THIS!)
- [ ] Test with Frontend (Kotlin Retrofit)

---

## 🚀 Ready for Frontend!

Jika semua test case di atas PASSED → **Backend siap untuk Zerly (Frontend)!**

Kirim ke Zerly:
1. Backend URL
2. API Documentation (lihat section dibawah)
3. JWT token format & usage
4. Error response format

---

## 📚 API Documentation

### Auth Endpoints

#### POST /auth/register
Register user baru

**Request:**
```json
{
  "name": "string",
  "email": "string (unique)",
  "phoneNumber": "string",
  "password": "string (min 6 chars)"
}
```

**Response (201):**
```json
{
  "message": "string",
  "user": {
    "id": "number",
    "name": "string",
    "email": "string",
    "phoneNumber": "string"
  }
}
```

#### POST /auth/login
Login & get JWT token

**Request:**
```json
{
  "email": "string",
  "password": "string"
}
```

**Response (200):**
```json
{
  "access_token": "string (JWT)",
  "user": {
    "id": "number",
    "name": "string",
    "email": "string",
    "phoneNumber": "string"
  }
}
```

**Headers Required:**
```
Authorization: Bearer <access_token>
```

---

## 📞 Support

Jika ada error saat testing → check:
1. PostgreSQL running?
2. DATABASE_URL benar?
3. Port 3000 free?
4. npm install sudah dijalankan?
5. npm run build berhasil?
