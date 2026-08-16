# 🧑‍💻 How to do it yourself

A copy-paste guide to pull these pass books straight from the **Softmax Learning** API, merge
them into complete PDFs, and verify them — without relying on this repo.

> **Before you start:** this requires a valid account (phone number that can receive an OTP).
> The app's data is provided under the app's own terms. Use it only for content you're allowed
> to access (e.g. your own purchased/registered account) and for personal study.

---

## 0. Prerequisites

```bash
# Linux with common tools
sudo apt install jq curl qpdf   # or: sudo pacman -S jq curl qpdf
```

Decompile the APK if you want the full endpoint table:

```bash
# get the APK from a device/APK mirror, then:
apktool d base.apk
cat base_dec/assets/flutter_assets/.env   # the endpoint list
```

## 1. Get a token

```bash
BASE=https://softmaxmanager.xyz/api/v1
APPKEY=Z6AeXIbpuVw
PHONE=01XXXXXXXXX            # your phone number

# request OTP (sends SMS)
curl -s "$BASE/user/request/otp/" \
  -H "X-APP-KEY: $APPKEY" \
  -H "Content-Type: application/json" \
  -d "{\"phone_number\":\"$PHONE\"}"

# verify OTP (SMS code) → returns access_token
TOKEN=$(curl -s "$BASE/user/verify/otp/" \
  -H "X-APP-KEY: $APPKEY" \
  -H "Content-Type: application/json" \
  -d "{\"phone_number\":\"$PHONE\",\"otp\":\"$CODE\"}" | jq -r .access_token)

AUTH=(-H "X-APP-KEY: $APPKEY" -H "Authorization: Bearer $TOKEN")
```

## 2. List technologies (departments)

```bash
curl -s "$BASE/academic/department/" "${AUTH[@]}" | jq .
# → [ { "id": 3, "name": "Civil Technology", ... }, ... ]
```

## 3. Find a subject's books

```bash
# books available for a course (use your department's course id)
curl -s "$BASE/ebooks/subjects/?course=3" "${AUTH[@]}" | jq .
# → subjects with id + title, and whether the user has access
```

## 4. Get one book's chapters

```bash
SUBJECT_ID=42   # e.g. "Water Supply Engineering"

curl -s "$BASE/ebooks/subject-details/$SUBJECT_ID/" "${AUTH[@]}" | jq .
# → { "title": "...", "chapters": [ { "id": 2137, "pdf": "sos-prod/ebook/chapter/preview/..." } ] }
```

## 5. Download + merge chapters

```bash
mkdir -p book && cd book

# fetch the chapter list into a file
curl -s "$BASE/ebooks/subject-details/$SUBJECT_ID/" "${AUTH[@]}" \
  | jq -r '.chapters[].pdf' > chapters.txt

# download each chapter
i=1
while read -r path; do
  curl -s -o "ch_$(printf '%03d' $i).pdf" \
    "https://softmaxmanager.xyz/media/$path" "${AUTH[@]}"
  i=$((i+1))
done < chapters.txt

# merge in order
qpdf --empty --pages ch_*.pdf -- "Full_Book.pdf"
```

## 6. Verify

```bash
pdfinfo Full_Book.pdf | grep Pages   # sanity check page count
```

## 7. Reorganize like the app

The layout in this repo (`Technology → Semester → Subject.pdf`) is built from the app's own
course structure. To reproduce it:

```bash
# each subject belongs to a department (technology) and a semester.
# the full_book_list.md here is already in that shape:
#   - Subject — ✅ [PDF: filename.pdf]
```

## 8. Run the whole pipeline

Everything above is the manual version. A scripted version would:

1. `GET academic/department/` → map department ids to names
2. `GET ebooks/subjects/?course=<id>` → for each course, list subjects
3. `GET ebooks/subject-details/<id>/` → collect chapters per subject
4. download + `qpdf` merge per subject
5. `pdfinfo` verify
6. place under `Books/<Technology>/Semester_<N>/`

That's exactly how the 93 books in this repo were produced.

---

## Common issues

| Problem | Fix |
|---|---|
| `NoSuchKey` on a chapter | The chapter is missing on the server (happens for a few books). Skip it. |
| 401 / token expired | Refresh the token: `POST user/updated-request/otp/` with the phone number. |
| 403 on media | You don't own the subject. Only books on your enrolled/purchased course work. |
| Chapter ordering wrong | Some APIs return chapters unsorted — sort by the chapter serial from the JSON before merging. |
