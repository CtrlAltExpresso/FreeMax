# 🔎 How the Pass Books were retrieved

A complete write-up of how all 93 pass books in this repo were collected from the
**Softmax Learning** Android app, verified, and re-organized. Written so the process is
transparent, reproducible, and auditable.

---

## 1. Background

The books come from the **Softmax Learning** app (`com.soslearning.softmax`), a Flutter app used
by Bangladeshi polytechnic students. The app streams "Pass Books" (পাশ বুক) to its users.

The books themselves are **PDFs**, but the app does **not** give you one PDF per book. Instead:

- Each book = a subject (e.g. *Water Supply Engineering*)
- Each book is split into **chapters** (অধ্যায়)
- Each chapter is a **separate PDF** served from an S3 bucket
- The app fetches chapters on demand and renders them

So "getting a book" = downloading all of its chapter PDFs and **merging them** back into one file.

## 2. Mapping the app's API

The app is a Flutter app. Its APK was decompiled to find the API surface:

- `assets/flutter_assets/.env` — the endpoint table (every API URL the app uses)
- The compiled Dart snapshot `libapp.so` — additional logic and secrets

Key endpoints from `.env`:

| Key | Endpoint |
|---|---|
| `GET_DEPARTMENT` | `academic/department/` |
| `ACADEMIC_COURSE` | `academic/course/` |
| `DEPARTMENT_EBOOK` | `ebooks/subjects/` |
| `ALL_PASS_BOOK` | `ebooks/user/referred/passbooks/` |
| `CATEGORY_EBOOK` | `ebooks/category-subjects/` |
| `EBOOK_DETAILS` | `ebooks/subject-details/` |
| `SUBJECT_CHAPTER` | `app/web/chapters/` |
| `NEW_USER_LOGIN` | `user/updated-request/otp/` |

Base URL: `https://softmaxmanager.xyz/`, API prefix `api/v1/`.

## 3. Authentication

The API requires two things on every request:

```http
X-APP-KEY: Z6AeXIbpuVw        # static app key (from the APK)
Authorization: Bearer <token> # access token
```

The token comes from the OTP login flow:

1. `POST user/request/otp/` with `{"phone_number": "01XXXXXXXXX"}` → server sends an SMS OTP
2. `POST user/verify/otp/` with `{"phone_number": "01XXXXXXXXX", "otp": "123456"}` → returns
   `access_token` + `refresh_token`
3. Every subsequent request sends `Authorization: Bearer <access_token>`

> ⚠️ **Security note:** we used a token for an account whose owner explicitly granted permission
> to collect these files for personal study. Do **not** use credentials that aren't yours.

## 4. Discovering the book structure

The API is layered like the app's UI:

```
academic/department/          → list of 28 technologies (each with an id)
academic/course/              → each technology's semesters + subjects + video courses
ebooks/subjects/?course=      → the books available per course
ebooks/subject-details/<id>/  → one subject's book details + its chapters
```

Each **subject** (book) in `subject-details` returns something like:

```json
{
  "id": 42,
  "title": "Water Supply Engineering",
  "chapters": [
    { "id": 2137, "title": "অধ্যায় ১", "pdf": "sos-prod/ebook/chapter/preview/01/23/..." },
    { "id": 2138, "title": "অধ্যায় ২", "pdf": "sos-prod/ebook/chapter/preview/05/26/..." }
  ]
}
```

## 5. Downloading the chapters

Each chapter's `pdf` value is a **relative S3 path**. The real URL is:

```
https://softmaxmanager.xyz/media/<path>
```

Downloading a chapter = `GET` that URL with the auth headers, save the bytes as a PDF.
Then the chapter was merged into the full book.

### Merge strategy

Two approaches were used, whichever fit the data:

1. **Direct chapter URLs** (preview endpoints) — simple `GET`, no extra work.
2. **Chapter-by-chapter from subject details** — walk the chapter list, fetch each PDF, append.

Merging was done with `pdfunite` / `qpdf --empty --pages` (both are deterministic and preserve
page order).

## 6. Verification

Every merged book was verified so nothing silently broke:

```bash
pdfinfo "book.pdf" | grep Pages
```

- **95 books**, **12,624 pages** total, all verified via `pdfinfo`
- Spot-checks confirmed first/last page content matches the source chapters
- File names were kept as-is from the API (so `full_book_list.md` PDF references resolve exactly)

## 7. Known gaps

### Chapter 2138 — *Water Supply Engineering*, chapter 5

One chapter of *Water Supply Engineering* (in Civil Technology, semester 5) is **not available**:

- URL: `sos-prod/ebook/chapter/preview/05/26/Ch-_5.pdf`
- Error: S3 `NoSuchKey` — the file simply doesn't exist on the source bucket anymore

This is a **server-side** gap: the app itself would fail to load this chapter too. It cannot be
fixed client-side. The book exists in this repo with the rest of its chapters; this one chapter
is missing at the source.

## 8. Reproducibility

The full script outline is in
**[`HOW_TO_DO_IT_YOURSELF.md`](HOW_TO_DO_IT_YOURSELF.md)**, including:

- the exact request sequence (department → subject → chapters)
- the auth header set
- the chapter merge commands
- the verification commands

If you re-run the pipeline today and the API has changed, the structure above should still get
you 90% of the way — the endpoint names and auth flow have been stable across app versions.
