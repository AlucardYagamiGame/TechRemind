# Feature: Personal collections and file import

> **Status:** planned — do not implement as part of the MVP

## Idea

A registered `User` can create private personal `Collection` decks, add questions one by one or import a batch from `.md` or `.txt` files, and resolve potential duplicates before saving.

Before writing imported questions, the backend validates the format, normalizes the text, and searches for both exact and **similar** existing questions in the user's collections (and/or public topics). When a likely duplicate is found, the UI shows the existing match and lets the user decide: **Add anyway** or **Skip**. Personal collections are visible only to their owner and can be practiced in `Self-Check` mode (`Quiz` mode is not available for personal collections).

## User stories

1. As a User, I want to create and manage private `Collection` decks and add `Self-Check` questions one by one, so that I can practice my own interview notes alongside public topics.
2. As a User, I want to upload a `.md` or `.txt` file of questions and answers and preview parsed items with validation errors before saving, so that I can import my existing study list quickly.
3. As a User, I want the importer to detect exact and similar duplicate questions and let me choose whether to add or skip each flagged item, so that my collections stay clean without silent data loss.

## Rough acceptance criteria

- **Access control:** `Guest` requests to list, create, update, delete, or import personal `Collection` data return `401`/`403` JSON errors; one `User` can never read or modify another user's `Collection`.
- **Single-item CRUD:** `User` can create, edit, and delete a `Collection` and questions inside it; empty question prompt or empty answer returns `400`.
- **File import preview:** Uploading a valid `.md` or `.txt` file returns parsed candidate items, format errors (if any), and suspected duplicates without writing to the database yet.
- **Duplicate resolution:** Every flagged candidate shows the closest existing question text; the user selects `add` or `skip` per conflict and confirms import.
- **Training compatibility:** Personal `Collection` decks appear in the user's Dashboard and can be selected for `Self-Check` sessions (`Quiz` mode remains disabled for personal collections).

## What it touches

- **Data:** `Collection` (`id`, `ownerId`, `title`, `createdAt`, `updatedAt`), personal `Question` records linked to `collectionId` and `ownerId` (`visibility: 'private'`).
- **API:** CRUD endpoints for `/api/collections` and `/api/collections/:id/questions`, plus `/api/collections/:id/import/preview` and `/api/collections/:id/import/confirm`.
- **Engine:** Deterministic `.md`/`.txt` parser and text-similarity / fuzzy duplicate matcher (no AI required in the first version; AI similarity may be added later if simple matching is not enough).
- **UI:** "My Collections" section on the Dashboard, collection editor, file upload dropzone, and duplicate-review modal/screen.
- **Roles:** Available to authenticated `User` only.

## Decisions the MVP must keep open

- Keep `Question` ownership/visibility (`public` vs `private`, `ownerId` nullable) and deck/topic abstraction clean in the data model so personal `Collection` decks plug into the existing training `Session`, progress tracking (`New` / `Known` / `For Review` / `Unknown`), and `Readiness` calculation without a rewrite.
- Keep `Self-Check` independent of `quizOptions` and reuse the `quizEnabled: false` pattern so personal `Collection` decks naturally allow `Self-Check` without `Quiz`.
