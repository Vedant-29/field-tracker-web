# Field Tracker Web

Admin web console for Field Tracker, a tool for managing a team of field employees. Admins sign in, browse their employees, see each employee's tasks for a given day, and see the team's last known locations on a clustered Google Map.

Related: [field-tracker-mobile](https://github.com/Vedant-29/field-tracker-mobile) (the employee app, which reports location and task status). Both apps share one Supabase project.

## Features

- Email and password sign-up, sign-in and password reset (Supabase Auth)
- Employee directory with a profile page per employee
- Task list per employee, filtered by date and by status (To complete, In Progress, Completed)
- Team map with advanced markers and marker clustering (`@vis.gl/react-google-maps`, `@googlemaps/markerclusterer`)

## Requirements

- Node 18+
- A Supabase project (see Services)
- A Google Maps JavaScript API key and a Map ID (see Services)

## Setup

```sh
git clone https://github.com/Vedant-29/field-tracker-web.git
cd field-tracker-web
npm install
# create .env.local with the variables below
npm run dev
```

Open http://localhost:5173 and go to `/signup` to create the first admin.

Other scripts: `npm run build`, `npm run preview`, `npm run lint`.

## Services

You need your own accounts and keys. Nothing is shipped with the repo.

| Service | Used for | Required | Env vars |
|---|---|---|---|
| Supabase | Auth and the Postgres tables below | Yes | `VITE_REACT_APP_SUPABASE_URL`, `VITE_REACT_APP_SUPABASE_ANON` |
| Google Maps Platform | Team map on the employee profile page | Yes, for the map | `VITE_GOOGLE_MAPS_API_KEY`, `VITE_PUBLIC_MAP_ID` |

### Supabase

1. Create a project at supabase.com. Copy the project URL and anon key from Project Settings > API.
2. Create the tables in the Database section. There are no migrations in the repo.
3. Under Authentication > URL Configuration, set the Site URL to `http://localhost:5173` (or your deployed URL). Password reset emails link there, because the code passes no `redirectTo`.
4. Sign-up sends the admin straight to `/employee-list`. For local testing, turn off Confirm email under Authentication > Providers > Email, or confirm the email and then sign in.
5. Use the same project for the mobile app.

### Google Maps

1. In Google Cloud console, enable the Maps JavaScript API and create an API key under APIs & Services > Credentials.
2. Create a JavaScript Map ID under Google Maps Platform > Map management. Advanced markers do not render without it.
3. Restrict the key by HTTP referrer to `http://localhost:5173/*` and your deployed domain.

Without these the map does not load. The rest of the console still works.

## Environment variables

Put these in `.env.local` at the repo root. There is no `.env.example`.

| Variable | Required | What it is for | Where to get it |
|---|---|---|---|
| `VITE_REACT_APP_SUPABASE_URL` | Yes | Supabase project URL | Supabase dashboard > Project Settings > API |
| `VITE_REACT_APP_SUPABASE_ANON` | Yes | Supabase anon key | Supabase dashboard > Project Settings > API |
| `VITE_GOOGLE_MAPS_API_KEY` | Yes | Google Maps JS API | Google Cloud console > APIs & Services > Credentials |
| `VITE_PUBLIC_MAP_ID` | Yes | Map ID for advanced markers | Google Cloud console > Google Maps Platform > Map management |

These values are bundled into the browser build. Set up row level security policies in Supabase before deploying.

## Database

The app reads and writes these Supabase tables:

| Table | Purpose |
|---|---|
| `admin_users` | One row per admin, written on sign-up: `admin_id` (auth user id), `email`, `admin_subrole` |
| `user_profiles` | The admin's own profile on `/profile`, looked up by `id` (auth user id), shows `user_name` |
| `employee_users` | Field employees: `employee_id` (auth user id), `name`, `email`, `phoneNo`, `role`, `role_assigned_by` (admin id), `latitude`, `longitude` |
| `employee_tasks` | Tasks: `assigned_to_id`, `status`, `completion_date`, `created_at`, `location_name`, `location_poc_name`, `location_poc_email`, `location_poc_phoneNo`, `location_map_link`, `latitude`, `longitude` |

Things to set by hand:

- Give `admin_users.admin_subrole` a default of `pending`. Sign-in sends an admin to `/employee-list` only when it is `pending`. Any other value goes to `/admin`, which does not exist.
- An employee appears on an admin's team map only when `employee_users.role_assigned_by` is that admin's user id. Neither app sets it, and neither sets `role`.
- `employee_tasks.status` must be exactly `To complete`, `In Progress` or `Completed`. Tasks are matched to a day by `completion_date`.
- Tasks are created in the Supabase table editor. The Add Task button has no handler yet.

Location and task status are updated by the mobile app.

## Deployment

`npm run build` writes a static site to `dist/`. Routing is client-side, so the host must rewrite all paths to `index.html`.

## Notes

- `src/utils/ProtectedRoutes.jsx` exists but is not wired into the routes, so pages are not guarded in the browser. Access control depends on Supabase RLS.
- The employee list shows every row in `employee_users`, not only the signed-in admin's team.
- On the employee profile page the map is centred on fixed coordinates, phone numbers get a hardcoded `+91` prefix, and the Targets Completed and Targets Pending counts are placeholders.
- `/test` and `/maps-test` (`src/pages/TestGoogleMaps/`) are leftover demo pages from building the map view.
- Built with React 18, Vite 5, React Router 6, Tailwind and Material UI.

## License

No license file. All rights reserved.
