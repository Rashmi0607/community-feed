# Community Feed - React Native + Node/Express + MongoDB

This is your uploaded project, completed and upgraded: a working JWT auth
system, a full Express/MongoDB backend, and a redesigned React Native
frontend — built on top of the bare React Native CLI app (with `ios/` and
`android/` folders) you already had.

## Important: rotate your MongoDB credentials

Your uploaded `backend/.env` contained a **live MongoDB Atlas connection
string with a real username and password**. I removed that file from this
project and replaced it with `backend/.env.example` (placeholders only) —
but since the real credentials were already in a file you shared, you
should treat them as compromised:

1. Go to MongoDB Atlas → Database Access → edit or delete that database
   user, and create a new one with a new password.
2. Update your **local, un-shared** `backend/.env` with the new connection
   string. Never commit or share `.env` — it's already in `.gitignore`.

## What was broken in the original upload

- `App.js` imported `./navigation/AppNavigator`, but the file actually
  lived at `frontend/src/navigation/AppNavigator.js` — the app couldn't
  resolve the import.
- `HomeScreen` navigated to a screen named `CreatePostScreen`, but the
  file was `CreatePost.js` exporting `CreatePostScreen` — navigation
  would fail to find the route.
- `backend/app.js` required `./routes/postRoutes` and
  `./routes/groupRoutes`, but the actual files were `posts.js` and
  `groups.js` — the server would crash on boot.
- `backend/config/db.js` was nested inside `backend/controllers/config/`.
- There was no authentication at all: no `User` model, no login, no way
  to know who posted, and no real way to trace an anonymous post back to
  an account (the assignment's moderation requirement).
- No like/unlike endpoint despite the frontend having no bonus feature.

All of that is fixed below.

## What's new

### Backend (Express + MongoDB/Mongoose)
- **JWT authentication**: `/api/auth/register`, `/api/auth/login`,
  `/api/auth/guest` (anonymous session, equivalent to Firebase Anonymous
  Auth), `/api/auth/me`.
- **`requireAuth` middleware** protects posts/groups routes and always
  resolves the real `req.user`, so a post's `userId` is authentic even
  when `isAnonymous` is true.
- **Posts**: `GET /api/posts` (Home Feed), `GET /api/posts/group/:groupId`
  (Group Feed — same collection, just filtered, so nothing is
  duplicated), `POST /api/posts` (create), `PATCH /api/posts/:id/like`
  (bonus like/unlike toggle).
- **Groups**: seeded automatically on boot (`Fitness`, `Travel`, `Food`).
- Input validation, consistent error responses, and a serializer that
  never leaks password hashes.

### Frontend (React Native)
- **Auth flow**: Login, Signup, and "Continue as Guest" screens, with the
  session (JWT) persisted via `AsyncStorage` so users stay logged in.
- **Design system** (`src/theme/theme.js`): shared colors, spacing,
  typography, and per-group accent colors, so every screen looks
  consistent instead of default `<Button>`/inline styles.
- **Reusable components**: `Button`, `Input`, `Avatar` (initials, colored
  per user), `GroupChip`, `EmptyState`, and a redesigned `PostCard` with
  relative timestamps ("3m ago") and an optimistic **like button**.
- **Bottom-tab navigation** (Home / Create Post) with a stack screen for
  Group Feed and a Log Out button in the header.
- Pull-to-refresh on both Home and Group feeds; loading and empty states
  everywhere instead of blank screens.

## Firestore-equivalent schema (MongoDB)

```
users
  _id, name, email (unique, sparse), passwordHash, isAnonymous, createdAt

groups
  _id (slug: "fitness" | "travel" | "food"), name

posts
  _id, userId (ref User — always the real author), userName ("Anonymous
  User" snapshot when isAnonymous), groupId, groupName, content,
  isAnonymous, likes: [userId], createdAt
```

Home Feed and Group Feed both query the **same** `posts` collection
(unfiltered vs. `groupId`-filtered) — a post is written once and never
copied.

## Setup

### 1. Backend
```bash
cd backend
cp .env.example .env
# edit .env: put in your NEW Mongo URI and a random JWT_SECRET
cd ..
npm install
npm run server
```
The server seeds `Fitness`, `Travel`, `Food` groups on first boot and
listens on the `PORT` from `.env` (default `8081`).

### 2. Frontend
```bash
npm install
npx pod-install ios   # first time only, macOS + iOS
npm run start         # Metro bundler
```
In another terminal:
```bash
npm run android   # or
npm run ios
```

### 3. Point the app at your backend
`frontend/src/config.js` picks the right host automatically for
emulators (`10.0.2.2` on Android, `localhost` on iOS simulator). If
you're running on a **physical device**, replace that with your
computer's LAN IP, e.g.:
```js
export const API_BASE_URL = 'http://192.168.1.23:8081/api';
```

## Bonus implemented

**Like button** — tap the heart on any post (Home or Group feed). It
updates optimistically and syncs with the server, reflecting the same
`likes` array stored on the post document — plus pull-to-refresh on both
feeds as a small extra.
