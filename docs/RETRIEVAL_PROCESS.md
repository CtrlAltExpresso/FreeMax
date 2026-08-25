# 🔎 How everything was collected

A complete write-up of how all **628 pass book PDFs** and **42,468 video links** in this repo
were collected from the **Softmax Learning** Android app, verified, and reorganized. Written so
the process is transparent, reproducible, and auditable.

---

## Table of contents

1. [Background](#1-background)
2. [Phase 1: APK analysis & API mapping](#2-phase-1-apk-analysis--api-mapping)
3. [Phase 2: Authentication](#3-phase-2-authentication)
4. [Phase 3: Book extraction](#4-phase-3-book-extraction)
5. [Phase 4: Video metadata extraction](#5-phase-4-video-metadata-extraction)
6. [Phase 5: Folder structure building](#6-phase-5-folder-structure-building)
7. [Known gaps](#7-known-gaps)

---

## 1. Background

The data comes from the **Softmax Learning** app (`com.soslearning.softmax`), a Flutter app used
by Bangladeshi polytechnic students. The app provides:

- **Pass Books** (পাশ বুক) — PDFs split into chapters, served from S3
- **Lecture Videos** — YouTube-hosted videos organized by subject
- **Question Banks, E-Books, Live Classes, etc.** — other content

The app is a Flutter app. Its compiled code and config files reveal the entire API surface.

## 2. Phase 1: APK analysis & API mapping

### 2a. Decompile the APK

```bash
apktool d base.apk -o base_dec
```

### 2b. Extract the `.env` file (endpoint table)

```bash
cat base_dec/assets/flutter_assets/.env
```

This contains every API URL the app uses. Key endpoints:

| Purpose | Endpoint |
|---|---|
| Departments (technologies) | `academic/department/` |
| Courses (semesters+subjects) | `academic/course/` |
| Department books | `ebooks/subjects/` |
| Book chapters | `ebooks/subject-details/<id>/` |
| E-chapters (user's purchases) | `ebooks/esubject/user/` |
| Pass books | `ebooks/user/referred/passbooks/` |
| Categories | `ebooks/category-subjects/` |
| Videos | `app/web/videos/` |
| Chapters (with descriptions) | `app/web/chapters/` |
| Subjects list | `app/web/subjects/` |
| Courses (with subject mapping) | `app/web/courses/<id>/` |
| Departments (web) | `app/web/departments/` |
| Courses (web) | `app/web/courses/` |

Base URL: `https://softmaxmanager.xyz/api/v1/`

### 2c. Extract the Flutter binary for secrets

```bash
# The Dart compiled snapshot contains hardcoded strings
strings libapp.so | grep -i "key\|secret\|token\|cert"
```

Found values:
- `X-APP-KEY`: `l0dtpwvzzmM` (the app sends this on every request)
- `BACKEND_CERT`: `f4266b644bf2d2bce30b12f40423d5f0b94a7b32f9963066a7ee94645230a940`
- App version: `2.3.13` (code `10012034`)
- Package: `com.soslearning.softmax`

### 2d. Discover the data hierarchy

The API mirrors the app's UI:

```
academic/department/          → list of 49 departments (with ids)
app/web/courses/              → 265 courses (each belongs to a department)
app/web/courses/<id>/         → course details + subjects array
app/web/subjects/             → 849 subjects (with names + codes)
app/web/chapters/             → 7,190 chapters (linked to subjects)
app/web/videos/               → 42,468 video records (with YouTube links)
```

## 3. Phase 2: Authentication

Every request requires two headers:

```http
X-APP-KEY: l0dtpwvzzmM
Authorization: Bearer <token>
```

Also needed for Cloudflare bypass (without this, requests get 403):

```http
User-Agent: Dart/3.2 (dart:io)
```

### OTP flow

```bash
BASE=https://softmaxmanager.xyz/api/v1

# 1. Request OTP (sends SMS to your phone)
curl -s "$BASE/user/request/otp/" \
  -H "X-APP-KEY: l0dtpwvzzmM" \
  -H "Content-Type: application/json" \
  -H "User-Agent: Dart/3.2 (dart:io)" \
  -d '{"phone_number":"01XXXXXXXXX"}'

# 2. Verify OTP → returns access_token
TOKEN=$(curl -s "$BASE/user/verify/otp/" \
  -H "X-APP-KEY: l0dtpwvzzmM" \
  -H "Content-Type: application/json" \
  -H "User-Agent: Dart/3.2 (dart:io)" \
  -d '{"phone_number":"01XXXXXXXXX","otp":"123456"}' | jq -r .access_token)

# 3. Save token for reuse
echo "$TOKEN" > .token
```

**Important:** Rate limits exist — 10 OTP requests per 2 days per phone number.
Tokens last ~1 year. Store them safely.

## 4. Phase 3: Book extraction

### 4a. Fetch department and course lists

```bash
curl -s "$BASE/app/web/departments/" -H "authorization: Bearer $TOKEN" \
  -H "x-app-key: l0dtpwvzzmM" -H "User-Agent: Dart/3.2 (dart:io)" > departments.json

curl -s "$BASE/app/web/courses/" -H "authorization: Bearer $TOKEN" \
  -H "x-app-key: l0dtpwvzzmM" -H "User-Agent: Dart/3.2 (dart:io)" > courses.json
```

### 4b. For each course, get its subjects

```bash
# Course 39 = "3rd Semester CST" → returns subjects with IDs
curl -s "$BASE/app/web/courses/39/" -H "authorization: Bearer $TOKEN" \
  -H "x-app-key: l0dtpwvzzmM" -H "User-Agent: Dart/3.2 (dart:io)"
```

### 4c. For each subject, get its chapters

```bash
# Subject 88 = "Physics-I" → returns 16 chapters with S3 paths
curl -s "$BASE/app/web/chapters/?subject=88&limit=200" \
  -H "authorization: Bearer $TOKEN" \
  -H "x-app-key: l0dtpwvzzmM" -H "User-Agent: Dart/3.2 (dart:io)"
```

### 4d. Download and merge chapter PDFs

Each chapter has a `chapter_description` array pointing to PDF files:

```bash
# Download a chapter PDF
curl -s -o chapter.pdf \
  "https://softmaxmanager.xyz/media/sos-prod/ebook/chapter/preview/..." \
  -H "authorization: Bearer $TOKEN" \
  -H "x-app-key: l0dtpwvzzmM" -H "User-Agent: Dart/3.2 (dart:io)"

# Merge all chapters into one book
qpdf --empty --pages ch_*.pdf -- Full_Book.pdf

# Verify
pdfinfo Full_Book.pdf | grep Pages
```

### 4e. Organize into Books/ folder

```
Books/
├── Computer Science & Technology/
│   ├── Semester 1/
│   │   ├── Bangla-I.pdf
│   │   └── Mathematics -I.pdf
│   ├── Semester 2/
│   └── ...
└── ...
```

The master list of all 628 PDFs and their correct locations is in `full_book_list.md`.

## 5. Phase 4: Video metadata extraction

### 5a. Fetch all video records

The `app/web/videos/` endpoint returns videos with pagination (200 per page):

```bash
python3 << 'EOF'
import json, subprocess, time

TOKEN = open('.token').read().strip()
KEY = 'l0dtpwvzzmM'
BASE = "https://softmaxmanager.xyz/api/v1"
UA = "user-agent: Dart/3.2 (dart:io)"

all_videos = []
offset = 0
PAGE = 200

while True:
    url = f"{BASE}/app/web/videos/?limit={PAGE}&offset={offset}"
    cmd = ['curl','-s','-m','30', url,
           '-H', f'authorization: Bearer {TOKEN}',
           '-H', f'x-app-key: {KEY}',
           '-H', UA, '-H', 'accept: application/json']
    r = subprocess.run(cmd, capture_output=True, text=True, timeout=45)
    data = json.loads(r.stdout)
    results = data.get('results', [])
    if not results: break
    all_videos.extend(results)
    if len(all_videos) >= data.get('count', 0): break
    offset += PAGE
    time.sleep(0.3)

json.dump(all_videos, open('all_videos.json', 'w'), ensure_ascii=False)
print(f"Fetched {len(all_videos)} videos")
EOF
```

Each video record contains:
- `id` — unique video ID
- `title` — video title
- `youtube_link` — YouTube URL
- `duration` — duration in seconds
- `order` — chapter/video order number
- `tags` — list of topic tags
- `bunny_collection` — `{"label": "Subject Name", ...}` (the subject grouping)

### 5b. Match videos to subjects

The video metadata doesn't have a direct `subject_id`. Instead, videos are grouped by
`bunny_collection.label` (e.g., "Digital Electronics", "Mathematics 01").

To map videos to the app's hierarchy:

```bash
python3 << 'EOF'
import json
from difflib import SequenceMatcher
from collections import defaultdict

# Load data
videos = json.load(open('all_videos.json'))
courses = json.load(open('course_subjects_map.json'))  # from Phase 3

# Build subject → course mapping
subj_to_course = {}
for cid, info in courses.items():
    for s in info.get('subjects', []):
        subj_to_course[s['value']] = {
            'name': s['label'],
            'dept': info['dept_name'],
            'course': info['name'],
        }

# Group videos by bunny_collection label
coll_vids = defaultdict(list)
for v in videos:
    bc = v.get('bunny_collection')
    if bc and bc.get('label'):
        coll_vids[bc['label']].append(v)

# Fuzzy-match each label to a subject name
for label, vids in coll_vids.items():
    best_score = 0
    best = None
    for sid, info in subj_to_course.items():
        score = SequenceMatcher(None, label.lower(), info['name'].lower()).ratio()
        if score > best_score:
            best_score = score
            best = info
    if best_score > 0.5:
        # This label belongs to best['dept'] → best['course'] → best['name']
        pass
EOF
```

### 5c. Build markdown files

For each matched subject, create a `.md` file with a table of video links:

```markdown
# Database Management System

> 403 videos | COMPUTER SCIENCE & TECHNOLOGY

| # | Title | Duration | Link |
|---|-------|----------|------|
| 1 | Chapter Overview | 0:04 | [Watch](https://youtu.be/...) |
| 2 | Transaction | 0:12 | [Watch](https://youtu.be/...) |
```

### 5d. Organize into Videos/ folder

The folder structure mirrors the Softmax app's course hierarchy:

```
Videos/
├── 💻 COMPUTER SCIENCE & TECHNOLOGY/
│   ├── Semester 4/
│   │   └── Data Structure and Algorithm.md
│   └── Semester 6/
│       └── Database Management System.md
├── 🏗️ Civil Technology (SAE)/
│   ├── Civil SAE/          ← job-pattern questions
│   ├── PSC Pattern/        ← PSC exam pattern
│   └── MIST Pattern/
└── ⚡ Electrical Technology (SAE)/
    └── General (Job)/      ← Bangla, English, Math, GK
```

**Non-semester courses** (SAE patterns, DUET Dreamer, Target-SAE, etc.) get their own folders
under the department, matching how the app organizes them as separate tabs/courses.

## 6. Phase 5: Folder structure building

The build script (`scripts/build_videos_md.py`) automates everything:

```bash
python3 scripts/build_videos_md.py
```

It:
1. Loads video metadata + course/subject mappings
2. Matches bunny_collection labels to subjects via fuzzy matching
3. Groups by department → course → subject
4. Generates one `.md` file per subject
5. Prints stats (departments, courses, subjects, video counts)

## 7. Known gaps

### Books
- **628 PDFs** across 28 technologies — all verified
- One chapter (*Water Supply Engineering*, chapter 5) is missing server-side (S3 `NoSuchKey`)
- All 628 PDFs are stored as independent copies (no symlinks), so the repo is ~3.7 GB

### Videos
- **42,468 videos** fetched from the API
- **37,480 matched** to subjects (88%)
- **4,988 unmatched** — these have no `bunny_collection` label or couldn't be fuzzy-matched
- ~2% of YouTube links may be private/unlisted
- ~4% may be broken (deleted/moved)
- The API defines 1,704 subject entries, but many are duplicates across years/patterns
  (e.g., "DUET Dreamer 2025" and "DUET Dreamer 2026" share most subjects)

### Authentication
- Both phone numbers used during collection are now OTP-rate-limited
- JWT signing key is server-side (Django SECRET_KEY) — cannot be cracked client-side
- Tokens last ~1 year; refresh before expiry
