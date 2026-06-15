---
name: rn-expo-starter
description: Bootstrap a new Expo React Native project with TypeScript, React Navigation v7, React Native Paper, and Axios. Optionally adds an Express + PostgreSQL backend following the same conventions as react-express-starter. Use when the user wants to start a fresh React Native mobile (or mobile+web) project.
version: 1.0.0
last_updated: 2026-06-15
---

# React Native (Expo) Starter

This skill bootstraps a new Expo project with the conventions, structure, and exact dependency versions used in this repo.

> Only invoke when starting a **new** project. For day-to-day work in an existing project follow `CLAUDE.md` and `AGENTS.md` instead.

---

## Step 0 — Ask the user

Before writing any code, ask **both** of the following questions (can be asked together):

> 1. "Where should the project be created?"
>    - **Current folder** — scaffold directly into the working directory
>    - **Sub-folder** — create a new directory (prompt for the name, e.g. `my-app`)
>
> 2. "Should this project include an Express + PostgreSQL backend? (y / n)"

Then proceed based on the answers:

- Sub-folder + no backend → run `npx create-expo-app <name> --template blank-typescript` then work inside `<name>/`
- Sub-folder + backend → create `<name>/` as the monorepo root, scaffold `<name>/mobile/` and `<name>/server/`
- Current folder + no backend → initialise Expo in `.` (skip the create-expo-app folder step; init manually)
- Current folder + backend → treat `.` as the monorepo root, scaffold `./mobile/` and `./server/`

See §Layout A (mobile only) and §Layout B (mobile + backend) for folder structures.

---

## Cross-skill rules (always active)

These existing skills govern all code written during bootstrapping:

| Skill | Applies to |
|---|---|
| `clean-typescript` | All `.ts` / `.tsx` files — `strict: true`, no `any`, no `!` assertions |
| `modern-react-components` | All React components — no `useEffect` for derived state, named function declarations, Axios hooks |
| `api-structure` | Backend domain layout (if Express chosen) — per-domain controllers, lazy models, `schemaTable()` |
| `web-security` | Trust boundaries — validate inputs, generic error envelopes, no secrets committed |
| `modern-browser-apis` | Web target (`react-native-web`) — prefer native APIs; `Intl.NumberFormat` for formatting |
| `code-review` | Run before declaring the bootstrap done |

---

## Stack

| Layer | Choice | Version |
|---|---|---|
| Runtime | Node | 20.6+ |
| Package manager | npm | — |
| Mobile framework | Expo (managed workflow) | ~54.0.33 |
| React | react | 19.1.0 |
| React Native | react-native | 0.81.5 |
| Language | TypeScript | ~5.9.2 |
| Navigation | @react-navigation/native + native-stack + bottom-tabs | ^7.2.4 / ^7.14.14 / ^7.15.13 |
| HTTP client | axios | ^1.16.0 |
| UI component library | react-native-paper | ^5.15.3 |
| Icons | lucide-react-native + react-native-svg | ^1.14.0 / 15.12.1 |
| Fonts | @expo-google-fonts/geist + geist-mono | ^0.4.1 |
| Storage | @react-native-async-storage/async-storage | 2.2.0 |
| Web compat | react-native-web + @expo/metro-runtime | ~0.21.0 / ~6.1.2 |
| Gradients | expo-linear-gradient | ~15.0.8 |
| Safe area | react-native-safe-area-context | ~5.6.0 |
| Screens | react-native-screens | ~4.16.0 |
| WebView | react-native-webview | 13.15.0 |
| File handling | expo-document-picker + expo-file-system + expo-sharing | ~14.0.8 / ~19.0.22 / ~14.0.8 |
| DOCX parsing | mammoth | ^1.12.0 |
| Text encoding | fast-text-encoding | ^1.0.6 |
| Status bar | expo-status-bar | ~3.0.9 |
| Types (React) | @types/react | ~19.1.10 |
| Types (RN) | @types/react-native | ^0.72.8 |
| Backend (optional) | Express 4 ESM + Sequelize 6 + PostgreSQL | (same as react-express-starter) |

---

## Layout A — Mobile only

```
<project-root>/
├── src/
│   ├── components/
│   │   └── ui/                    # Shared design-system wrappers (Button, Card, etc.)
│   ├── screens/
│   │   ├── landing/
│   │   │   └── LandingScreen.tsx  # Initial screen — always scaffolded
│   │   └── <domain>/
│   │       └── <Domain>Screen.tsx
│   ├── navigation/
│   │   ├── RootNavigator.tsx      # NavigationContainer + stack root
│   │   └── TabNavigator.tsx       # Bottom tab navigator
│   ├── features/
│   │   └── <domain>/
│   │       ├── components/        # Domain-specific components
│   │       ├── hooks/             # Domain hooks (useXxx.ts)
│   │       ├── api.ts             # Axios calls for this domain
│   │       └── types.ts           # Domain TypeScript types
│   ├── services/
│   │   └── api.ts                 # Axios instance + base URL + interceptors
│   ├── hooks/                     # Global custom hooks
│   ├── types/                     # Global TypeScript types
│   ├── utils/                     # Pure helpers (formatters, toNumber, etc.)
│   └── constants/
│       ├── theme.ts               # Paper theme, colours, spacing, typography
│       └── config.ts              # API_URL and other env-derived constants
├── assets/
│   ├── fonts/
│   └── images/
├── app.json                        # Expo config
├── tsconfig.json                   # strict: true, path aliases
├── package.json
├── .env.example
├── SPEC.md
├── AGENTS.md
└── CLAUDE.md
```

## Layout B — Mobile + Express backend (monorepo)

```
<project-root>/
├── mobile/                         # Expo app — Layout A structure inside
│   └── ... (same as Layout A)
├── server/                         # Express backend — follows react-express-starter
│   ├── index.js
│   ├── app.js
│   ├── api/
│   │   ├── index.js
│   │   ├── health/
│   │   │   ├── healthController.js
│   │   │   └── index.js
│   │   └── <domain>/
│   │       ├── <domain>Controller.js
│   │       └── index.js
│   ├── config/
│   │   ├── database.js
│   │   └── schema.js
│   ├── models/
│   │   ├── index.js
│   │   └── <domain>.js
│   └── utils/
│       └── number.js
├── .env.example
├── package.json                    # Root scripts using concurrently
├── SPEC.md
├── AGENTS.md
└── CLAUDE.md
```

---

## Key conventions

### TypeScript (`clean-typescript` skill)

- `tsconfig.json`: `strict: true`, `noImplicitAny: true`
- Path aliases (add to `tsconfig.json` and `babel.config.js` / `metro.config.js`):
  - `@components/*` → `src/components/*`
  - `@screens/*` → `src/screens/*`
  - `@features/*` → `src/features/*`
  - `@services/*` → `src/services/*`
  - `@hooks/*` → `src/hooks/*`
  - `@utils/*` → `src/utils/*`
  - `@constants/*` → `src/constants/*`
- No `enum` — use `type Status = "OPEN" | "CLOSED"` or `as const` objects
- Explicit return types on all exported functions
- No `any`, no `!` non-null assertions; use narrowing instead

### Components (`modern-react-components` skill)

- Named `function` declarations, not arrow functions
- Single responsibility per file; one default-exported component per file
- Kebab-case filenames (`landing-screen.tsx`), PascalCase identifiers (`LandingScreen`)
- Platform-specific extensions when needed: `.ios.tsx`, `.android.tsx`, `.web.tsx`
- No `useEffect` for derived state — compute during render
- No boolean prop explosion — use variants or `children`

### UI components (`react-native-paper`)

- Wrap the entire app in `PaperProvider` at the entry point (`App.tsx` / `index.tsx`)
- Pass a custom theme object from `src/constants/theme.ts`:

  ```ts
  // src/constants/theme.ts
  import { MD3LightTheme } from "react-native-paper";

  export const appTheme = {
    ...MD3LightTheme,
    colors: {
      ...MD3LightTheme.colors,
      primary: "#6200ee",
    },
  };
  ```

- Prefer Paper components as the **first choice** for UI:
  - Layout → `Surface`
  - Typography → `Text` (with `variant` prop: `displayLarge`, `headlineMedium`, `bodyMedium`, etc.)
  - Actions → `Button`, `FAB`, `IconButton`
  - Containers → `Card`, `Card.Content`, `Card.Actions`
  - Navigation chrome → `Appbar.Header`, `Appbar.Content`
- Do **not** use raw `View`/`TouchableOpacity` where a Paper equivalent exists

### Landing screen

Always scaffold `src/screens/landing/LandingScreen.tsx` as the initial route. Minimum viable shape:

```tsx
import { StyleSheet, View } from "react-native";
import { Button, Surface, Text } from "react-native-paper";

function LandingScreen(): React.JSX.Element {
  return (
    <Surface style={styles.container}>
      <Text variant="displaySmall" style={styles.title}>
        Welcome
      </Text>
      <Text variant="bodyLarge" style={styles.subtitle}>
        Your app is ready.
      </Text>
      <Button mode="contained" onPress={() => {}}>
        Get started
      </Button>
    </Surface>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, alignItems: "center", justifyContent: "center", gap: 16 },
  title: { fontFamily: "Geist_700Bold" },
  subtitle: { fontFamily: "Geist_400Regular" },
});

export default LandingScreen;
```

### Navigation

- `RootNavigator.tsx` wraps `NavigationContainer` from `@react-navigation/native`
- Screens registered with `createNativeStackNavigator` from `@react-navigation/native-stack`
- Tab bar via `createBottomTabNavigator` from `@react-navigation/bottom-tabs`
- Icons via `lucide-react-native` (requires `react-native-svg`)
- `LandingScreen` is always the initial route in the root stack

### Fonts

Load Geist at the app entry before rendering:

```ts
import { useFonts, Geist_400Regular, Geist_700Bold } from "@expo-google-fonts/geist";

const [fontsLoaded] = useFonts({ Geist_400Regular, Geist_700Bold });
if (!fontsLoaded) return null;
```

### HTTP / API (`api-structure` + `web-security` skills)

- Single Axios instance in `src/services/api.ts`:

  ```ts
  import axios from "axios";
  import { API_URL } from "@constants/config";

  export const apiClient = axios.create({ baseURL: API_URL });
  ```

- One `api.ts` per feature domain under `src/features/<domain>/api.ts`
- Generic error envelope on the server: `{ error: "<short message>" }` — show generic text in UI, log full error to console
- Never commit `.env`; provide `.env.example` with placeholder values only

### Formatting

- Numbers: use `Intl.NumberFormat` — do **not** add `numeral.js` or similar
- Decimal coercion: `toNumber()` util (mirrors the pattern in `react-express-starter`)

### Backend (if chosen) — follow `api-structure` + `react-express-starter` skills exactly

- `server/index.js` listens on `API_PORT`, binds to `127.0.0.1`
- Per-domain layout: `server/api/<domain>/<domain>Controller.js` + `index.js`
- Lazy Sequelize model accessors via `getXxxModels()`
- `schemaTable()` for every raw SQL identifier interpolation
- `toNumber()` on every Decimal before returning JSON
- Read-only database contract; no `INSERT/UPDATE/DELETE` paths

---

## Bootstrap order

### Mobile only

1. `npx create-expo-app <name> --template blank-typescript`
2. Install all deps from the stack table: `npm install react-native-paper@^5.15.3 @react-navigation/native@^7.2.4 ...`
3. Add `tsconfig.json` — `strict: true`, path aliases
4. Configure path alias resolution in `babel.config.js` (`babel-plugin-module-resolver`) or `metro.config.js`
5. Create `src/` folder hierarchy (Layout A above)
6. Create `src/constants/theme.ts` (Paper theme)
7. Create `src/services/api.ts` (Axios instance)
8. Create `src/navigation/RootNavigator.tsx` + `TabNavigator.tsx`
9. Scaffold `src/screens/landing/LandingScreen.tsx` (see template above)
10. Wire `LandingScreen` as the initial route in `RootNavigator.tsx`
11. At app entry: load Geist fonts, wrap in `PaperProvider` + `NavigationContainer`
12. Scaffold first feature domain under `src/features/<domain>/`
13. Add `SPEC.md`, `AGENTS.md`, `CLAUDE.md`
14. Validate (see §Validation)

### Mobile + backend

1. Create root `package.json` with `concurrently` in devDependencies and a `dev` script
2. Follow Mobile bootstrap steps inside `mobile/`
3. Follow `react-express-starter` bootstrap steps inside `server/`
4. Set `API_URL` in `mobile/src/constants/config.ts` to the Express port (e.g. `http://127.0.0.1:3001`)
5. Add root dev script:

   ```json
   "dev": "concurrently \"npm run start --workspace=mobile\" \"npm run dev:server --workspace=server\""
   ```

---

## `package.json` scripts (mobile)

```json
{
  "scripts": {
    "start": "expo start",
    "android": "expo run:android",
    "ios": "expo run:ios",
    "web": "expo start --web",
    "typecheck": "tsc --noEmit"
  }
}
```

---

## Validation

```bash
# TypeScript — must be clean before declaring done
npx tsc --noEmit

# Expo dependency check
npx expo-doctor

# Start dev server on port 3000
npx expo start --port 3000
```

If any of these fail, report the error explicitly — do **not** claim the bootstrap is complete.

---

## Definition of done

- [ ] All deps from the stack table installed at the exact versions listed
- [ ] `tsconfig.json` with `strict: true` and path aliases configured
- [ ] `src/` folder structure matches Layout A (or Layout B for full-stack)
- [ ] `src/constants/theme.ts` with Paper MD3 theme
- [ ] Axios instance in `src/services/api.ts`
- [ ] `RootNavigator.tsx` wired and renders without errors
- [ ] `LandingScreen` scaffolded and set as the initial route
- [ ] App entry wrapped in `PaperProvider` with custom theme
- [ ] Geist fonts loaded at app entry
- [ ] `.env.example` present; `.env` in `.gitignore`
- [ ] `SPEC.md`, `AGENTS.md`, `CLAUDE.md` present
- [ ] `npx tsc --noEmit` passes
- [ ] `npx expo-doctor` reports no blockers
- [ ] Backend (if chosen): `npm run lint && npm run typecheck` clean per `react-express-starter` rules
