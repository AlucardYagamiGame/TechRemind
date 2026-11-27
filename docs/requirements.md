# Product Requirements

> **Status:** draft

This file describes **what** TechRemind must do. AI agents read it before planning or implementing anything (it is loaded automatically through `opencode.jsonc`), so clear requirements lead to better code. Write it in English.

Keep **how** it is built (technical decisions) in `docs/architecture.md`. Domain terms are defined in `docs/glossary.md`: use them exactly.

---

## 1. Product overview

**TechRemind** is a deterministic trainer for technical interviews and tech checks with no AI at its core. Questions are grouped into public topics (such as JavaScript, React, and SQL) in Ukrainian and English, and the application tracks honest per-topic readiness including two-stage knowledge decay (14 days to `For Review`, then 30 days to `Unknown`).

Target users: developers preparing for technical screenings or interviews—both guests trying a quick session and registered users tracking long-term readiness across topics.

## 2. Users

- **Guest:** unauthenticated visitor. Can browse public topics, run `Self-Check` or `Quiz` sessions on public topics, and view the current `Session Result`, with no saved progress, no Dashboard readiness percentages, and no suggested topics.
- **User:** registered user (`email` + `password`). Has all Guest capabilities, plus persistent per-question statuses (`New` by default until answered), Dashboard `Reviewed` counts and `Readiness` percentages, suggested weak topics, prioritized queues, and automatic 14-day / 30-day status expiration.
- **Admin:** seeded account with role `admin`. In MVP it has the same capabilities as a User; admin tools are out of scope.

## 3. User stories

1. As a Guest or User, I want to browse public `Topic` cards, see total question counts and whether `Quiz` is enabled (`"Quiz is still in development"` when unavailable), select topics, set question count via slider (`Easy <= 20`, `Medium 21–50`, `Hard >= 51`), and start a `Self-Check` or `Quiz` `Session`, so that I can practice immediately at the right volume.
2. As a Guest or User, I want to practice in `Self-Check` mode where I can skip a question without changing its status or reveal the reference answer and rate myself (`Known`, `Unknown`, or `For Review`), so that I honestly check my recall before seeing the answer.
3. As a Guest or User, I want to practice quiz-enabled public topics in `Quiz` mode with shuffled answer options and the ability to skip, so that I verify my knowledge objectively (`correct = Known`, `incorrect = Unknown` with explanation shown) without memorizing option positions.
4. As a Guest or User, I want a `Session Result` summary showing `Known`, `Unknown`, `For Review`, and `Skipped` counts with `Skipped` excluded from the session percentage, so that I immediately see what needs review.
5. As a User, I want to register and log in so that my progress persists across sessions, the Dashboard displays per-topic `Reviewed` counts, `Readiness` percentages, and suggested lowest-readiness topics, session queues prioritize `For Review` and `Unknown` before `New` and `Known`, and statuses automatically decay (`Known` → `For Review` after 14 days; `For Review` → `Unknown` after 30 days).
6. As a Guest or User, I want to switch the interface and public questions between Ukrainian (`uk`) and English (`en`), so that I can prepare in the language of my target interview.

## 4. Scope

### In scope (MVP)

- Public bilingual (`uk`, `en`) question bank grouped by `Topic`, seeded from `data/seed.json` (no public editing UI in MVP).
- Per-topic gradual quiz rollout (`quizEnabled`): topics without quiz options remain available for `Self-Check` and display `"Quiz is still in development"` (`"Квіз ще в розробці"` in `uk`) so Quiz cannot be started on unready topics. In `Quiz` mode, Topics with `quizEnabled: false` cannot be selected (their checkbox is disabled and labeled `"Quiz is still in development"`), so a Quiz `Session` never mixes quiz-enabled and quiz-disabled Topics.
- Simple authentication (`email` + `password` register, login, logout; no email verification, password reset, or OAuth).
- Dashboard (dark theme with blue accents and stat cards):
  - For **Guest**: public Topic cards with total question counts, quiz availability state, and topic selection checkboxes.
  - For **User**: all Guest elements plus per-topic `Reviewed` count (questions with status other than `New`), `Readiness` percentage (`Known / total in Topic`), and top 3 suggested topics with lowest `Readiness` (including `0%`).
- Training setup: select one or more Topics, choose `Self-Check` or `Quiz` mode, and adjust question count via slider (`1..available`) with automatic `Easy` (`<= 20`), `Medium` (`21..50`), and `Hard` (`>= 51`) preset state and disabling of presets that exceed `available`.
- Session queue priority for signed-in Users: `For Review` and `Unknown` first, then `New` (never answered), then `Known` last, with no duplicates in a Session.
- `Self-Check` mode (skip leaves saved status unchanged, reveal answer, grade `Known` / `Unknown` / `For Review`) and `Quiz` mode (shuffled options per quiz, skip leaves saved status unchanged, immediate check + explanation on wrong answer).
- `Session Result` summary (`Known`, `Unknown`, `For Review`, `Skipped`; percentage = `Known / (total in Session - Skipped)`).
- Two-stage automatic status decay for registered Users (configurable `KNOWLEDGE_TTL_DAYS=14`, `REVIEW_TTL_DAYS=30`):
  - `Known` → `For Review` after 14 days without re-confirmation (lowering Topic `Readiness`).
  - `For Review` (whether auto-expired or manually marked) → `Unknown` after 30 days without re-confirmation.
- Full Ukrainian and English UI + public question switching.

### Out of scope

- Personal `Collection` decks and file import (`md`, `txt`) with fuzzy duplicate detection: see [docs/features/personal-collections-import.md](features/personal-collections-import.md).
- User requests to add questions to public Topics.
- Admin tools: reviewing User-submitted questions (approve / reject), topic popularity (number of Users per Topic) and total number of Users.
- Training history for Users: one line per Session (date, all selected Topics, total `Known` / `Unknown` / `Skipped`), newest first, can be cleared manually, auto-deleted after 30 days.
- `pdf` and `docx` file parsing.
- Mixed mode (`Self-Check` + `Quiz` combined in one session).
- Saving Guest progress in `localStorage` across sessions or merging it on registration.
- AI/LLM answer evaluation, embeddings, or AI-generated questions at runtime.

## 5. Acceptance criteria

<!--
To be completed in Step 4 with the analyst agent:
For each user story, list checks that prove it works (Given / When / Then). They become your tests.
-->

## 6. Data

<!--
To be completed in Step 4 with the analyst agent:
List the main entities, fields, types, and validation rules based on data/seed.json.
-->

## 7. Open questions

<!--
To be reviewed in Step 4:
- Exact minimum value for the question count slider (1 vs 5) when a topic has very few questions.
- Exact shuffle/tie-breaker order inside each queue priority group.
-->
