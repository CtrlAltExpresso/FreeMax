# 🧑‍💻 Complete step-by-step tutorial

A full, copy-paste guide to reproduce everything in this repo: extracting pass books and
video metadata from the **Softmax Learning** app, building the folder structure, and pushing
to GitHub. Written so you never have to figure things out the hard way.

> **Before you start:** You need a phone number that can receive SMS OTPs. The app's data
> is provided under the app's own terms. Use it only for content you're allowed to access
> and for personal study.

---

## Table of contents

- [Prerequisites](#prerequisites)
- [Step 1: Decompile the APK](#step-1-decompile-the-apk)
- [Step 2: Find the API endpoints and secrets](#step-2-find-the-api-endpoints-and-secrets)
- [Step 3: Get an authentication token](#step-3-get-an-authentication-token)
- [Step 4: Fetch all API metadata](#step-4-fetch-all-api-metadata)
- [Step 5: Extract pass books (PDFs)](#step-5-extract-pass-books-pdfs)
- [Step 6: Extract video metadata](#step-6-extract-video-metadata)
- [Step 7: Build the folder structures](#step-7-build-the-folder-structures)
- [Step 8: Push to GitHub](#step-8-push-to-github)
- [Troubleshooting](#troubleshooting)
- [Full API reference](#full-api-reference)

---

## Prerequisites

```bash
# Ubuntu/Debian
sudo apt install jq curl qpdf pdfinfo python3 git apktool

# Arch
sudo pacman -S jq curl qpdf poppler python git

# macOS
brew install jq curl qpdf poppler python3 git
```

You also need:
- The Softmax APK file (download from device or APK mirror)
- A phone number for OTP login
- A GitHub account (for pushing)

---

## Step 1: Decompile the APK

The APK contains the entire API surface — endpoint URLs, secrets, app config.

```bash
# Decompile with apktool
apktool d base.apk -o base_dec

# The key files:
cat base_dec/assets/flutter_assets/.env          # API endpoint table
strings base_dec/lib/arm64-v8a/libapp.so | head  # Flutter binary with secrets
```

### What you'll find in `.env`

The `.env` file is a key=value list of every API endpoint the app uses. Key ones:

```
GET_DEPARTMENT=academic/department/
ACADEMIC_COURSE=academic/course/
DEPARTMENT_EBOOK=ebooks/subjects/
EBOOK_DETAILS=ebooks/subject-details/
SUBJECT_CHAPTER=app/web/chapters/
ALL_VIDEOS=app/web/videos/
SUBJECT_LIST=app/web/subjects/
COURSE_DETAIL=app/web/courses/
```

Base URL: `https://softmaxmanager.xyz/api/v1/`

### What you'll find in the binary

```bash
# Find the app key
strings base_dec/lib/arm64-v8a/libapp.so | grep -i "app.key\|x-app-key"
# → l0dtpwvzzmM

# Find the backend cert
strings base_dec/lib/arm64-v8a/libapp.so | grep -i "backend.cert\|BACKEND_CERT"
# → f4266b644bf2d2bce30b12f40423d5f0b94a7b32f9963066a7ee94645230a940
```

Save these:
```bash
mkdir -p ~/softmax_dbg/api_data
echo "l0dtpwvzzmM" > ~/softmax_dbg/api_data/.key
```

---

## Step 2: Find the API endpoints and secrets

The `.env` file gives you the endpoint paths. The full API surface is:

| What | Endpoint | Method |
|---|---|---|
| Request OTP | `user/request/otp/` | POST |
| Verify OTP | `user/verify/otp/` | POST |
| Departments | `academic/department/` | GET |
| Courses | `academic/course/` | GET |
| Ebooks by course | `ebooks/subjects/?course=<id>` | GET |
| Book details | `ebooks/subject-details/<id>/` | GET |
| Chapters | `app/web/chapters/?subject=<id>` | GET |
| Videos | `app/web/videos/` | GET (paginated) |
| Subjects | `app/web/subjects/` | GET (paginated) |
| Course details | `app/web/courses/<id>/` | GET |
| Departments (web) | `app/web/departments/` | GET |
| Courses (web) | `app/web/courses/` | GET |

**Critical:** You MUST send these headers on every request or Cloudflare will block you:

```http
User-Agent: Dart/3.2 (dart:io)
X-APP-KEY: l0dtpwvzzmM
Authorization: Bearer <token>
```

---

## Step 3: Get an authentication token

```bash
BASE=https://softmaxmanager.xyz/api/v1
KEY=l0dtpwvzzmM
PHONE=01XXXXXXXXX    # your phone number
UA="User-Agent: Dart/3.2 (dart:io)"

# 1. Request OTP (sends SMS)
curl -s "$BASE/user/request/otp/" \
  -H "X-APP-KEY: $KEY" \
  -H "Content-Type: application/json" \
  -H "$UA" \
  -d "{\"phone_number\":\"$PHONE\"}"

# 2. Check your phone for the 6-digit code, then verify
TOKEN=$(curl -s "$BASE/user/verify/otp/" \
  -H "X-APP-KEY: $KEY" \
  -H "Content-Type: application/json" \
  -H "$UA" \
  -d "{\"phone_number\":\"$PHONE\",\"otp\":\"XXXXXX\"}" | jq -r .access_token)

echo "$TOKEN" > ~/softmax_dbg/api_data/.token
echo "Token saved: ${TOKEN:0:20}..."
```

**Rate limits:** 10 OTP requests per 2 days per phone. Tokens last ~1 year.

**If you get 401 later:** The token expired. Re-run step 3 to get a new one.

---

## Step 4: Fetch all API metadata

Save this as `fetch_all_metadata.sh` and run it:

```bash
#!/bin/bash
BASE=https://softmaxmanager.xyz/api/v1
TOKEN=$(cat ~/softmax_dbg/api_data/.token)
KEY=$(cat ~/softmax_dbg/api_data/.key)
UA="User-Agent: Dart/3.2 (dart:io)"
OUT=~/softmax_dbg/api_data

auth=(-H "authorization: Bearer $TOKEN" -H "x-app-key: $KEY" -H "$UA" -H "accept: application/json")

echo "Fetching departments..."
curl -s "$BASE/app/web/departments/" "${auth[@]}" | python3 -m json.tool > "$OUT/01_departments.json"

echo "Fetching courses..."
curl -s "$BASE/app/web/courses/?limit=300" "${auth[@]}" | python3 -m json.tool > "$OUT/02_courses.json"

echo "Fetching subjects (paginated)..."
python3 -c "
import json, subprocess, time
TOKEN = open('$OUT/.token').read().strip()
KEY = open('$OUT/.key').read().strip()
UA = 'User-Agent: Dart/3.2 (dart:io)'
BASE = 'https://softmaxmanager.xyz/api/v1'
all_items = []
offset = 0
while True:
    url = f'{BASE}/app/web/subjects/?limit=200&offset={offset}'
    r = subprocess.run(['curl','-s','-m','30', url,
        '-H', f'authorization: Bearer {TOKEN}',
        '-H', f'x-app-key: {KEY}', '-H', UA, '-H', 'accept: application/json'],
        capture_output=True, text=True, timeout=45)
    data = json.loads(r.stdout)
    results = data.get('results', [])
    if not results: break
    all_items.extend(results)
    if len(all_items) >= data.get('count', 0): break
    offset += 200
    time.sleep(0.3)
json.dump(all_items, open('$OUT/subjects_849.json', 'w'), ensure_ascii=False)
print(f'  Fetched {len(all_items)} subjects')
"

echo "Fetching course→subject mappings..."
python3 -c "
import json, subprocess, time
TOKEN = open('$OUT/.token').read().strip()
KEY = open('$OUT/.key').read().strip()
UA = 'User-Agent: Dart/3.2 (dart:io)'
BASE = 'https://softmaxmanager.xyz/api/v1'
courses = json.load(open('$OUT/02_courses.json'))
if isinstance(courses, dict) and 'data' in courses: courses = courses['data']
result = {}
for c in courses:
    cid = c['id']
    r = subprocess.run(['curl','-s','-m','15', f'{BASE}/app/web/courses/{cid}/',
        '-H', f'authorization: Bearer {TOKEN}',
        '-H', f'x-app-key: {KEY}', '-H', UA, '-H', 'accept: application/json'],
        capture_output=True, text=True, timeout=20)
    try:
        data = json.loads(r.stdout)
        result[str(cid)] = {
            'name': c.get('name',''),
            'dept_id': c.get('department'),
            'dept_name': '',  # fill from departments
            'subjects': data.get('subjects', [])
        }
    except: pass
    time.sleep(0.15)
json.dump(result, open('$OUT/course_subjects_map.json', 'w'), ensure_ascii=False)
print(f'  Mapped {len(result)} courses')
"

echo "Fetching video metadata (this takes ~2 minutes)..."
python3 -c "
import json, subprocess, time
TOKEN = open('$OUT/.token').read().strip()
KEY = open('$OUT/.key').read().strip()
UA = 'User-Agent: Dart/3.2 (dart:io)'
BASE = 'https://softmaxmanager.xyz/api/v1'
all_vids = []
offset = 0
PAGE = 200
while True:
    url = f'{BASE}/app/web/videos/?limit={PAGE}&offset={offset}'
    r = subprocess.run(['curl','-s','-m','30', url,
        '-H', f'authorization: Bearer {TOKEN}',
        '-H', f'x-app-key: {KEY}', '-H', UA, '-H', 'accept: application/json'],
        capture_output=True, text=True, timeout=45)
    data = json.loads(r.stdout)
    results = data.get('results', [])
    if not results: break
    all_vids.extend(results)
    total = data.get('count', 0)
    print(f'  [{len(all_vids)}/{total}]', end='\r')
    if len(all_vids) >= total: break
    offset += PAGE
    time.sleep(0.3)
json.dump(all_vids, open('$OUT/all_videos.json', 'w'), ensure_ascii=False)
print(f'  Fetched {len(all_vids)} videos')
"

echo "Done! All metadata saved to $OUT/"
```

---

## Step 5: Extract pass books (PDFs)

### 5a. Understand the book structure

Each book is a subject, split into chapters (PDFs on S3). To get a complete book:
1. Get the subject's chapters: `GET app/web/chapters/?subject=<id>`
2. Each chapter has `chapter_description` IDs → fetch `app/web/chapters/description/<id>/`
3. The description contains the S3 path to the PDF
4. Download all chapter PDFs and merge with `qpdf`

### 5b. Download a book

```bash
TOKEN=$(cat ~/softmax_dbg/api_data/.token)
KEY=$(cat ~/softmax_dbg/api_data/.key)
BASE="https://softmaxmanager.xyz/api/v1"
UA="User-Agent: Dart/3.2 (dart:io)"

SUBJECT_ID=88  # e.g., Physics-I

# Get chapters
curl -s "$BASE/app/web/chapters/?subject=$SUBJECT_ID&limit=200" \
  -H "authorization: Bearer $TOKEN" \
  -H "x-app-key: $KEY" \
  -H "$UA" | jq '.results[].chapter_description' > chapter_ids.json

# For each chapter_description, get the PDF path and download
mkdir -p book_pdfs
i=1
for id in $(jq -r '.[]' chapter_ids.json); do
  curl -s "$BASE/app/web/chapters/description/$id/" \
    -H "authorization: Bearer $TOKEN" \
    -H "x-app-key: $KEY" \
    -H "$UA" | jq -r '.pdf' | while read -r path; do
      curl -s -o "book_pdfs/ch_$(printf '%03d' $i).pdf" \
        "https://softmaxmanager.xyz/media/$path" \
        -H "authorization: Bearer $TOKEN" \
        -H "x-app-key: $KEY" \
        -H "$UA"
    done
  i=$((i+1))
done

# Merge into one book
qpdf --empty --pages book_pdfs/ch_*.pdf -- Physics_I.pdf

# Verify
pdfinfo Physics_I.pdf | grep Pages
```

### 5c. Organize into Books/ folder

Place each PDF in the correct location based on `full_book_list.md`:

```
Books/<Technology>/Semester <N>/<Subject>.pdf
```

The master list (`full_book_list.md`) has the exact mapping of 628 PDFs.

---

## Step 6: Extract video metadata

### 6a. Fetch all videos

Run the video fetching part of Step 4. This produces `all_videos.json` with 42,468 records.

### 6b. Match videos to subjects

The key challenge: video metadata has `bunny_collection.label` (a subject name string) but
no direct `subject_id`. You need fuzzy matching:

```python
from difflib import SequenceMatcher

def match_video_to_subject(collection_label, subjects_map):
    """Match a video's collection label to a known subject."""
    best_score = 0
    best_match = None
    label_lower = collection_label.lower().strip()
    
    for subject_id, subject_name in subjects_map.items():
        score = SequenceMatcher(None, label_lower, subject_name.lower()).ratio()
        if score > best_score:
            best_score = score
            best_match = (subject_id, subject_name)
    
    return best_match if best_score > 0.5 else None
```

This gets ~88% of videos matched. The remaining 12% have labels that don't match any
known subject (e.g., "HSC ICT", "Trigonometry", "Algebra" — generic names without
enough context).

---

## Step 7: Build the folder structures

### 7a. Build Books/ structure

```bash
# Copy PDFs to the correct locations based on full_book_list.md
# The structure is: Books/<Technology>/Semester <N>/<Subject>.pdf
```

### 7b. Build Videos/ structure

```bash
cd ~/softmax_dbg/FreeMax
python3 ~/softmax_dbg/scripts/build_videos_md.py
```

This creates `Videos/<Department>/<Course>/<Subject>.md` with video tables.

### 7c. Verify everything

```bash
# Count books
find Books/ -name "*.pdf" | wc -l    # should be 628

# Count video files
find Videos/ -name "*.md" | wc -l    # should be ~165

# Check for broken symlinks
find . -type l ! -exec test -e {} \; -print   # should be empty

# Check for empty PDFs
find Books/ -name "*.pdf" -empty               # should be empty
```

---

## Step 8: Push to GitHub

```bash
cd ~/softmax_dbg/FreeMax

# Stage everything
git add Books/ Videos/ Extras/ docs/ full_book_list.md README.md

# Check what's staged
git status

# Commit
git commit -m "Add 628 pass books + 42K video links organized like Softmax app"

# Push
git push origin main
```

---

## Troubleshooting

### "403 Forbidden" on API requests

You're missing the `User-Agent: Dart/3.2 (dart:io)` header. Cloudflare blocks requests
without a Dart user agent.

### "401 Unauthorized" on API requests

Your token expired. Re-run Step 3 to get a new OTP and token.

### "OTP limit exceeded"

You've requested too many OTPs (limit: 10 per 2 days per phone). Wait or use a
different phone number.

### Videos not matching to subjects

Some video collection labels are too generic ("Mathematics 01", "Algebra") and fuzzy
matching can't determine which department/semester they belong to. This is a known
limitation — the API doesn't provide direct subject→video mappings.

### S3 "NoSuchKey" errors when downloading chapters

Some chapter PDFs have been deleted from the server. The app itself can't load them
either. Nothing you can do — skip those chapters.

### Large repo size (3.7 GB)

The 628 PDFs are stored as independent copies (no symlinks) because GitHub doesn't
preserve symlinks well. If size is a concern, you could use Git LFS for the PDFs.

---

## Full API reference

### Headers required on every request

```http
User-Agent: Dart/3.2 (dart:io)
X-APP-KEY: l0dtpwvzzmM
Authorization: Bearer <token>
Accept: application/json
```

### Endpoints

| Endpoint | Returns |
|---|---|
| `user/request/otp/` | Sends OTP SMS |
| `user/verify/otp/` | Returns access + refresh tokens |
| `app/web/departments/` | All 49 departments |
| `app/web/courses/` | All 265 courses |
| `app/web/courses/<id>/` | Course details + subjects array |
| `app/web/subjects/` | All 849 subjects (paginated, 200/page) |
| `app/web/chapters/?subject=<id>` | Chapters for a subject |
| `app/web/videos/` | All 42,468 videos (paginated, 200/page) |
| `ebooks/subjects/?course=<id>` | Books available for a course |
| `ebooks/subject-details/<id>/` | Book chapters + S3 paths |

### Pagination

Most list endpoints support `?limit=N&offset=N` pagination. Default page size varies
(10-200). Always paginate to get all results:

```python
offset = 0
while True:
    data = requests.get(f"{BASE}/endpoint/?limit=200&offset={offset}", headers=auth)
    results = data.json()['results']
    if not results: break
    all_items.extend(results)
    offset += 200
```

### Rate limits

- OTP: 10 requests per 2 days per phone
- API: No documented limit, but be respectful (0.3s delay between requests is safe)
- Video fetching: ~42,468 records at 200/page = ~213 requests, takes ~2 minutes
