# Field Tracker Web

Admin web console for Field Tracker, a tool for managing a team of field employees. Admins sign in, browse their employees, see each employee's tasks for a given day, and view the whole team's last known location on a clustered Google Map.

The employees use the companion mobile app, [field-tracker-mobile](https://github.com/Vedant-29/field-tracker-mobile), which reports their location and task status.

## Features

- Email and password sign-up with email verification and password reset (Supabase Auth)
- Employee directory with a profile page per employee
- Task board per employee, filtered by date and grouped by status
- Team map with marker clustering (`@vis.gl/react-google-maps`)

## Requirements

- Node 18+
- A Supabase project with Auth enabled and the tables listed below
- A Google Maps JavaScript API key and a Map ID (the Map ID is needed for advanced markers)

## Setup

```sh
git clone https://github.com/Vedant-29/field-tracker-web.git
cd field-tracker-web
npm install
# create .env.local with the variables below
npm run dev
```

Open http://localhost:5173.

Other scripts: `npm run build`, `npm run preview`, `npm run lint`.

## Environment variables

Put these in `.env.local` at the repo root. There is no `.env.example`.

| Variable | Required | What it is for | Where to get it |
|---|---|---|---|
| `VITE_REACT_APP_SUPABASE_URL` | Yes | Supabase project URL | Supabase dashboard > Project Settings > API |
| `VITE_REACT_APP_SUPABASE_ANON` | Yes | Supabase anon key | Supabase dashboard > Project Settings > API |
| `VITE_GOOGLE_MAPS_API_KEY` | Yes | Google Maps JS API | Google Cloud console > APIs & Services > Credentials |
| `VITE_PUBLIC_MAP_ID` | Yes | Map ID for vector maps and advanced markers | Google Cloud console > Google Maps Platform > Map management |

The anon key is exposed to the browser, so set up row level security policies in Supabase before deploying.

## Database

The app reads and writes these Supabase tables:

| Table | Purpose |
|---|---|
| `admin_users` | One row per admin, written on sign-up (`admin_id`, `email`) |
| `user_profiles` | The admin's own profile, shown on `/profile` |
| `employee_users` | Field employees: `name`, `email`, `phoneNo`, `role`, `role_assigned_by` (admin id), `latitude`, `longitude` |
| `employee_tasks` | Tasks: `assigned_to_id`, `status`, `completion_date`, `location_name`, `location_poc_name`, `location_poc_email`, `location_poc_phoneNo`, `location_map_link`, `latitude`, `longitude` |

There are no migrations in the repo, so the tables have to be created by hand. Location and task status are updated by the mobile app.

## Notes

- `/test` and `src/pages/TestGoogleMaps/` are leftover demo pages from building the map view.
- Built with React 18, Vite, React Router 6, Tailwind and Material UI.

## License

No license file. All rights reserved.
