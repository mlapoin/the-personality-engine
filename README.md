# Personality Engine

A set of quick, shareable personality quizzes. Type in a username, get a result. The same name always gets the same answer, so results can be shared and checked by anyone.

It's a fully static site: plain HTML, CSS and JavaScript with no backend and no database. That's what let it handle more than 1.5 million visits in one month with zero downtime and zero server costs.

## Where to look

If you only have a few minutes, these are the interesting parts:

- **`toxicity.html` → `analyze()`**: the core idea. A username is hashed into a stable number, which becomes the score. The meter fills and shakes harder the higher the score goes.
- **`compatibility.html` → `analyze()`**: two names are sorted before hashing, so "A + B" and "B + A" always give the same result.
- **`toxictrait.html` → `render()` / `renderCard()`**: results are written as templates (`{u}` for the name, `{r:...}` for the highlighted phrase), so one string drives both the page and the share card.
- **Any quiz → `saveCard()`**: builds a hidden share card and turns it into a PNG with html2canvas, so people can post an image instead of a link.

## Quizzes

| Page | What it does |
| --- | --- |
| `toxicity.html` | Red Flag Rating: a 1–99% score on an animated meter |
| `toxictrait.html` | Toxic Trait: one result picked from a list |
| `compatibility.html` | Compatibility between two usernames |
| `fictrope.html` | Which story trope two usernames would be |

## How it works

```js
function hash(str) {
  let h = 0;
  for (let i = 0; i < str.length; i++) {
    h = Math.imul(31, h) + str.charCodeAt(i) | 0;
  }
  return Math.abs(h);
}
```

Each quiz cleans the input (trims it, drops a leading `@`, lowercases it), hashes it, and uses the number to pick a result or a percentage. Some quizzes add a short salt (for example `hash(u + 'tox')`) so the same name doesn't land on matching results across quizzes.

There is no server call anywhere. Share links and the site address on the share card are built from `location` at runtime, so the code works on any host without edits.

## Stack

- HTML + CSS, JavaScript (no framework, no build step)
- [html2canvas](https://html2canvas.hertzen.com/) for "Save as Image"
- GitHub Pages for hosting
- Google Analytics (optional)

## Run it

Open `index.html` in a browser, or serve the folder:

```bash
npx serve .
```

To deploy, push the folder to any static host (GitHub Pages, Netlify, Vercel).

## Setup notes

- **Analytics:** each page has a Google Analytics snippet with the placeholder `G-XXXXXXXXXX`. Swap in your own ID or delete the block.
- **Link previews:** `og:image` points at `preview.jpg`. Some sites only read absolute URLs, so change it to the full address once the site is live.

## Structure

```
index.html            quiz menu
toxicity.html         Red Flag Rating
toxictrait.html       Toxic Trait
compatibility.html    Compatibility
fictrope.html         Story trope
404.html              not-found page
preview.jpg           link-preview image
```

## What I'd change

I'd pull the shared code into one file. Each quiz has its own copy of the hash function, styles and share-card logic, so a small fix meant editing every page. A single `quiz.js` and `quiz.css`, with each page holding only its title and result list, would have made it much faster to add new quizzes.
