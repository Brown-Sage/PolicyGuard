# PolicyGuard Frontend

This directory contains the React/Vite frontend for PolicyGuard. For the full project overview, backend setup, API routes, and architecture notes, see the root [`../README.md`](../README.md).

## What Is Used

- React 19
- Vite 7
- Tailwind CSS
- ESLint
- Browser `localStorage` for JWT token, username, and theme preference

## Main Screens

- `AuthPage.jsx`: login and account creation.
- `Dashboard.jsx`: policy PDF upload, employee CSV upload, scan trigger, scan summary, and scan history.
- `ViolationsPage.jsx`: global violation listing with refresh.
- `PolicyDocuments.jsx`: MVP placeholder for future policy repository management.
- `Sidebar.jsx`: navigation, dark mode toggle, and logout.

## Setup

```bash
npm install
```

Create `.env` only if the backend API is not running at the default URL:

```env
VITE_API_URL=http://localhost:8000/api
```

Start the development server:

```bash
npm run dev
```

Build for production:

```bash
npm run build
```

Run linting:

```bash
npm run lint
```

## Sample Files

You can use these local files when running a demo scan:

- `Global_Policy_V2.pdf`
- `Global_Policy_V3.pdf`
- `Policy_Compliance_Dataset_Updated.csv`
