---
name: api-structure
description: Scaffold a new API domain in an Express backend, with or without Sequelize. Use when the user asks to add an endpoint, add a domain, mirror the existing backend API structure, or follow SPEC.md.
version: 2.0.0
last_updated: 2026-06-16
---

# API Structure — Express backend (optional Sequelize)

This skill scaffolds a new per-domain API slice in an Express backend. The data layer is **either Sequelize 6 over `pg`/PostgreSQL, or the `pg` driver directly** — match whatever the project already uses (set in `react-web-express-starter` Step 0).

## When to use

Invoke this skill when the user asks to:

- Create a new API endpoint or domain
- Mirror the existing per-domain backend layout
- Add model/data-access code on the server
- Follow / update `SPEC.md`

## When NOT to use

- For client-side API calls — use the RTK Query pattern in `src/client/store/api/` instead.
- For DB schema migrations — only if the project owns its schema. Confirm ownership with the user first.

---

## Detect the data layer first

Before scaffolding, determine which data layer the project uses:

- **Sequelize** if `sequelize` is in `package.json` and `server/config/database.js` + `server/models/` exist → follow the **Sequelize variant** below.
- **Plain `pg`** if `server/config/db.js` exports a `query()` helper and there is no `server/models/` → follow the **Plain `pg` variant**. (This setup also ships an `axios` instance in `server/config/http.js` for calling external/upstream REST APIs — use it for any non-DB data source.)

If neither exists yet, ask the user (or fall back to `react-web-express-starter` Step 0).

---

## Read vs write

Do **not** assume the API is read-only. Confirm with the user whether the domain needs mutating endpoints (`POST`/`PUT`/`PATCH`/`DELETE`).

- If **read-only by design** (e.g. dashboards over a view), scaffold only `GET` handlers and say so in `SPEC.md`.
- If **read-write**, scaffold the mutating handlers too, and validate every request body at the trust boundary (see `web-security`).

---

## Inputs

| Input             | Source                                                            |
| ----------------- | ----------------------------------------------------------------- |
| Domain name       | User (e.g. `widget`)                                              |
| Source table/view | User or `SPEC.md`                                                 |
| Response shape    | User, or derived from the table's columns                        |

## Outputs

For a new domain `widget`:

1. `server/api/widget/widgetController.js`
2. `server/api/widget/index.js`
3. Edit `server/api/index.js` — mount the router
4. **Sequelize variant only**: `server/models/widget.js` + a `getWidgetModels()` lazy accessor in `server/models/index.js`
5. Edit `SPEC.md` — add a section for the new domain
6. (Client follow-up) RTK Query service + types — flag this to the user; it's outside the scope of this skill

---

## Folder layout (target)

```
server/
└── api/
    └── widget/
        ├── widgetController.js
        └── index.js
```

The folder name **must** match the domain name. The controller filename **must** be `<domain>Controller.js`.

---

## Templates

### Router (`server/api/widget/index.js`) — same for both variants

```js
import express from "express";
import { index } from "./widgetController.js";

const router = new express.Router();

router.get("/", index);

export default router;
```

### Router registration (`server/api/index.js`) — same for both variants

```js
import widgetRoutes from "./widget/index.js";

router.use("/widget", widgetRoutes);
```

---

### Controller — Sequelize variant (`server/api/widget/widgetController.js`)

```js
import { getWidgetModels, sequelize } from "../../models/index.js";
import { toNumber } from "../../utils/number.js";

export async function index(req, res) {
  try {
    const { Widget } = getWidgetModels();

    const rows = await Widget.findAll({
      attributes: ["id", "name", "amount"],
      order: [["amount", "DESC"]],
      limit: 10,
      raw: true,
    });

    return res.json({
      widgets: rows.map((row) => ({
        id: row.id,
        name: row.name,
        amount: toNumber(row.amount), // DECIMAL arrives as a string from pg
      })),
    });
  } catch (error) {
    console.error("Widget query failed.", error);
    return res.status(500).json({ error: "Unable to load widgets." });
  }
}
```

### Controller — Plain `pg` variant (`server/api/widget/widgetController.js`)

```js
import { query } from "../../config/db.js";
import { toNumber } from "../../utils/number.js";

export async function index(req, res) {
  try {
    // Always parameterise — never interpolate user input into SQL.
    const { rows } = await query(
      `select id, name, amount
         from widget
        order by amount desc
        limit $1`,
      [10],
    );

    return res.json({
      widgets: rows.map((row) => ({
        id: row.id,
        name: row.name,
        amount: toNumber(row.amount),
      })),
    });
  } catch (error) {
    console.error("Widget query failed.", error);
    return res.status(500).json({ error: "Unable to load widgets." });
  }
}
```

> **Calling an external API instead of (or alongside) the DB?** Import the shared axios instance from `server/config/http.js` rather than creating a new client per controller:
>
> ```js
> import { httpClient } from "../../config/http.js";
> const { data } = await httpClient.get("/widgets", { params: { limit: 10 } });
> ```

Controller rules (both variants):

- **Named exports only.** Never `module.exports`.
- **Use `Promise.all`** for parallel independent reads.
- **Aggregate in SQL** (`COUNT`, `SUM`, `COALESCE`), not in JS.
- **`toNumber()` every Decimal** before returning — `pg` serialises `DECIMAL`/`NUMERIC` as strings.
- **camelCase** all JSON keys.
- **Generic error message** in the response. Log the underlying error to the server console only.

---

### Model (`server/models/widget.js`) — Sequelize variant only

```js
import { DataTypes } from "sequelize";

export function initWidgetModels(sequelize) {
  const Widget = sequelize.define(
    "Widget",
    {
      id: { type: DataTypes.INTEGER, primaryKey: true },
      name: DataTypes.STRING,
      amount: DataTypes.DECIMAL,
    },
    {
      tableName: "widget",
      freezeTableName: true,
      // timestamps: true is the Sequelize default. Set timestamps: false ONLY if
      // the table has no createdAt/updatedAt columns (e.g. a database view).
    },
  );

  return { Widget };
}
```

Model rules:

- `freezeTableName: true` — the model name does not auto-pluralise.
- Set `timestamps: false` only when the table/view genuinely lacks `createdAt`/`updatedAt`.
- Use `constraints: false` on associations **only** when the schema is owned externally and you don't want Sequelize to enforce FKs. For a project that owns its schema, leave constraints on.
- Compound primary keys for views (multiple `primaryKey: true` columns).

### Lazy accessor (`server/models/index.js`) — Sequelize variant only

```js
import { initWidgetModels } from "./widget.js";

let widgetModels;

export function getWidgetModels() {
  if (!widgetModels) {
    widgetModels = initWidgetModels(sequelize);
  }
  return widgetModels;
}
```

---

### Dynamic SQL identifiers (optional, both variants)

Only needed if the project interpolates **dynamic** schema/table names into raw SQL (e.g. a configurable schema). Most projects don't — prefer hard-coded identifiers + parameterised values.

If you do need it, validate the identifier against a strict allowlist and never concatenate user input:

```js
// server/config/schema.js
export function schemaTable(name) {
  if (!/^[a-zA-Z_][a-zA-Z0-9_]*$/.test(name)) {
    throw new Error(`Invalid identifier: ${name}`);
  }
  return name;
}
```

---

## Definition of done

- [ ] `server/api/<domain>/<domain>Controller.js` exists with named-export handlers.
- [ ] `server/api/<domain>/index.js` registers the routes (read-only → `GET` only; read-write → mutating routes with body validation).
- [ ] **Sequelize variant**: model defined with `freezeTableName: true`; lazy `getXxxModels()` added to `server/models/index.js`.
- [ ] **Plain `pg` variant**: queries go through `config/db.js` `query()` and are fully parameterised.
- [ ] Router mounted from `server/api/index.js`.
- [ ] All `Decimal`/`NUMERIC` fields piped through `toNumber()`.
- [ ] Response keys camelCase.
- [ ] No string-concatenated SQL identifiers — dynamic ones go through an allowlist helper.
- [ ] `SPEC.md` updated with the new domain section, including request/response shapes.
- [ ] `npm run lint && npm run typecheck` clean.

---

## Pitfalls

- **(Sequelize) Forgetting `getXxxModels()`**: importing a model directly instantiates Sequelize at module load time, breaking the lazy-init pattern.
- **(Sequelize) Calling `await sequelize.authenticate()` in controllers**: not needed; the pool lazy-connects.
- **Returning Decimals raw**: `pg` serialises them as strings; clients then break on arithmetic. Always `toNumber()`.
- **String-concatenating SQL**: interpolating identifiers or values into a query string is an injection risk. Parameterise values (`$1`), allowlist identifiers.
