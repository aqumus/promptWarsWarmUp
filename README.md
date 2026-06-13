# 🍳 CookPal — your daily mise en place

> **PromptWars · Build with AI — warm-up submission.**
> Challenge: *"A cooking to-do list."* Build a simple AI micro-app that helps a user generate a
> personal cooking to-do list based on their day — a Breakfast/Lunch/Dinner plan, a grocery
> list, substitutions, and budget feasibility logic. CookPal delivers all four in a single,
> zero-dependency `index.html`, with a swappable offline / Claude-AI meal engine.

A tiny **AI cooking micro-app** that turns your day into a personal cooking to-do list:
a Breakfast / Lunch / Dinner plan, a grocery list, budget feasibility, and ingredient
substitutions when you're over budget.

Built for the **PromptWars · Build with AI** warm-up challenge ("A cooking to-do list").

**▶ Live demo: https://prompt-wars-warm-up.vercel.app/**

> Single `index.html`. No build step, no dependencies, no server required. Just open it.

---

## ✨ What it does

1. **Onboarding** (saved to `localStorage`): learns your name, currency, eating style,
   liked cuisines, allergies/dislikes, and two budgets — an everyday **daily budget** and a
   bigger **special-day budget**.
2. **Plans your day** based on the current **time of day** (the next meal is flagged
   *"up next"*) and your preferences.
3. **Prices everything**: ingredient prices are kept in **USD** and converted to your
   currency via a stored FX rate.
4. **Checks the budget**: totals the grocery cost against today's budget (daily vs. special)
   and tells you if you're within or over.
5. **Suggests substitutions**: if you're over budget, it swaps the priciest ingredients for
   cheaper alternatives (e.g. salmon → tilapia, beef → lentils) until it fits.
6. **Hands you a checklist**: a grouped **Shop → Cook** to-do list with progress counters,
   persisted across reloads.

---

## 🚀 Setup

No install. Pick any:

**Option A — use the live demo**
Just open **https://prompt-wars-warm-up.vercel.app/** — nothing to set up.

**Option B — just open it locally**
```bash
open index.html          # macOS
# or double-click index.html in your file manager
```

**Option C — serve locally** (recommended if you'll use the live AI engine, to avoid any
`file://` quirks)
```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

Requirements: any modern browser. Fonts load from Google Fonts when online and fall back to
system serif/sans offline.

### Optional: enable the live Claude AI engine
The app works fully offline with its built-in engine. To plan meals with a real LLM instead:
1. In the app, switch the **Engine** control to **🤖 Claude**.
2. Paste an **Anthropic API key** (`sk-ant-...`) and click **Use Claude**.
3. The browser calls Claude (`claude-haiku-4-5`) directly and returns a structured plan.
   The key is stored only in your `localStorage`. If it's missing or the call fails, the app
   **automatically falls back** to the offline engine.

---

## 🧭 Quick user workflows

### First run — set up your kitchen (≈30s)
1. Open the app → a **3-step wizard** appears.
2. **Step 1 – Basics:** name + currency.
3. **Step 2 – How you eat:** pick a diet (Non-veg / Vegetarian / Vegan), tap liked cuisines,
   add any allergies/dislikes.
4. **Step 3 – Budget:** set a daily budget (use the *Thrifty / Comfy / Generous* presets) and
   a special-day budget.
5. Hit **Start cooking** → your plan is generated instantly.

### Plan a normal day
- The dashboard opens on your last plan. Hit **Plan my day** for a fresh one, or
  **↻ Shuffle** to get different meals from the same preferences.

### Plan a special day 🎉
- Flip the **Day** switch to **Special** → the plan re-generates against your larger
  special-day budget (notice the budget meter and "to spare" change).

### Watch the budget logic work
- Set a tight daily budget (e.g. **$8** / **₹600**) in your profile, then generate.
- If the meals cost more, the **Budget check** card shows the swaps it made
  (struck-through → green) to bring you back under budget.

### Cook from the list
- Scroll to **Your to-do list**: tick off **Shop** items, then **Cook, in order**.
- Checks persist across reloads (click or use keyboard: focus a row, press **Space/Enter**).

### Try the AI engine
- Switch **Engine → 🤖 Claude**, paste your API key, and regenerate — same UI, meals now
  composed by the LLM. Toggle back to **🧮 Offline** anytime.

### Update preferences
- Click **Profile** to reopen the wizard with your current settings pre-filled.

---

## 🏗️ How it's built

The design cleanly separates **what to cook** (swappable) from **the money math** (always
deterministic).

```
Profile + day context
        │
        ▼
┌─────────────────────────────┐     MealProvider contract
│  MealProvider.generate(req) │ ──▶ { meals: { breakfast, lunch, dinner } }
└─────────────────────────────┘        each meal: { name, cuisine, diet,
        │   (offline OR Claude)                      ingredients:[{item,qty,unit}], steps }
        ▼
┌─────────────────────────────────────────────────────────┐
│  Deterministic pipeline (provider-agnostic, in JS)        │
│  1. aggregate grocery list                                │
│  2. price in USD → convert to user currency               │
│  3. budget feasibility (daily vs. special)                │
│  4. substitutions (swap to cheaperAlt until it fits)      │
│  5. cooking to-do list (Shop → Cook)                      │
└─────────────────────────────────────────────────────────┘
```

- **`OfflineProvider`** — filters a built-in recipe set by diet/allergies/dislikes, scores by
  cuisine match, time of day, ingredient variety, and cost, then picks one recipe per meal
  (seeded random, so **Shuffle** is fresh but reproducible).
- **`AIProvider`** — calls the Claude Messages API from the browser with a JSON-schema
  structured-output contract, then normalizes the result to the same shape.
- **Why the split?** The LLM only does the *creative* part (choosing meals). All arithmetic —
  pricing, currency conversion, budget feasibility, substitutions — stays in JavaScript, so
  the budget logic is exact and never hallucinated, regardless of which engine you pick.

### Data lives in the file
- `INGREDIENTS` — ~50 ingredients with **USD** price, unit, and optional `cheaperAlt`.
- `RECIPES` — 24 recipes (8 each for breakfast/lunch/dinner), tagged by diet, cuisine, speed.
- `CURRENCIES` — 9 currencies with symbol + FX rate from USD.

### Persistence (`localStorage`)
| Key | Contents |
|-----|----------|
| `cookpal_profile` | your onboarding answers |
| `cookpal_plan`    | the last generated plan + grocery + budget |
| `cookpal_apikey`  | Anthropic key (only if you use the AI engine) |
| `cookpal_checked` | which to-do items are ticked |

---

## ⚠️ Notes & assumptions

- **Prices are illustrative** — a built-in USD table × static FX rates, not live market data.
- **Substitutions** keep the same quantity at the cheaper alternative's price (a reasonable
  approximation for a warm-up).
- The AI engine sends your API key directly from the browser to Anthropic
  (`anthropic-dangerous-direct-browser-access`); fine for a local demo, not for production.

---

## 📁 Project layout

```
.
├── index.html   # the entire app — markup, styles, and logic
├── README.md    # this file
└── LICENSE      # MIT
```

---

## 📄 License

[MIT](./LICENSE) © 2026 Aquib Vadsaria
