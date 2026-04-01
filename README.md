# resume-analyzer

A fully client-side resume analyzer. Paste a job description and your resume, and it extracts TF-IDF weighted keywords, scores your match percentage, and highlights exactly which terms you are missing and which you already have. No backend, no API key, no data leaves your browser.

---

## Overview

TF-IDF (term frequency-inverse document frequency) is used to identify which terms in a job description are actually distinctive versus just common filler. Your resume is then scored against those weighted terms. The result is a match percentage and two annotated keyword lists — matched terms highlighted in the resume, missing terms flagged for you to address.

---

## Project Structure
```
resume-analyzer/
├── src/
│   ├── components/
│   │   ├── InputPanel.tsx         # job description + resume paste inputs
│   │   ├── ScoreCard.tsx          # match % display
│   │   ├── KeywordList.tsx        # matched and missing term lists
│   │   └── HighlightedResume.tsx  # resume text with matched terms marked
│   ├── lib/
│   │   ├── tfidf.ts               # TF-IDF vectorizer
│   │   ├── tokenizer.ts           # lowercase, stem, remove stopwords
│   │   ├── scorer.ts              # cosine similarity + match % logic
│   │   └── highlighter.ts        # wraps matched terms in highlight spans
│   ├── data/
│   │   └── stopwords.ts           # English stopword list
│   ├── App.tsx
│   └── main.tsx
├── public/
│   └── index.html
├── tests/
│   └── lib/
├── package.json
└── vite.config.ts
```

---

## Prerequisites

- Node 18+
- npm or pnpm

---

## Quickstart
```bash
# 1. Clone and install
git clone https://github.com/you/resume-analyzer && cd resume-analyzer
npm install

# 2. Start dev server
npm run dev

# 3. Open in browser
http://localhost:5173
```

---

## How It Works

### 1. Tokenization

Raw text is lowercased, stripped of punctuation, split on whitespace, filtered against a stopword list, and lightly stemmed:
```ts
function tokenize(text: string): string[] {
  return text
    .toLowerCase()
    .replace(/[^a-z0-9\s]/g, "")
    .split(/\s+/)
    .filter(token => !STOPWORDS.has(token))
    .map(stem)
}
```

### 2. TF-IDF Weighting

The job description is treated as the target document. TF-IDF is computed treating the job description and resume as a two-document corpus, so terms that appear heavily in the job description but not in generic text get boosted:
```ts
function tfidf(term: string, doc: string[], corpus: string[][]): number {
  const tf = doc.filter(t => t === term).length / doc.length
  const df = corpus.filter(d => d.includes(term)).length
  const idf = Math.log((corpus.length + 1) / (df + 1)) + 1
  return tf * idf
}
```

### 3. Scoring

Cosine similarity is computed between the TF-IDF vector of the job description and the TF-IDF vector of the resume. The result is normalized to a 0–100% match score:
```ts
function cosineSimilarity(a: number[], b: number[]): number {
  const dot = a.reduce((sum, val, i) => sum + val * b[i], 0)
  const magA = Math.sqrt(a.reduce((sum, val) => sum + val ** 2, 0))
  const magB = Math.sqrt(b.reduce((sum, val) => sum + val ** 2, 0))
  return dot / (magA * magB)
}
```

### 4. Highlighting

Matched terms are wrapped in `<mark>` spans directly in the rendered resume text. Missing terms are listed separately with their TF-IDF weight shown so you know which gaps matter most.

---

## Output

| Output | Description |
|---|---|
| Match % | cosine similarity score between job description and resume vectors |
| Matched terms | keywords present in both documents, sorted by TF-IDF weight |
| Missing terms | high-weight job description keywords absent from your resume |
| Highlighted resume | full resume text with matched terms marked inline |

---

## Building for Production
```bash
npm run build
# output in /dist — deploy to any static host (Vercel, Netlify, GitHub Pages)
```

No environment variables needed. No server required.

---

## Testing
```bash
npm run test
```

Unit tests cover the tokenizer, TF-IDF weights, cosine similarity, and the highlighter against fixture resume and job description strings. Tests run in jsdom with no browser required.

---

## Notes

- The stemmer is a lightweight Porter stemmer. It handles common suffixes but is not a full morphological analyzer — "managing" and "management" will score as separate terms unless you extend the stemmer or add a synonym map.
- TF-IDF across a two-document corpus is a simplification. For more accurate weighting, seed the IDF calculation with a larger background corpus of job descriptions.
- All text stays in memory and is never persisted. Closing the tab clears everything.

---

## Contributing

PRs welcome for multi-document IDF seeding, PDF resume parsing via pdf.js, or an export-to-PDF report feature. Run `npm run lint` and `npm run test` before opening a PR.
