# Calorie Calculator

This app lets a signed-in user enter a list of foods, looks up calorie and protein values from the USDA FoodData Central API, and stores each calculation in Postgres for later review.

## Features

- Multi-user sign up and sign in with email + password
- Per-user calorie history
- Per-item and total calorie calculation
- Per-item and total protein calculation
- Vercel-friendly deployment setup

## Local setup

1. Create a local environment file:

```bash
cp .env.example .env.local
```

If `.env.example` is not present, create `.env.local` manually.

2. Add these values:

```env
DATABASE_URL="postgresql://user:password@host:5432/database"
USDA_API_KEY="your_usda_key"
AUTH_SECRET="long_random_secret"
```

3. Install dependencies and create the database tables:

```bash
npm install
npx prisma db push
npm run dev
```

4. Open the app at http://localhost:3000 and create an account.

Example food input:

```text
2 eggs, 100g oats, banana
```

## Architecture

```mermaid
flowchart LR
    U[User] --> UI[Next.js App UI]
    UI --> API["/api/calculate\nPOST"]
    UI --> HIST["/api/history\nGET"]
    API --> USDA[USDA FoodData Central API]
    API --> DB[(PostgreSQL / Prisma)]
    HIST --> DB
    UI --> AUTH[JWT Session Cookie]
    AUTH --> DB
```

## How it works

1. The user signs up or logs in with an email and password.
2. A JWT session cookie is created and stored in the browser.
3. The user enters a food list such as `2 eggs, 100g oats, banana`.
4. The app parses the list, sends each item to USDA, and fetches calorie and protein values per 100 g.
5. The values are scaled to the entered grams and summarized into per-item and total results.
6. Each calculation is saved to PostgreSQL with the authenticated user ID.
7. The history page fetches only the current user’s saved calculations.

## Deployment on Vercel

1. Push this repo to GitHub.
2. Import the project in Vercel.
3. Add these environment variables in Vercel project settings:
   - `DATABASE_URL`
   - `USDA_API_KEY`
   - `AUTH_SECRET`
4. Deploy the project.
5. If you need to update the Prisma schema in production, run:

```bash
npx prisma db push
```

Use a hosted PostgreSQL database such as Prisma Postgres, Neon, or Supabase. Do not use `localhost` in production.

## Future improvements

- Add charts for calories and protein trends across multiple days.
- Let users save favorite meals or custom food entries.
- Add daily calorie and protein targets for each user.
- Support meal grouping such as breakfast, lunch, dinner, and snacks.
- Improve USDA matching accuracy with stronger food disambiguation rules.

## Notes

- The app stores each calculation under the authenticated user.
- The `Calculation` model includes `totalProtein` and item-level `proteinGrams` values for new records.
- Older rows may have `null` protein values if they were created before the protein feature was added.

## Screenshots

![alt text](<Screenshot 2026-09-06 at 9.11.04 PM.png>)
![alt text](<Screenshot 2026-09-06 at 9.11.17 PM.png>)
![alt text](<Screenshot 2026-09-06 at 9.12.24 PM.png>)
![alt text](<Screenshot 2026-09-06 at 9.12.55 PM.png>)
