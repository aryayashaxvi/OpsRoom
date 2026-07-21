# OpsRoom API

Copy `.env.example` to `.env`, set MongoDB and JWT values, then run `npm install`, `npm run seed`, and `npm run dev`.

The API runs at `http://localhost:5000`. All protected requests use `Authorization: Bearer <token>`. REST mutations persist first and Socket.IO broadcasts confirmed changes afterwards.
