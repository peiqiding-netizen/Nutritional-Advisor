<div align="center">
<img width="1200" height="475" alt="GHBanner" src="https://github.com/user-attachments/assets/0aa67016-6eaf-458a-adb2-6e31a0763ed6" />
</div>

# NutriSnap — Nutritional Advisor

A personal nutrition web app that uses AI to analyse meal photos, generate personalised meal plans, and provide daily coaching based on your health profile.

- Meal photos are analysed via `POST /api/vision/analyze` (currently using Google Gemini as a placeholder)
- Daily coaching goes through `POST /api/coach/daily-feedback` (powered by a custom-trained LLM served via Colab + Cloudflare Tunnel)
- Confirmed meals are stored locally in the browser and drive the dashboard and daily coach review

---

## Run Locally (Mac + VS Code)

### Prerequisites

| Tool | Purpose | Download |
|------|---------|----------|
| Node.js v24 LTS | Run the project | [nodejs.org](https://nodejs.org) |
| VS Code | Code editor | [code.visualstudio.com](https://code.visualstudio.com) |
| OpenRouter API Key | AI features | [openrouter.ai](https://openrouter.ai) → Sign In → Keys |

### Steps

1. **Clone the repo:**
   ```bash
   git clone https://github.com/peiqiding-netizen/Nutritional-Advisor.git
   cd Nutritional-Advisor/Nutrition_Advisor
   ```

2. **Install dependencies:**
   ```bash
   npm install
   ```

3. **Create `.env.local`** in the project root:
   ```bash
   AI_API_KEY="sk-or-v1-..."
   AI_HTTP_REFERER="http://127.0.0.1:3000"
   AI_APP_TITLE="NutriSnap"
   AI_REVIEW_API_BASE_URL="https://your-cloudflare-url.trycloudflare.com"
   VISION_PYTHON_URL="http://127.0.0.1:8010/api/vision/analyze"
   VITE_APP_API_BASE_URL="http://127.0.0.1:8787"
   ```

   > ⚠️ `AI_REVIEW_API_BASE_URL` must point to the Colab Cloudflare Tunnel URL (changes every session — see LLM Setup below).

4. **Start the backend API (Terminal 1):**
   ```bash
   npm run dev:api
   ```

5. **Start the frontend (Terminal 2):**
   ```bash
   npm run dev
   ```

6. **Open in browser:**
   ```
   http://127.0.0.1:3000
   ```

---

## LLM Setup (Daily Coach)

The Daily Coach uses a custom-trained nutrition LLM hosted on Google Colab.

1. Open `API_Calling.ipynb` in Google Colab
2. Run each cell in order (Cell 1–3 install dependencies and load the model)
3. Cell 4 starts a Cloudflare Tunnel and prints a public URL like:
   ```
   https://xxxx.trycloudflare.com/analyze
   ```
4. Copy the URL **without `/analyze`** and paste it into `.env.local` as `AI_REVIEW_API_BASE_URL`
5. Keep Cell 4 running — closing it will kill the tunnel

> ⚠️ The URL changes every time Colab restarts. Update `.env.local` and restart `npm run dev:api` each session.

---

## What Changed (vs Original Repo)

### Branding
- Renamed app from "The Vitality Edit / Vitality Advisor" to **NutriSnap** in `Sidebar.tsx` and `Onboarding.tsx`

### Vision Model (`server/index.ts`)
- Vision model (`vision_api_server.py`) is **not yet connected** — currently using **Google Gemini** (`google/gemini-2.0-flash-lite-001`) via OpenRouter as a temporary substitute for food photo analysis

### Daily Coach (`server/index.ts`)
- Connected to a custom-trained LLM via Colab + Cloudflare Tunnel
- Fixed `inferUserGroup` to correctly detect diagnosed conditions (`diabetes`, `depression`, `sleep difficulties`) from user profile
- Added `Diabetes+Depression` combined condition group
- Improved prompt logic so the model respects user conditions and does not infer conditions that were not selected
- When no meals are logged, the model now generates a personalised meal plan based on the user's profile instead of returning an error
- Added support for user's `Nutrition Plan Preference` (1-Day Meal Plan, 1-Week Meal Plan, Recommended Targets, Intake Critique) — the model generates content accordingly
- Removed `Current Intake: 0cal` from the prompt when no meals are logged to prevent the model from misinterpreting the user's state

### Coach Screen (`src/screens/Coach.tsx`)
- Added **Markdown rendering** using `react-markdown` so model output (tables, bold text, headings, lists) displays correctly
- Intake Critique now **requires at least 3 logged meals** before generating a review

### Log Meal (`src/screens/LogMeal.tsx`)
- Fixed **Confirm and Log Meal button** not working over HTTP by replacing `crypto.randomUUID()` with `Math.random().toString(36)` (browser security restriction)

### Dashboard (`src/screens/Dashboard.tsx` + `src/App.tsx`)
- Added **delete meal** button to each logged meal card

---

## Environment Variables Reference

| Variable | Required | Description |
|----------|----------|-------------|
| `AI_API_KEY` | ✅ Yes | OpenRouter API key (`sk-or-v1-...`) |
| `AI_HTTP_REFERER` | No | Keep default |
| `AI_APP_TITLE` | No | App name shown in OpenRouter dashboard |
| `AI_REVIEW_API_BASE_URL` | ✅ Yes | Colab Cloudflare Tunnel URL (changes each session) |
| `VISION_PYTHON_URL` | No | Local Python vision service URL (not yet active) |
| `VITE_APP_API_BASE_URL` | No | Keep default |

---

## Project Structure

```
Nutrition_Advisor/
├── server/
│   └── index.ts          # Express backend (vision + coach API routes)
├── src/
│   ├── components/
│   │   ├── Sidebar.tsx   # App name + navigation
│   │   └── TopBar.tsx
│   ├── lib/
│   │   ├── coach.ts      # Daily coach API call
│   │   ├── meal-log.ts   # Local meal storage
│   │   ├── profile.ts    # User profile + nutrition targets
│   │   └── vision.ts     # Meal photo analysis API call
│   └── screens/
│       ├── App.tsx        # Root component + state management
│       ├── Coach.tsx      # Daily Coach screen
│       ├── Dashboard.tsx  # Dashboard screen
│       ├── LogMeal.tsx    # Meal logging screen
│       └── Onboarding.tsx # Profile setup screen
├── .env.local             # ⚠️ Not committed — create manually
└── package.json
```
