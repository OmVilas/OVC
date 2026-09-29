# Omsvila

A MERN e-commerce app for traditional snacks: browse products, cart, checkout, order history, and an admin dashboard for product/order management.

## Project structure

```
Omsvila/
  client/   React (Vite) frontend
  server/   Express + MongoDB backend
```

## Setup

### 1. Server

```bash
cd Omsvila/server
npm install
cp .env.example .env   # then edit .env with real values
```

Required `.env` variables (see `.env.example`):

| Variable | Description |
|---|---|
| `PORT` | Port the API listens on (default 5000) |
| `MONGO_URI` | MongoDB connection string |
| `JWT_SECRET` | Long random string used to sign JWTs |
| `JWT_EXPIRES_IN` | Token lifetime, e.g. `7d` |
| `ADMIN_EMAIL` / `ADMIN_PASSWORD` | Credentials for the one admin account created by the seed script |
| `CLIENT_URL` | Origin allowed by CORS, e.g. `http://localhost:5173` |

Seed the database with an admin user and sample products, then start the server:

```bash
npm run seed
npm run dev     # nodemon, for local development
npm start       # plain node, for production
```

### 2. Client

```bash
cd Omsvila/client
npm install
npm run dev
```

The Vite dev server proxies `/api` requests to `http://localhost:5000`, so no CORS setup is needed in development.

## API reference

All endpoints are prefixed with `/api`.

### Auth (`/api/auth`)
| Method | Route | Access | Description |
|---|---|---|---|
| POST | `/register` | Public | Create a customer account |
| POST | `/login` | Public | Log in, returns `{ token, user }` |

### Products (`/api/products`)
| Method | Route | Access | Description |
|---|---|---|---|
| GET | `/` | Public | List all products |
| GET | `/:id` | Public | Get one product |
| POST | `/` | Admin | Create a product (`multipart/form-data`: `name`, `price`, `description`, `category`, `stock`, `image` file) |
| PUT | `/:id` | Admin | Update a product (same `multipart/form-data` shape; `image` optional — omit to keep the existing one) |
| DELETE | `/:id` | Admin | Delete a product (also removes its uploaded image file) |

Uploaded images are stored on disk under `server/uploads/` and served at `/uploads/<filename>` (accepts JPEG/PNG/WEBP/GIF, 5MB max).

### Cart (`/api/cart`) — requires a logged-in user
| Method | Route | Description |
|---|---|---|
| GET | `/` | Get the current user's cart |
| POST | `/add` | Add `{ productId, quantity }` to the cart |
| DELETE | `/:productId` | Remove a product from the cart |

### Orders (`/api/orders`) — requires a logged-in user
| Method | Route | Access | Description |
|---|---|---|---|
| POST | `/` | Any user | Place an order from the current cart (`{ shippingAddress }`) |
| GET | `/mine` | Any user | List the current user's orders |
| GET | `/` | Admin | List all orders |
| PUT | `/:orderId/status` | Admin | Update an order's status |

### Admin (`/api/admin`)
| Method | Route | Access | Description |
|---|---|---|---|
| GET | `/dashboard` | Admin | Aggregate stats: users, products, orders, revenue |

Protected routes expect `Authorization: Bearer <token>`.

## Security

- Passwords are hashed with bcrypt; JWTs are used for stateless auth.
- `helmet`, restricted CORS, `express-rate-limit`, and `express-mongo-sanitize` are applied globally.
- All write endpoints validate input with `express-validator`.

## Deployment notes

- Set real values for every variable in `.env` (especially `JWT_SECRET`, `ADMIN_PASSWORD`, and `MONGO_URI` pointing at your production database).
- Set `CLIENT_URL` to your deployed frontend's origin.
- Build the client with `npm run build` (in `client/`) and serve the static output from your host of choice; point `VITE_API_URL` at your deployed API if the client and API aren't served from the same origin.
- If the client and API are on different origins in production, proxy or CORS-enable `/uploads` the same way `/api` is handled (the dev server does this via `vite.config.js`).
- `server/uploads/` needs persistent disk storage — on ephemeral hosts (e.g. most PaaS free tiers) uploaded images will be lost on redeploy unless you mount a persistent volume or switch to object storage (S3, etc.).
