# MaxBlog

MaxBlog is a full-stack personal blog and portfolio application built with React, Express, and MongoDB. It combines a public website for reading articles and viewing profile information with an administration interface for managing posts and site content.

## Features

- Public home page with profile and project information.
- Article list and individual posts with slug-based URLs.
- Markdown editing and rendering with GitHub Flavored Markdown support.
- Article creation, editing, deletion, and draft/published status fields.
- User registration and login using JWT authentication in an HTTP-only cookie.
- Protected administration pages for post management and site settings.
- Configurable blog name, description, profile, and project content.

The application is under development. See [Current limitations](#current-limitations) before using it beyond a local development environment.

## Technology Stack

| Layer          | Technologies                                  |
| -------------- | --------------------------------------------- |
| Frontend       | React 18, Vite 4, React Router 6              |
| UI             | Bootstrap 5, React Bootstrap, Bootstrap Icons |
| Markdown       | EasyMDE, React Markdown, remark-gfm           |
| HTTP client    | Axios                                         |
| Backend        | Node.js, Express 4                            |
| Database       | MongoDB, Mongoose 5                           |
| Authentication | JSON Web Tokens, bcrypt, cookie-parser        |
| Code quality   | ESLint, Prettier                              |

## Project Structure

```text
MaxBlog/
|-- client/
|   |-- public/             # Static assets
|   |-- src/
|   |   |-- components/     # Shared UI components and route guard
|   |   |-- context/        # Authentication state
|   |   |-- features/       # API helpers for auth, articles, and site data
|   |   |-- layout/         # Public and administration layouts
|   |   |-- pages/          # Public, authentication, and administration pages
|   |   |-- utils/          # Shared Axios client
|   |   |-- config.js       # Frontend environment configuration
|   |   `-- routes.jsx      # Frontend routes
|   `-- vite.config.js      # Development server and API proxy
|-- server/
|   |-- config/             # Environment configuration and database setup
|   |-- controllers/        # Request handlers
|   |-- middleware/         # Authentication and error handling
|   |-- models/             # User, Article, and Site schemas
|   |-- routes/             # API routes
|   |-- utils/              # Response and error helpers
|   |-- views/              # EJS templates
|   |-- app.js              # Express application setup
|   `-- server.js           # Server entry point
|-- package.json            # Repository-level formatting and lint scripts
`-- README.md
```

## Getting Started

### Prerequisites

- Node.js and npm. The repository does not currently specify or verify a supported Node.js version.
- A running MongoDB instance, either local or hosted, accessible from the backend.

### 1. Install Dependencies

Run the following commands from the repository root. The root, frontend, and backend have separate package manifests; this project does not use npm workspaces.

```sh
npm install
npm --prefix client install
npm --prefix server install
```

### 2. Configure the Backend

Create `server/.env` with the following values, replacing the example secrets with independent, randomly generated values:

```dotenv
db=mongodb://127.0.0.1:27017/maxblog
sessionSecret=replace-with-a-random-session-secret
secretKey=replace-with-a-random-jwt-signing-secret
NODE_ENV=development
```

| Variable        | Purpose                                                                     |
| --------------- | --------------------------------------------------------------------------- |
| `db`            | MongoDB connection URI; the lowercase name is required by the current code. |
| `sessionSecret` | Secret for Express session middleware.                                      |
| `secretKey`     | Secret used to sign and verify authentication JWTs.                         |
| `NODE_ENV`      | Runtime mode; the server defaults to `development`.                         |

The configuration module also reads `PORT` and `JWT_SECRET`, but the current server entry point listens on **5001**, and authentication uses **`secretKey`**. Setting `PORT` or `JWT_SECRET` alone does not change those behaviors.

The repository contains a file named `server/env`. The default dotenv loader expects `.env`, so create `server/.env` explicitly. Keep credentials out of version control; `.env` is already ignored.

### 3. Configure the Frontend

The default API base URL is `/api`, which works with the Vite development proxy. For an explicit local configuration, create `client/.env`:

```dotenv
VITE_API_BASE_URL=/api
VITE_APP_TITLE=MaxBlog
```

Vite expects filenames such as `.env`, `.env.development`, or `.env.production`. The existing `client/env.development` and `client/env.production` files lack the leading dot and are not loaded automatically. Restart the development server after changing environment files. Frontend `VITE_*` values are exposed to the browser; do not put secrets in them.

### 4. Start the Backend

In one terminal, run from the repository root:

```sh
cd server
npm run dev
```

The backend listens at `http://localhost:5001`. Check the terminal for `DB Connected!` to confirm the database connection; the server can begin listening before the database is ready.

When the `Site` collection is empty, initialization attempts to create default site settings, a sample published article, and a user with username `admin` and password `password`. These credentials are intended for local setup only. Replace them through a verified account-management process before exposing the application; the current account settings page has an API mismatch described below.

### 5. Start the Frontend

In a second terminal, run from the repository root:

```sh
cd client
npm run dev
```

Open `http://localhost:3000`. Vite forwards `/api` requests to `http://localhost:5001`.

| Route                | Purpose                  |
| -------------------- | ------------------------ |
| `/`                  | Home page                |
| `/writing`           | Published article list   |
| `/posts/:slug`       | Individual article       |
| `/login`             | Login                    |
| `/signup`            | Registration             |
| `/admin/dashboard`   | Administration dashboard |
| `/admin/posts`       | Post management          |
| `/admin/post/create` | New post editor          |
| `/admin/setting`     | Site settings            |

## Development Commands

| Directory | Command            | Purpose                                              |
| --------- | ------------------ | ---------------------------------------------------- |
| Root      | `npm run lint`     | Run the repository ESLint configuration.             |
| Root      | `npm run lint:fix` | Apply automatic ESLint fixes.                        |
| Root      | `npm run format`   | Format the repository with Prettier; modifies files. |
| `client`  | `npm run dev`      | Start the Vite development server.                   |
| `client`  | `npm run build`    | Build frontend assets into `client/dist`.            |
| `client`  | `npm run preview`  | Preview the frontend build locally.                  |
| `server`  | `npm run dev`      | Start the backend with nodemon.                      |
| `server`  | `npm start`        | Also start the backend with nodemon.                 |

The root `npm test` command is a placeholder that exits with an error. No automated test suite is currently configured.

## API Overview

All backend routes below are prefixed with `/api`. Authentication uses a cookie named `token`.

| Method                 | Endpoint               | Purpose                                                                 |
| ---------------------- | ---------------------- | ----------------------------------------------------------------------- |
| `POST`                 | `/auth/login`          | Log in with `{ "auth": { "username": "...", "password": "..." } }`.     |
| `GET`                  | `/auth/signout`        | Clear the authentication cookie.                                        |
| `POST`                 | `/users`               | Register a user.                                                        |
| `GET`                  | `/users/me`            | Check login status and return JWT identity data.                        |
| `GET`                  | `/users`               | List users; requires login.                                             |
| `GET`, `PUT`, `DELETE` | `/users/:userId`       | Read, update, or delete a user.                                         |
| `GET`                  | `/articles`            | List articles.                                                          |
| `POST`                 | `/articles`            | Create an article; requires login and an `author` username in the body. |
| `GET`                  | `/articles/:articleId` | Read an article by ID.                                                  |
| `PUT`, `DELETE`        | `/articles/:articleId` | Update or delete an article; requires login.                            |
| `GET`                  | `/posts/:slug`         | Read an article by slug.                                                |
| `GET`                  | `/site`                | Read site settings.                                                     |
| `PUT`                  | `/site`                | Update site settings; requires login.                                   |

## Build and Deployment

Build the frontend from the repository root:

```sh
npm --prefix client run build
```

The output is in `client/dist`. A deployment must provide:

- Static hosting with a fallback to `index.html` for frontend routes.
- A separately running Express backend with access to MongoDB.
- Routing for `/api` requests to the backend, or an appropriate `VITE_API_BASE_URL` set before building.
- HTTPS and compatible cookie settings for production authentication.

The Vite `/api` proxy is a development setting. It does not apply to a built site or automatically configure `npm run preview`. The backend does not currently serve `client/dist`. Separate frontend/backend origins also require explicit credential-aware CORS configuration; the current backend uses default `cors()` settings.

No production deployment procedure has been verified for this README.

## Current Limitations

- The public article list filters published posts in the frontend. The read APIs do not enforce published-only access, so draft status is not a privacy boundary.
- User routes for reading, updating, and deleting by ID lack authentication middleware. Article update and delete routes check login but do not apply the existing ownership middleware. Review these permissions before public deployment.
- The account settings page calls `/api/users/update-profile`, but no matching profile-update handler is defined; the backend instead exposes `/api/users/:userId`.
- Package metadata still contains historical MaxPortfolio names and repository links.
- Installation, startup, and deployment instructions are based on source inspection and have not been validated end to end.

## License

The root `package.json` declares the ISC license. A standalone `LICENSE` file is not currently included.
