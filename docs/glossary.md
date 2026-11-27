# Glossary

The exact domain words of **TechRemind** and what they mean in this application. AI agents read this file before every task (via `opencode.jsonc`). Use these exact terms in requirements, UI texts, code, and tests. If you rename a term, update it here first.

## Terms

| Term | Meaning in this app | Not to be confused with | In code |
| --- | --- | --- | --- |
| **Guest** | An unauthenticated visitor who practices public topics and sees only the current Session Result without saved progress, Readiness, or suggested topics. | User | `'guest'` |
| **User** | An authenticated user (`email` + `password`) whose question statuses, Dashboard stats, and 14/30-day expirations are saved. | Guest, Admin | `User`, `'user'` |
| **Admin** | A seeded account with role `admin`; in MVP has the same capabilities as a User (moderation and admin statistics are out of scope). | User | `'admin'` |
| **Topic** | A public, bilingual (`uk`, `en`) category of questions seeded for all users, with a `quizEnabled` flag indicating whether Quiz mode is ready for it. | Collection | `Topic`, `quizEnabled` |
| **Collection** | A private deck of questions created and owned by a single User (planned feature, not in MVP). | Topic | `Collection` |
| **Question** | One study item belonging to a Topic (with bilingual prompt, reference answer, and optional quiz options when Quiz is enabled). | Session | `Question` |
| **Self-Check** | Training mode where the user sees the question, optionally reveals the reference answer, and rates themselves (`Known`, `Unknown`, `For Review`) or skips. | Quiz | `'self_check'` |
| **Quiz** | Training mode for public Topics with `quizEnabled: true`, where the user picks from shuffled answer options (or skips) and gets an immediate check. | Self-Check | `'quiz'` |
| **Session** | A single training run with selected Topics, mode, non-repeating ordered queue of Questions, and recorded per-question outcomes. | Topic | `Session` |
| **Difficulty Preset** | Volume level synced with the question-count slider: `Easy` (`<= 20`), `Medium` (`21–50`), `Hard` (`>= 51`); presets above available count are disabled. | Question difficulty | `'easy' \| 'medium' \| 'hard'` |
| **New** | Default initial state of a Question for a User before any non-skipped answer is recorded; excluded from `Reviewed` and `Readiness`, and placed second in queue priority after `For Review` and `Unknown`. | Unknown, Skipped | `'new'` |
| **Known** | Question status meaning the user answered confidently (or chose the right quiz option); counts toward Readiness and decays to `For Review` after 14 days (`KNOWLEDGE_TTL_DAYS`). | Reviewed, For Review | `'known'` |
| **For Review** | Question status meaning the user flagged it to revisit, or a `Known` answer expired after 14 days; decays to `Unknown` after 30 days (`REVIEW_TTL_DAYS`). | Unknown, Skipped | `'for_review'` |
| **Unknown** | Question status meaning the user failed the question (in Self-Check or Quiz), or a `For Review` item expired after 30 days; does not count toward Readiness. | New, Skipped, For Review | `'unknown'` |
| **Skipped** | Outcome when a user skips a question during a Session; excluded from Session Result denominator and leaves the Question's saved status (`New`, `Known`, `For Review`, or `Unknown`) unchanged. | New, Unknown | `'skipped'` |
| **Reviewed** | Any Question in a Topic for which the User has a status other than `New` (`Known`, `Unknown`, or `For Review`), across either mode. | Known, Readiness | `reviewedCount` |
| **Readiness** | Topic percentage on a User's Dashboard: `Known / total questions in Topic` (never includes `New`, `Unknown`, `For Review`, or `Reviewed`). | Session Result | `readinessPercent` |
| **Session Result** | Score percentage shown after finishing a Session: `Known / (total in Session - Skipped)`. | Readiness | `sessionScorePercent` |
| **Seed** | Curated public bilingual starter topics, questions, and demo user progress in `data/seed.json` (using relative `statusUpdatedDaysAgo` offsets so demo states never go stale). | Personal Collection | `seed` |
