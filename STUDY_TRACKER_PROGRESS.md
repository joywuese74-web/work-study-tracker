# STUDY_TRACKER_PROGRESS.md

_Last updated: 2026-10-29 · Current stage: **Stage 1 — Foundation (not started; planning approved)**_

---

## 1. Project overview

Study Tracker is a general-purpose, multi-user personal activity management platform (website + Android app). Users register, build timetables, manage recurring activities, receive **spoken reminders** (the defining feature), and use an **AI scheduling assistant** that proposes schedule changes and applies them only with approval or under user-defined automation rules.

This is a new product, separate from the earlier Google Apps Script "Study Tracker" (Code.gs / Index.html + Google Sheet).

Development rule: each stage must be tested and meet its acceptance criteria before the next stage begins.

---

## 2. Selected technologies (approved 2026-10-29)

| Layer | Choice | Why |
|---|---|---|
| Website + Android UI | **Flutter** (one codebase, Dart) | Write each screen once for web and Android |
| Android reminder engine | **Kotlin module** inside the Flutter Android project | Exact alarms + short foreground service + Android TextToSpeech work while the app is closed or the phone is locked |
| Backend API | **Python + FastAPI** | Builds on existing Python skills; strong AI ecosystem; keeps secrets server-side |
| Database | **PostgreSQL on Supabase** | Managed Postgres; row-level security as a second privacy layer |
| Authentication | **Supabase Auth** (email + password first) | Tested login, password reset; avoids hand-rolled auth |
| Push messages | **Firebase Cloud Messaging** (Stage 3) | Tells phones to resync alarms when the schedule changes |
| AI assistant | LLM API called **only from the backend** (Stage 4; provider TBD) | AI can only read the schedule and submit proposals; backend validates and applies |
| Hosting | API: Render / Railway / Fly.io · Web: static hosting (Stage 5) | Low-cost tiers |

### Architecture

```
 Android app (Flutter + Kotlin reminder engine)      Website (Flutter web)
                    \                                   /
                     \______ HTTPS + login token ______/
                                     |
                         Backend API (FastAPI)
                  /            |              |              \
          Supabase Auth   PostgreSQL     AI provider     Firebase Cloud Messaging
```

### Known platform limits (by design, not bugs)
- The **website** can speak reminders only while its tab is open; otherwise it sends silent web-push notifications.
- On **Android 14+**, exact-alarm permission is denied by default for new installs; the app must explain why and send the user to settings. If refused, reminders still arrive but may be late.
- Some phone makers (Tecno, Infinix, Xiaomi, Samsung, etc.) kill background apps; the app needs a "make reminders reliable" setup screen.

---

## 3. Folder structure (planned)

```
study-tracker/
├── STUDY_TRACKER_PROGRESS.md
├── README.md
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── config.py
│   │   ├── auth.py
│   │   ├── models/
│   │   ├── routes/
│   │   ├── services/
│   │   └── tests/
│   ├── migrations/
│   ├── requirements.txt
│   └── .env.example
├── app/
│   ├── lib/
│   │   ├── main.dart
│   │   ├── screens/
│   │   ├── widgets/
│   │   ├── services/
│   │   └── models/
│   ├── android/app/src/main/kotlin/…/reminders/
│   └── test/
└── docs/
```

---

## 4. Completed features

_None yet. Planning and architecture approved on 2026-10-29._

| Feature | Status |
|---|---|
| Architecture & technology selection | ✅ Approved |
| Everything else | ⏳ Not started |

---

## 5. Database schema

_To be designed in Stage 1, step 1.3. Planned core tables:_ `profiles`, `categories`, `activities`, `activity_occurrences` / recurrence rules, `reminder_settings`, `devices` (push tokens), `ai_proposals`, `change_log`. Every user-owned row carries `user_id`; times stored in UTC plus the user's IANA time zone; recurrence in iCalendar RRULE format.

---

## 6. Configuration requirements

| Item | Needed by | Status |
|---|---|---|
| GitHub account + repo | Stage 1 | ⏳ |
| Supabase project (URL, anon key, service key — service key server-only) | Stage 1 | ⏳ |
| Development environment that survives logout | Stage 1 | ⏳ Decision pending |
| Android test phone | Stage 1 (voice proof of concept) | ⏳ Decision pending |
| Firebase project | Stage 3 | ⏳ |
| AI provider API key + monthly budget | Stage 4 | ⏳ |
| Hosting accounts | Stage 5 | ⏳ |
| Google Play developer account ($25) + 12 testers for 14 days | Stage 5 | ⏳ |

Secrets live only in `backend/.env` (never committed); `.env.example` lists the names.

---

## 7. Commands used to run and test the project

_None yet._

---

## 8. Known errors and limitations

- Development computer wipes its setup on logout; a persistent environment must be chosen before installing tools.
- Background voice reliability must be proven on a real phone (Stage 1 proof of concept).

---

## 9. Decisions

| Decision | Outcome |
|---|---|
| Stack (Flutter + Kotlin, FastAPI, Supabase) | ✅ Approved |
| Voice proof of concept in Stage 1 | ✅ Approved |
| Default AI permission mode | ✅ Approval required for every change |
| First login method | ✅ Email + password (Google sign-in later) |
| Development environment | ✅ VS Code on own computer (backend first; Flutter/Android install deferred to step 1.5) |
| Test phone (brand / Android version) | ⏳ Pending (needed by step 1.7) |
| AI provider + monthly budget | ⏳ Pending (needed by Stage 4) |
| Product name | ⏳ Pending (keeping "Study Tracker" for now) |
| Launch scope (personal/testers vs public) | ⏳ Pending (needed by Stage 5) |
| Weekly hours available | ⏳ Pending |

---

## 10. Stage 1 plan — Foundation

**Objective:** a person can sign up, log in, set their profile and time zone, and see a basic dashboard on the website and on Android — with their data private to them. Plus proof that the phone can speak a reminder while locked.

| Step | Task | Status |
|---|---|---|
| 1.1 | Set up development environment + Git repo + folder skeleton | ⏳ |
| 1.2 | Create Supabase project; store keys safely | ⏳ |
| 1.3 | Design database schema (profiles, categories) + row-level security | ⏳ |
| 1.4 | FastAPI skeleton: health check, token verification, profile routes | ⏳ |
| 1.5 | Flutter app: sign-up, log-in, log-out, profile/time-zone screen | ⏳ |
| 1.6 | Basic dashboard (today's date, greeting, placeholder for activities) | ⏳ |
| 1.7 | Voice proof of concept: Android test app speaks at a set time while locked | ⏳ |
| 1.8 | Stage 1 tests + update this document | ⏳ |

### Stage 1 acceptance criteria
- [ ] New user can register, log in, log out, and reset password (web and Android).
- [ ] Profile (name, time zone, notification preference) saves and reloads correctly.
- [ ] User A cannot read or change User B's profile (tested via the API directly).
- [ ] No secret key appears in Flutter code or in the Git repository.
- [ ] Dashboard loads for a logged-in user; logged-out users are sent to login.
- [ ] Voice proof of concept speaks within ~1 minute of the set time with the screen locked and the app closed, on the real test phone.

---

## 11. Exact next development step

**Stage 1, step 1.1:** choose the development environment (GitHub Codespaces + cloud Android builds, or a personal laptop) and name the Android test phone. Then create the repository and folder skeleton.
