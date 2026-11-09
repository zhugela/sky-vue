# sky-vue · Admin Dashboard (Sky Take-out)

> Vue 2 admin console for the [sky-take-out](https://github.com/zhugela/sky-take-out) delivery system — dashboards, order handling, dish and set-meal management, employee and shop settings.

![Vue](https://img.shields.io/badge/Vue-2-42b883?logo=vuedotjs&logoColor=white)
![Element UI](https://img.shields.io/badge/Element%20UI-2-409EFF)
![ECharts](https://img.shields.io/badge/ECharts-5-AA344D)

---

## Modules

| Module | Screens |
|---|---|
| Dashboard | today's turnover, orders, effective users, unit price; order & sales trend charts |
| Orders | pending / in-progress / completed, accept, reject, deliver |
| Dishes | list, create, edit, flavour options, on/off shelf |
| Set meals | list, create, edit, category association |
| Employees | list, add, edit, enable/disable |
| Shop | business status, settings |

## Stack

Vue 2 · Vue Router · Element UI · ECharts · axios · vue-admin-template scaffold

## Getting started

```bash
npm install
npm run serve        # dev server, proxies /api to the backend on :8080
npm run build
```

Environment files (`.env.development`, `.env.production`) hold the API base URL; no secrets are committed.

## Note

Course project (recreated from the well-known 苍穹外卖 curriculum) kept for practice and reference.
