<div align="center">

# 📚 FreeMax

**Complete Pass Book library + YouTube lecture video index for Bangladeshi Diploma Engineering students**

**628 pass book PDFs** + **42,574 YouTube lecture videos** across **28 technologies**,
organized to match the official BTEB course structure from `a.txt`
(**Technology → Semester → Subject**), plus a full write-up of how they were collected.

[![Books](https://img.shields.io/badge/books-628-16a34a?style=flat-square)](#-catalog)
[![Technologies](https://img.shields.io/badge/technologies-28-0f172a?style=flat-square)](#-catalog)
[![Videos](https://img.shields.io/badge/videos-42%2C574-f59e0b?style=flat-square)](#-video-links)
[![Made for students](https://img.shields.io/badge/made%20for-students-64748b?style=flat-square)](#)

</div>

---

## 🎯 What is this?

Every semester, polytechnic students across Bangladesh buy **Pass Books** (পাশ বুক) — the
question-answer guidebooks that cover the full BTEB diploma syllabus subject by subject.

This repository collects **all of the pass books available inside the Softmax Learning app**,
re-organized into a clean, browsable structure that matches the official BTEB course structure:

```
Books/
├── Architecture Technology 🏛️/
│   ├── Semester 1/
│   │   ├── Bangla-I.pdf
│   │   ├── Mathematics -I.pdf
│   │   └── Basic Electricity.pdf
│   ├── Semester 2/
│   └── ...
├── Automobile Technology 🚗/
│   ├── Semester 1/
│   └── ...
└── ... (28 technologies total)
```

Each technology contains `Semester 1` … `Semester 7` folders. The semester assignments are
verified against the official BTEB course structure (`a.txt`). Subjects marked **❌** in the
course structure don't have a pass book yet (shown as "coming soon" in the app).

> **Note on duplicates:** The same PDF (e.g. *Bangla-I*, *Social Science*) appears in many
> technologies' curricula. Each copy is stored independently, so the repo is ~4.3 GB.

## 📊 Catalog

The master syllabus is [`full_book_list.md`](full_book_list.md) — every technology, semester,
subject, and PDF. You can also browse:

| Folder | Contents |
|---|---|
| `Books/` | 628 pass book PDFs, organized by technology → semester (matches BTEB syllabus) |
| `Extras/` | 46 extra PDFs not matched to the BTEB syllabus (general CS books, duplicates, etc.) |
| `Videos/` | 42,574 YouTube lecture links — markdown files organized by BTEB department → semester → subject |
| `QuestionBanks/` | 74 question-bank PDFs merged by subject (BTEB diploma question papers) |
| `docs/` | How the books + videos were collected, plus full API documentation |

### Quick facts

- **628 pass book PDFs** across 28 technologies, organized to match the official BTEB course structure
- **4,300+ MB** of educational content
- **42,574 lecture videos** organized into 684 subject files across 33 BTEB departments
- **74 question-bank PDFs** merged by subject (BTEB diploma question papers)
- Every PDF verified with `pdfinfo` (page counts correct, files open cleanly)

## 🧭 How it's organized

The layout mirrors the official BTEB course structure, so whatever semester a subject belongs in
maps 1:1 to a file here:

```
BTEB:  Civil Technology → Semester 5 → Water Supply Engineering
repo: Books/Civil Technology 🏗️/Semester 5/Water Supply Engineering.pdf
```

If you're a student, just find your technology, your semester, and read.

## 🎬 Video Links

`Videos/` contains **685 markdown files** with **42,574 clickable YouTube lecture links** scraped
from the Softmax app's video metadata API. The BTEB diploma lectures are organized to match the
official BTEB course structure, and everything outside the diploma syllabus is kept in its own
separate folder:

```
Videos/
├── BTEB Diploma/
│   ├── Aircraft Maintenance Technology- Aerospace/
│   │   ├── Semester 1/
│   │   │   ├── Bangla-I.md
│   │   │   └── Basic Electricity.md
│   │   └── ...
│   ├── Computer Science & Technology/
│   │   ├── Semester 4/
│   │   │   ├── Data Structure & Algorithm.md
│   │   │   └── Java Programming.md
│   │   └── ...
│   └── ... (33 departments, Semester 1 through 7/8)
├── DUET Admission/          ← DUET university admission lectures
├── Job Preparation/         ← SAE / govt-job pattern lectures
├── Polytechnic Admission/   ← polytechnic entry-test prep
├── 9-10 School/             ← school-level content
├── ICT Softmax/             ← HSC ICT course
├── Soft Skills/             ← career guidance
├── Smart Book Videos/       ← supplementary "smart book" videos
└── Other/Unclassified/      ← videos with no collection metadata
```

Each `.md` file is a table of videos with title, duration, and clickable `[Watch](YouTube URL)` links.
Diploma subjects that appear in many technologies (Bangla, English, Mathematics, Physics, …) are
duplicated into every department that teaches them, matching the official syllabus layout 1:1.

**Stats:**
| Metric | Count |
|---|---|
| BTEB diploma departments | 33 |
| BTEB diploma subject files | 538 |
| Non-BTEB / misc subject files | 147 |
| Total subject files | 685 |
| BTEB diploma videos | 27,592 |
| DUET admission videos | 5,350 |
| Job preparation videos | 3,796 |
| Other non-BTEB videos | 829 |
| Uncategorized videos | 5,007 |
| Total | 42,574 |

> **Note:** ~2% of links may be private/unlisted and require Softmax app access to watch.
> ~4% may be broken (deleted/moved). The rest are publicly accessible YouTube links.

## 📝 Question Banks

`QuestionBanks/` holds **74 BTEB diploma question-bank PDFs**, each merged by subject across
institutes and years (689 exam-paper pages total). Unlike `Books/`, these are standalone subject
question papers rather than pass books, so they live in a flat subject-keyed folder:

```
QuestionBanks/
├── Mathematics 3.pdf
├── Java Application Development.pdf
├── Computer Architecture and Microprocessor.pdf
├── Basic Electronics.pdf
└── ... (74 subjects)
```

Every file is named for its subject in plain English (e.g. *Mathematics 3*, *Physics 1*,
*Electrical Circuit 2*). They came from the same Softmax app API as the pass books and videos
(`web/question-banks` endpoint), and were merged from 433 individual per-institute papers.

## 🔎 How this data was collected

The PDFs and video links come from the **Softmax Learning app's own API** — the same data the
app streams to paying users. Everything was downloaded with an authenticated session and
reorganized into this clean structure.

The semester assignments were verified against the **official BTEB course structure** (`a.txt`),
which lists every department, semester, subject code, and subject name for the Diploma in
Engineering program. 18 PDFs that didn't match any BTEB subject were moved to `Extras/`.

The full process is documented in **[`docs/RETRIEVAL_PROCESS.md`](docs/RETRIEVAL_PROCESS.md)**,
including:

- How the app's API was mapped (the `.env` endpoint table)
- How authentication works (`X-APP-KEY`, OTP flow)
- The exact request → response flow for books and videos
- How chapters were stitched into single PDFs and verified
- How 42,574 video metadata records were fetched and matched to subjects

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
