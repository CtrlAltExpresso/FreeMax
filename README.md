<div align="center">

# 📚 FreeMax

**Complete Pass Book library + YouTube lecture video index for Bangladeshi Diploma Engineering students**

**717 pass book PDFs** + **42,468 YouTube lecture videos** across **35 technologies** and
**34 departments**, organized exactly like the Softmax Learning app
(**Technology → Semester → Subject**), plus a full write-up of how they were collected.

[![Books](https://img.shields.io/badge/books-717-16a34a?style=flat-square)](#-catalog)
[![Technologies](https://img.shields.io/badge/technologies-35-0f172a?style=flat-square)](#-catalog)
[![Videos](https://img.shields.io/badge/videos-42%2C468-f59e0b?style=flat-square)](#-video-links)
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
│   ├── Semester 1/
│   │   ├── Bangla-I.pdf
│   │   ├── Mathematics -I.pdf
│   │   └── Basic Electricity.pdf
│   ├── Semester 2/
│   └── ...
├── 1. Automobile Technology 🚗/
│   ├── Semester 1/
│   └── ...
└── 27. Footwear Technology 👟/
```

Each technology contains `Semester 1` … `Semester 7` folders. Every **✅** subject in
[`full_book_list.md`](full_book_list.md) has its PDF in the matching folder. Subjects marked **❌**
don't have a pass book yet (shown as "coming soon" in the app).

> **Note on duplicates:** The same PDF (e.g. *Bangla-I*, *Social Science*) appears in many
> technologies' curricula. Each copy is stored independently, so the repo is ~4.5 GB.

## 📊 Catalog

The master syllabus is [`full_book_list.md`](full_book_list.md) — every technology, semester,
subject, and PDF. You can also browse:

| Folder | Contents |
|---|---|
| `Books/` | All 717 pass book PDFs, organized by technology → semester |
| `Videos/` | 42,468 YouTube lecture links — markdown files organized by department → course → subject |
| `docs/` | How the books + videos were collected, plus full API documentation |

### Quick facts

- **717 pass book PDFs** across 35 technologies (many duplicated across technologies)
- **4,500+ MB** of educational content
- **42,468 lecture videos** organized into 165 subject files across 34 departments
- Every PDF verified with `pdfinfo` (page counts correct, files open cleanly)

## 🧭 How it's organized

The layout mirrors the Softmax Learning app exactly, so whatever you see in the app maps 1:1 to a
file here:

```
app:  My Books → 3. Civil Technology → Semester 5 → Water Supply Engineering
repo: Books/3. Civil Technology 🏗️/Semester_5/Water Supply Engineering.pdf
```

If you're a student, just find your technology, your semester, and read.

## 🎬 Video Links

`Videos/` contains **165 markdown files** with **42,468 clickable YouTube lecture links** scraped
from the Softmax app's video metadata API, organized to match the app's course hierarchy:

```
Videos/
├── 💻 COMPUTER SCIENCE & TECHNOLOGY/
│   ├── Semester 4/
│   │   ├── Data Structure and Algorithm (Update).md
│   │   └── Java Programming (Update).md
│   └── Semester 6/
│       ├── Database Management System.md
│       └── ...
├── 🏗️ Civil Technology (SAE)/
│   ├── Civil SAE/          ← job-pattern questions
│   ├── PSC Pattern/        ← PSC exam pattern
│   └── MIST Pattern/
├── ⚡ Electrical Technology (SAE)/
│   └── General (Job)/      ← Bangla, English, Math, GK
└── ...
```

Each `.md` file is a table of videos with title, duration, and clickable `[Watch](YouTube URL)` links.

**Stats:**
| Metric | Count |
|---|---|
| Departments | 34 |
| Courses (folders) | 40+ |
| Subject files | 165 |
| Matched videos | 37,480 |
| Uncategorized videos | 4,988 |
| Total | 42,468 |

> **Note:** ~2% of links may be private/unlisted and require Softmax app access to watch.
> ~4% may be broken (deleted/moved). The rest are publicly accessible YouTube links.

## 🔎 How this data was collected

The PDFs and video links come from the **Softmax Learning app's own API** — the same data the
app streams to paying users. Everything was downloaded with an authenticated session and
reorganized into this clean structure.

The full process is documented in **[`docs/RETRIEVAL_PROCESS.md`](docs/RETRIEVAL_PROCESS.md)**,
including:

- How the app's API was mapped (the `.env` endpoint table)
- How authentication works (`X-APP-KEY`, OTP flow)
- The exact request → response flow for books and videos
- How chapters were stitched into single PDFs and verified
- How 42,468 video metadata records were fetched and matched to subjects

## 🧑‍💻 Do it yourself

Want to pull these yourself, re-run the pipeline, or check for newly added books/videos?
See **[`docs/HOW_TO_DO_IT_YOURSELF.md`](docs/HOW_TO_DO_IT_YOURSELF.md)** — a copy-paste guide with
the exact API calls, scripts, and full step-by-step instructions.

## ⚖️ Disclaimer

This project is for **personal educational use** only. The PDFs are owned by their respective
authors/publishers (provided through the Softmax Learning app). If you are a rights holder and
want content removed, open an issue and it will be taken down promptly.

---

<div align="center">

**Made with 💚 for Bangladeshi diploma students 🇧🇩**

</div>
