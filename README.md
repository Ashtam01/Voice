<p align="center">
  <!-- TODO: Replace with your logo file -->
  <img src="public/logo.svg" alt="Sampark" width="60" height="60" />
</p>

<h1 align="center">Sampark</h1>

<p align="center">
  AI-powered text-to-speech and voice cloning platform.
  <br />
  Generate natural-sounding speech from text, clone voices from audio samples, and manage a full voice library — all from a single dashboard.
</p>

<p align="center">
  <a href="#features">Features</a> •
  <a href="#tech-stack">Tech Stack</a> •
  <a href="#architecture">Architecture</a> •
  <a href="#getting-started">Getting Started</a> •
  <a href="#environment-variables">Environment Variables</a> •
  <a href="#project-structure">Project Structure</a> •
  <a href="#deployment">Deployment</a>
</p>

<br />

<!-- TODO: Replace with a screenshot of the app -->
<p align="center">
  <img src="" alt="Sampark — Dashboard" width="100%" />
</p>

---

## Features

- **Text-to-Speech Generation** — Convert text up to 5,000 characters into natural-sounding speech with configurable parameters (temperature, top-p, top-k, repetition penalty).
- **Voice Cloning** — Clone any voice by uploading an audio file or recording directly from the browser. Minimum 10 seconds of audio required.
- **Voice Library** — Browse and search across system voices and your custom-cloned voices, organized by category and language.
- **Audio Playback** — Full waveform visualization with WaveSurfer.js, play/pause, seek, and download controls.
- **Generation History** — View and replay all previously generated audio.
- **Usage-Based Billing** — Pay-as-you-go pricing powered by Polar with real-time usage metering, subscription management, and customer portal.
- **Multi-Organization Support** — Team workspaces via Clerk organizations with per-org voice libraries and billing.
- **Responsive Design** — Optimized layouts for desktop (resizable panels) and mobile (drawer-based navigation).

---

## Tech Stack

| Layer | Technology |
| --- | --- |
| **Framework** | [Next.js 16](https://nextjs.org/) (App Router) |
| **Language** | TypeScript |
| **API** | [tRPC v11](https://trpc.io/) + [TanStack React Query](https://tanstack.com/query) |
| **Authentication** | [Clerk](https://clerk.com/) (Organizations) |
| **Database** | PostgreSQL via [Prisma 7](https://www.prisma.io/) |
| **Object Storage** | [Cloudflare R2](https://www.cloudflare.com/products/r2/) (S3-compatible) |
| **TTS Engine** | [Chatterbox TTS](https://github.com/resemble-ai/chatterbox) on [Modal](https://modal.com/) (GPU A10G) |
| **Billing** | [Polar](https://polar.sh/) (Subscriptions + Usage Metering) |
| **UI** | [shadcn/ui](https://ui.shadcn.com/) + [Tailwind CSS v4](https://tailwindcss.com/) |
| **Audio** | [WaveSurfer.js](https://wavesurfer.xyz/) · [RecordRTC](https://recordrtc.org/) · [music-metadata](https://github.com/borewit/music-metadata) |
| **Observability** | [Sentry](https://sentry.io/) (Error tracking + Structured logging) |
| **Validation** | [Zod](https://zod.dev/) (End-to-end runtime validation) |

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                     Browser                         │
│  Next.js App (React 19) + tRPC Client + TanStack RQ│
└──────────────────────┬──────────────────────────────┘
                       │ tRPC (HTTP Batch)
                       ▼
┌─────────────────────────────────────────────────────┐
│               Next.js Server (App Router)           │
│                                                     │
│  ┌──────────┐  ┌───────────┐  ┌──────────────────┐  │
│  │  tRPC    │  │ API Routes│  │   Clerk Auth     │  │
│  │ Routers  │  │ (audio,   │  │   Middleware      │  │
│  │          │  │  voices)  │  │                  │  │
│  └────┬─────┘  └─────┬─────┘  └──────────────────┘  │
│       │              │                               │
│  ┌────▼──────────────▼──────┐                        │
│  │      Prisma ORM          │──── PostgreSQL         │
│  └──────────────────────────┘                        │
│                                                     │
│  ┌──────────────────────────┐                        │
│  │    Cloudflare R2 Client  │──── R2 Bucket          │
│  └──────────────────────────┘    (voices + audio)    │
│                                                     │
│  ┌──────────────────────────┐                        │
│  │    Polar SDK             │──── Billing & Metering │
│  └──────────────────────────┘                        │
└──────────────────────┬──────────────────────────────┘
                       │ OpenAPI (HTTPS)
                       ▼
┌─────────────────────────────────────────────────────┐
│          Modal (Serverless GPU — A10G)               │
│                                                     │
│  ┌──────────────────────────────────────────────┐    │
│  │  Chatterbox TTS (FastAPI)                    │    │
│  │  - Voice cloning from R2-mounted audio       │    │
│  │  - WAV generation with configurable params   │    │
│  │  - Auto-scaling with 5-min scaledown         │    │
│  └──────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────┘
```

---

## Getting Started

### Prerequisites

- **Node.js** ≥ 18
- **PostgreSQL** instance (local or hosted)
- **Cloudflare R2** bucket + API credentials
- **Clerk** application with organizations enabled
- **Polar** account with a product configured
- **Modal** account (for TTS backend deployment)

### Installation

```bash
# Clone the repository
git clone https://github.com/your-username/voice-agent.git
cd voice-agent

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env
# Fill in all required values (see Environment Variables below)

# Generate Prisma client
npx prisma generate

# Run database migrations
npx prisma migrate deploy

# Seed system voices (requires audio files in scripts/system-voices/)
npx tsx scripts/seed-system-voices.ts

# Start the development server
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) to access the app.

### Deploying the TTS Backend

```bash
# Install Modal CLI
pip install modal

# Deploy the Chatterbox TTS service
modal deploy chatterbox_tts.py
```

---

## Environment Variables

Create a `.env` file in the project root with the following variables:

| Variable | Description |
| --- | --- |
| `DATABASE_URL` | PostgreSQL connection string |
| `APP_URL` | Public URL of the app (e.g., `https://sampark.app`) |
| **Clerk** | |
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Clerk publishable key |
| `CLERK_SECRET_KEY` | Clerk secret key |
| **Cloudflare R2** | |
| `R2_ACCOUNT_ID` | Cloudflare account ID |
| `R2_ACCESS_KEY_ID` | R2 API access key ID |
| `R2_SECRET_ACCESS_KEY` | R2 API secret access key |
| `R2_BUCKET_NAME` | R2 bucket name (e.g., `sampark-app`) |
| **Chatterbox TTS** | |
| `CHATTERBOX_API_URL` | Modal-deployed TTS endpoint URL |
| `CHATTERBOX_API_KEY` | API key for the TTS service |
| **Polar (Billing)** | |
| `POLAR_ACCESS_TOKEN` | Polar API access token |
| `POLAR_SERVER` | `sandbox` or `production` |
| `POLAR_PRODUCT_ID` | Polar product ID for the subscription plan |
| **Sentry** | |
| `SENTRY_AUTH_TOKEN` | Sentry auth token (build-time source map upload) |

> **Note:** Environment variables are validated at startup via `@t3-oss/env-nextjs`. The app will fail to start if any required variable is missing. Set `SKIP_ENV_VALIDATION=true` to bypass validation during builds.

---

## Project Structure

```
voice-agent/
├── chatterbox_tts.py              # Modal-deployed TTS backend (Python)
├── prisma/
│   ├── schema.prisma              # Database schema (Voice, Generation)
│   └── migrations/                # Database migrations
├── scripts/
│   ├── seed-system-voices.ts      # System voice seeder
│   ├── sync-api.ts                # OpenAPI type generator
│   └── system-voices/             # System voice audio files (.wav)
├── src/
│   ├── app/
│   │   ├── layout.tsx             # Root layout (Clerk, tRPC, NuqsAdapter)
│   │   ├── globals.css            # Design tokens & theme
│   │   ├── (dashboard)/           # Authenticated dashboard routes
│   │   │   ├── text-to-speech/    # TTS generation pages
│   │   │   └── voices/            # Voice library pages
│   │   ├── api/
│   │   │   ├── audio/             # Audio streaming endpoint
│   │   │   ├── voices/            # Voice CRUD + audio serving
│   │   │   └── trpc/              # tRPC HTTP handler
│   │   ├── sign-in/               # Clerk sign-in page
│   │   ├── sign-up/               # Clerk sign-up page
│   │   └── org-selection/         # Organization selection
│   ├── components/
│   │   ├── ui/                    # shadcn/ui primitives (57 components)
│   │   └── voice-avatar/          # DiceBear avatar generator
│   ├── features/
│   │   ├── billing/               # Subscription, checkout, usage display
│   │   ├── dashboard/             # Dashboard layout, sidebar, quick actions
│   │   ├── text-to-speech/        # TTS form, voice selector, audio preview
│   │   └── voices/                # Voice library, creation, recording
│   ├── hooks/                     # Shared React hooks
│   ├── lib/                       # Server utilities (db, r2, env, polar)
│   ├── trpc/                      # tRPC client, server, and routers
│   └── types/                     # Shared TypeScript types
└── public/                        # Static assets
```

---

## Scripts

| Command | Description |
| --- | --- |
| `npm run dev` | Start the development server |
| `npm run build` | Create a production build |
| `npm run start` | Start the production server |
| `npm run lint` | Run ESLint |
| `npm run sync-api` | Regenerate OpenAPI types from the Chatterbox TTS API |
| `npx prisma migrate dev` | Create and apply a new migration |
| `npx tsx scripts/seed-system-voices.ts` | Seed system voices into the database and R2 |
| `modal deploy chatterbox_tts.py` | Deploy the TTS backend to Modal |

---

## Deployment

### Next.js App

The app is optimized for deployment on [Vercel](https://vercel.com/):

1. Connect your Git repository to Vercel.
2. Add all environment variables from the table above.
3. Vercel will auto-detect the Next.js framework and configure the build.

### TTS Backend

The Chatterbox TTS backend runs on [Modal](https://modal.com/) with:

- **GPU:** NVIDIA A10G
- **Autoscaling:** Scales to zero after 5 minutes of inactivity
- **Concurrency:** Up to 10 concurrent requests per container
- **Storage:** Cloudflare R2 bucket mounted read-only for voice files

---

## License

This project is proprietary. All rights reserved.
