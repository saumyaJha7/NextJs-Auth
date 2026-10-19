# Next.js Auth System

A full-stack authentication system built with Next.js (App Router), MongoDB, Mongoose, JWT, and Nodemailer.

## Features

- User signup with duplicate email check
- Email verification via a tokenized link (Mailtrap SMTP in development)
- Login with JWT access and refresh tokens
- Tokens stored in `httpOnly` cookies
- Protected profile page
- Logout that clears cookies and invalidates the stored refresh token
- Route-level protection for `/profile`

## Tech Stack

| Layer | Technology |
| --- | --- |
| Framework | Next.js 16 (App Router) |
| Language | TypeScript |
| Database | MongoDB via Mongoose |
| Auth | `jsonwebtoken` (access + refresh tokens) |
| Email | Nodemailer |
| Styling | Tailwind CSS v4 |
| HTTP client | Axios |

## Getting Started

### 1. Install dependencies

```bash
npm install
```

### 2. Configure environment variables

Create a `.env.local` file in the project root:

```env
MONGO_URI=mongodb+srv://<user>:<password>@<cluster>/<db>
DOMAIN=http://localhost:3000

ACCESS_TOKEN_SECRET=<random-secret>
ACCESS_TOKEN_EXPIRY=1h

REFRESH_TOKEN_SECRET=<random-secret>
REFRESH_TOKEN_EXPIRY=180d
```

For email delivery, provide SMTP credentials in `src/utils/mail.ts` (or better, move them to environment variables):

```env
SMTP_HOST=sandbox.smtp.mailtrap.io
SMTP_PORT=2525
SMTP_USER=<mailtrap-user>
SMTP_PASS=<mailtrap-password>
MAIL_FROM=<sender-address>
```

### 3. Run the development server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

### Available scripts

```bash
npm run dev    # start development server
npm run build  # create production build
npm run start  # run production build
npm run lint   # run ESLint
```

## Project Structure

```
src/
├── app/
│   ├── api/auth/
│   │   ├── signup/route.ts    # POST - register user & send verification email
│   │   ├── login/route.ts     # POST - authenticate & set JWT cookies
│   │   ├── logout/route.ts    # POST - clear session & refresh token
│   │   ├── profile/route.ts   # GET  - return current user
│   │   └── verify/route.ts    # POST - verify email with token
│   ├── login/page.tsx         # Login form
│   ├── signup/page.tsx        # Signup form
│   ├── verify/page.tsx        # Email verification page
│   ├── profile/page.tsx       # Protected profile page
│   ├── layout.tsx
│   └── page.tsx               # Landing page
├── models/
│   └── user.model.ts          # Mongoose user schema
├── utils/
│   ├── db.ts                  # MongoDB connection helper
│   └── mail.ts                # Token generation & email sending
└── proxy.ts                   # Route guard for /profile
```

## API Reference

### `POST /api/auth/signup`

Creates a user and sends a verification email.

```json
{ "username": "johndoe", "email": "john@example.com", "password": "secret" }
```

### `POST /api/auth/login`

Authenticates a user and sets `accessToken` and `refreshToken` cookies.

```json
{ "email": "john@example.com", "password": "secret" }
```

### `GET /api/auth/profile`

Returns the authenticated user (password and refresh token excluded). Requires an `accessToken` cookie.

### `POST /api/auth/logout`

Clears the auth cookies and removes the stored refresh token. Requires an `accessToken` cookie.

### `POST /api/auth/verify`

Verifies the email using the token from the verification link.

```json
{ "token": "<token-from-email-link>" }
```

## Auth Flow

1. **Signup** — user data is saved, a random token is generated, hashed with SHA-256, stored with a 10-minute expiry, and emailed as a link.
2. **Verify** — the raw token from the link is hashed and matched against the stored value. On success `isVerified` is set to `true`.
3. **Login** — credentials are checked, an access token (short-lived) and refresh token (long-lived) are signed and stored in cookies.
4. **Protected route** — `proxy.ts` redirects unauthenticated users away from `/profile`.
5. **Logout** — cookies are deleted and the refresh token is removed from the user document.

## Notes / Known Limitations

- Passwords are currently stored in plain text. They should be hashed with `bcryptjs` before saving and compared with `bcrypt.compare` on login.
- SMTP credentials are hard-coded in `src/utils/mail.ts` and should be moved to environment variables.
- The logout route sets `isVerified: false`; typically only the session/refresh token should be cleared.
- `src/proxy.ts` guards `/profile`, but Next.js middleware conventionally lives in `middleware.ts` (project root or `src/`), so confirm the guard is actually applied.
- The secret values present locally should be rotated if they have ever been shared or committed.
- Password reset tokens exist in the schema, but there is no reset endpoint or page yet.
