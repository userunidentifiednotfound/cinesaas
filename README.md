# 🎬 CineSaaS — Multi-Tenant Theatre Booking Platform

A production-ready, real-time, multi-tenant theatre booking SaaS platform.

## Architecture

```
theatre-saas/
├── backend/     Node.js + Express + Socket.io + PostgreSQL + Prisma
└── frontend/    React + Vite + TailwindCSS + Socket.io-client + Zustand
```

## Features

- **Multi-tenant**: Each theatre is isolated with its own screens, layouts, pricing
- **Multi-screen**: Each theatre supports unlimited screens (Audi 1, Audi 2, etc.)
- **Visual Layout Builder**: Drag-paint seat grids with aisles, golden seats, accessible seats
- **Real-time seat locking**: WebSocket-based 5-minute holds, scoped to `(screen_id + show_id)`
- **Dynamic pricing**: Base / Weekend / Peak pricing per seat type per screen
- **QR code tickets**: Auto-generated on booking confirmation
- **Admin dashboard**: Theatre overview, screen analytics, live seat monitor
- **Seat heatmap analytics**: Per-screen occupancy and revenue tracking

## Quick Start

### Prerequisites
- Node.js 18+
- PostgreSQL 14+

### 1. Database Setup

```bash
# Create database
createdb theatre_saas

# Or via psql:
psql -U postgres -c "CREATE DATABASE theatre_saas;"
```

### 2. Backend

```bash
cd backend
cp .env.example .env
# Edit .env with your DATABASE_URL

npm install
npx prisma migrate dev --name init
npm run db:seed
npm run dev
```

Backend runs on: http://localhost:5000

### 3. Frontend

```bash
cd frontend
npm install
npm run dev
```

Frontend runs on: http://localhost:5173

## Demo Credentials

| Role | Email | Password |
|------|-------|----------|
| Theatre Admin | admin@theatre.com | admin123 |
| User | user@theatre.com | user123 |

## Data Model

```
Theatre
  └── Screens (Audi 1, Audi 2, ...)
        ├── Seats (unique layout per screen)
        │     ├── SeatType (Normal/Premium/VIP/Gold)
        │     └── ScreenPricing (base/weekend/peak)
        └── Shows
              ├── Movie
              ├── Bookings
              │     ├── BookingSeats
              │     └── Payment
              └── SeatHolds (real-time, 5-min TTL)
```

## API Endpoints

### Auth
- `POST /api/auth/register`
- `POST /api/auth/login`
- `GET  /api/auth/me`

### Theatres
- `GET  /api/theatres`
- `POST /api/theatres`
- `GET  /api/theatres/admin/mine`
- `PUT  /api/theatres/:id`

### Screens
- `GET  /api/screens/theatre/:theatreId`
- `POST /api/screens`
- `POST /api/screens/:id/layout`
- `POST /api/screens/:id/pricing`

### Shows
- `GET  /api/shows/screen/:screenId?date=`
- `GET  /api/shows/movie/:movieId?date=&city=`
- `GET  /api/shows/:id/seats`
- `POST /api/shows`
- `PATCH /api/shows/:id/toggle`

### Bookings
- `POST /api/bookings`
- `GET  /api/bookings/my`
- `GET  /api/bookings/:ref`
- `PATCH /api/bookings/:id/cancel`

### Analytics
- `GET /api/analytics/theatre/:theatreId`
- `GET /api/analytics/screen/:screenId`

## WebSocket Events

| Event | Direction | Description |
|-------|-----------|-------------|
| `show:join` | Client→Server | Subscribe to show room |
| `seats:hold` | Client→Server | Hold seats for 5 min |
| `seats:release` | Client→Server | Release held seats |
| `seats:held` | Server→Client | Broadcast hold to room |
| `seats:booked` | Server→Client | Broadcast booking to room |
| `seats:released` | Server→Client | Broadcast release to room |
| `seats:holdExpired` | Server→Client | Broadcast expired holds |

## Security

- JWT authentication on all protected routes
- Theatre admin can only manage their own theatres
- Seat holds scoped to `(seatId + showId)` — no cross-screen conflicts
- DB transactions prevent double-booking race conditions
- Hold cleanup runs every 30 seconds server-side
# cinesaas
