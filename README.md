<div align="center">

# 📚 FreeMax

**Complete Pass Book library for Bangladeshi Diploma Engineering students**

93 pass books across **28 technologies**, organized exactly like the Softmax Learning app
(**Technology → Semester → Subject**), plus a full write-up of how they were collected.

[![Books](https://img.shields.io/badge/books-93-16a34a?style=flat-square)](#-catalog)
[![Technologies](https://img.shields.io/badge/technologies-28-0f172a?style=flat-square)](#-catalog)
[![Made for students](https://img.shields.io/badge/made%20for-students-64748b?style=flat-square)](#)

</div>

---

## 🎯 What is this?

Every semester, polytechnic students across Bangladesh buy **Pass Books** (পাশ বুক) — the
question-answer guidebooks that cover the full BTEB diploma syllabus subject by subject.

This repository collects **all of the pass books available inside the Softmax Learning app**,
re-organized into a clean, browsable structure:

```
Books/
├── 0. Architecture 🏛️/
│   ├── Semester_1/
│   │   ├── Bangla-I.pdf
│   │   ├── Mathematics -I.pdf
│   │   └── Basic Electricity.pdf
│   ├── Semester_2/
│   └── ...
├── 1. Automobile Technology 🚗/
│   ├── Semester_1/
│   └── ...
└── 27. Footwear Technology 👟/
```

Each technology contains `Semester_1` … `Semester_7` folders. Every **✅** subject in
[`full_book_list.md`](full_book_list.md) has its PDF in the matching folder. Subjects marked **❌**
don't have a pass book yet (shown as "coming soon" in the app).

> **Note on duplicates:** one physical PDF (e.g. *Bangla-I*, *Social Science*) appears in many
> technologies' curricula. Only one real file is stored per PDF; the other locations are
> **symlinks** to it, so the repo stays small (~600 MB) instead of 3.4 GB.

## 📊 Catalog

The master syllabus is [`full_book_list.md`](full_book_list.md) — every technology, semester,
subject, and PDF. You can also browse:

| Folder | PDFs |
|---|---|
| `Books/` | All 93 pass books, organized by technology → semester |
| `Extras/` | 12 bonus books not in the pass-book catalog (Clean Code, Programming, etc.) |
| `docs/` | How the books were collected + how to do it yourself |

### Quick facts

- **93 unique PDFs** (the complete set referenced in `full_book_list.md`)
- **527 placements** across all 28 technologies' semesters
- **12 bonus PDFs** in `Extras/`
- Every PDF verified with `pdfinfo` (page counts correct, files open cleanly)

## 🧭 How it's organized

The layout mirrors the Softmax Learning app exactly, so whatever you see in the app maps 1:1 to a
file here:

```
app:  My Books → 3. Civil Technology → Semester 5 → Water Supply Engineering
repo: Books/3. Civil Technology 🏗️/Semester_5/Water Supply Engineering.pdf
```

If you're a student, just find your technology, your semester, and read.

## 🔎 How this data was collected

This wasn't scraped from some random site — the PDFs come from the **Softmax Learning app's own
API** (the same files the app streams to paying users), downloaded with an authenticated session
and reassembled chapter-by-chapter into complete books.

The full, step-by-step process is documented in **[`docs/RETRIEVAL_PROCESS.md`](docs/RETRIEVAL_PROCESS.md)**,
including:

- How the app's API was mapped (the `.env` endpoint table)
- How authentication works (`X-APP-KEY`, OTP flow)
- The exact request → response flow that returns each book's chapters
- How chapters were stitched into single PDFs and verified
- Why **one book chapter is missing** and what to do about it

## 🧑‍💻 Do it yourself

Want to pull these yourself, re-run the pipeline, or check for newly added books?
See **[`docs/HOW_TO_DO_IT_YOURSELF.md`](docs/HOW_TO_DO_IT_YOURSELF.md)** — a copy-paste guide with
the exact API calls and a script outline.

## ⚖️ Disclaimer

This project is for **personal educational use** only. The PDFs are owned by their respective
authors/publishers (provided through the Softmax Learning app). If you are a rights holder and
want content removed, open an issue and it will be taken down promptly.

---

<div align="center">

**Made with 💚 for Bangladeshi diploma students 🇧🇩**

</div>
