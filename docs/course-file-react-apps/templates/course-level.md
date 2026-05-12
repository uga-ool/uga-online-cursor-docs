# Brightspace React Template — Course Files

[![Vite](https://img.shields.io/badge/Vite-5.x-blue.svg)](https://vitejs.dev/)
[![React](https://img.shields.io/badge/React-19.x-blue.svg)](https://react.dev/)

A starter Vite + React + TypeScript + SWC app designed for deployment in Brightspace **course files**.

Includes:

- Vite configuration for Brightspace static hosting (relative paths, no hashed filenames).
- CourseContext to automatically pull OU, XSRF token, WhoAmI data, and course role.
- Basic API utilities (`fetchData` and `postData`) with Brightspace-friendly defaults.
- Local development fallbacks for OU and other data.
- Minimal starter view for local testing and initial LMS deployment.

---

## 🏗 Project Structure

```plaintext
src/
├── api/            # Brightspace API route helpers
├── assets/
├── components/     # UI components (StarterView included by default)
├── context/        # CourseContext (OU, WhoAmI, Role, XSRF)
├── reducers/       # For apps that need robust state management
├── styles/         # For any styling needed beyond the design system
├── utils/          # Generic utilities (apiUtils, etc.)
├── App.tsx
├── main.tsx
```

---

## 🚀 Getting Started

```bash
npm install
npm run dev
```

**Default dev server:** http://localhost:5173

---

## ⚙ Environment Variables

Create a `.env` file in the root:

```plaintext
VITE_API_BASE_URL=/d2l/api
VITE_LP_VERSION=1.47
VITE_LE_VERSION=1.47
```

---

## ✅ Build for Brightspace

```bash
npm run build
```

Upload the **dist/** folder contents to your Brightspace **course files**.

---

## 📝 Notes

- StarterView provides a simple view for testing CourseContext data. Delete or replace it when building real app views.
- `import.meta.env.DEV` is used to distinguish local development from Brightspace deployment.
