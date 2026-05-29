# RASS — Topic hierarchy evaluation tool

Human quality assessment for RASS-generated topic hierarchies.  
Raters evaluate **coverage**, **granularity**, and **interpretability** on a 1–7 Likert scale.

---

## Repository structure

```
rass-eval/
├── index.html                              ← single-page rating tool
├── hierarchies/
│   ├── index.json                          ← master map: rater_id → [slug, slug, ...]
│   ├── template.json                       ← copy this to create a new hierarchy
│   └── rater_01/
│       └── microplastics/
│           └── hierarchy.json
```

---

## Adding a new rater

1. Add the rater and their topic slugs to `hierarchies/index.json`:
   ```json
   {
     "rater_01": ["microplastics"],
     "rater_02": ["climate-change", "biodiversity"]
   }
   ```
2. Create a folder for each slug: `hierarchies/{rater_id}/{slug}/`
3. Copy `template.json` into the folder as `hierarchy.json` and populate with RASS output.
4. Push to GitHub — the rater can access their session immediately.

The tool loads hierarchies **only for the rater ID entered at login**. No rater sees another rater's topics.

---

## Hierarchy JSON format

Flat dict keyed by topic ID. Tree encoded via `parentId` / `childrenIds`.

```json
{
  "meta": {
    "topic": "Microplastics in marine environments",
    "rater_id": "rater_01",
    "n_topics": 11,
    "n_root": 3,
    "generated_at": "2025-05-29T00:00:00Z",
    "corpus_size": 312
  },
  "topics": {
    "t1": {
      "id": "t1", "level": 1, "order": 1,
      "name": "Root topic name",
      "description": "Full description.",
      "keywords": ["kw1", "kw2"],
      "confidence": 0.91,
      "parentId": null,
      "childrenIds": ["t1.1", "t1.2"],
      "tags": ["preliminary"]
    },
    "t1.1": {
      "id": "t1.1", "level": 2, "order": 1,
      "name": "Child topic name",
      "description": "Full description.",
      "keywords": ["kw1", "kw2"],
      "confidence": 0.88,
      "parentId": "t1",
      "childrenIds": [],
      "tags": ["preliminary"]
    }
  }
}
```

---

## GitHub Pages deployment

1. Create repo `rass-eval` under `jrcf7`
2. Push all contents
3. **Settings → Pages → Source: Deploy from branch → main → / (root)**
4. Live at `https://jrcf7.github.io/rass-eval/`

Share this single URL with all raters. Each rater logs in with their own ID and sees only their hierarchies.

---

## Collecting results

Raters download a `.json` file at session end. Combine them:

```python
import json, glob, pandas as pd

records = []
for f in glob.glob("responses/*.json"):
    d = json.load(open(f))
    for r in d["ratings"]:
        records.append({
            "rater_id":         d["rater_id"],
            "domain":           d["domain"],
            "slug":             r["slug"],
            "topic":            r["topic"],
            "n_topics":         r["n_topics"],
            "coverage":         r["coverage"],
            "granularity":      r["granularity"],
            "interpretability": r["interpretability"],
            "notes":            r["notes"],
        })

df = pd.DataFrame(records)
df.to_csv("ratings.csv", index=False)
```
