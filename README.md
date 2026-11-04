# Builder-Card

Enter a GitHub username and get a developer profile card built from that person's real public GitHub data.

Built with React, Vite, JavaScript and plain CSS. There is no backend, database or API key.

## Setup

Requires Node.js 20.19 or newer (on the 22 line, 22.12 or newer). Check with `node -v`.

```bash
npm install
npm run dev
```

Then open the local URL Vite prints (usually http://localhost:5173).

Other scripts:

- `npm run build` creates a production build in `dist/`
- `npm run preview` serves that build locally

## What the card shows

- Avatar, name, username, bio
- Followers, following and public repository count
- The three most-starred repositories
- The most common languages across their repositories
- A few recent public events (pushes, created repositories, forks, stars, pull requests, issues)

## API endpoints used

All requests are plain `fetch()` calls to the GitHub REST API, with no authentication.

| Purpose | Endpoint |
| --- | --- |
| Profile | `GET https://api.github.com/users/{username}` |
| Repositories | `GET https://api.github.com/users/{username}/repos?per_page=100&page={n}` (one request per 100 repositories, up to 5) |
| Recent activity | `GET https://api.github.com/users/{username}/events/public?per_page=30` |

## Project structure

```
src/
  main.jsx                  Mounts the app
  App.jsx                   State (status, error, profile) and the search handler
  api/github.js             fetch() calls, error handling, username validation
  utils/summarize.js        Turns raw API data into top repos, languages, events
  components/
    SearchForm.jsx          Input and Search button
    ProfileCard.jsx         Header, stats, and the three sections below
    TopRepos.jsx            Three most-starred repositories
    Languages.jsx           Language chips with percentages
    RecentActivity.jsx      Recent events list
  styles.css                All styling
```

## How it works

1. `SearchForm` collects the username and calls `onSearch` when the form is submitted. The page never reloads, so you can search again right away.
2. `App.handleSearch` sets the status to `loading`, then calls `fetchProfile` in `api/github.js`.
3. `fetchProfile` validates the username, fetches the user, then fetches the repository pages and the events together with `Promise.all`. The number of repository pages comes from `public_repos` on the profile.
4. The raw data goes through the helpers in `utils/summarize.js`: `getTopRepos`, `getLanguages` and `getRecentEvents`.
5. `App` stores the result and sets the status to `success`, and `ProfileCard` renders it. If anything throws, the status becomes `error` and the message is shown instead.

The app is always in exactly one of four states: `idle`, `loading`, `error` or `success`. The page shows a different piece of UI for each.

## Error handling

| Situation | What the user sees |
| --- | --- |
| Empty input | "Enter a GitHub username." |
| Malformed username | "That doesn't look like a valid GitHub username." |
| User doesn't exist (404) | "No GitHub user found with that username." |
| Rate limit (403 / 429) | "GitHub rate limit reached. Please try again in a while." |
| Network failure | "Could not reach GitHub. Check your internet connection." |
| Any other API error | "GitHub returned an error (status). Please try again." |
| No bio, repos, languages or activity | A short placeholder in that section |

## Known limitations

- **Rate limit:** unauthenticated requests are limited to 60 per hour per IP address. A typical search (up to 100 repositories) makes 3 requests, so that is about 20 searches per hour. A user with no repositories costs 2 requests; a user with 500 or more costs 7.
- **Repositories:** GitHub returns 100 per request in alphabetical order, so the app reads one page per 100 repositories, up to 5 pages (500 repositories). Accounts with more than 500 repositories are ranked from the first 500 alphabetically.
- **Forks are excluded** from top repositories and languages, because they are other people's code. The repository count in the stats still includes them, since that number comes straight from the profile.
- **Languages** are counted per repository using the repository's main language (GitHub's `language` field). This avoids one extra request per repo, but it is a count of repos, not lines of code. Only the top five are shown, and a language under 0.5% of repositories is shown as `<1%`.
- **Activity:** GitHub's public events feed only goes back about 90 days and can lag behind by a few minutes. Only the 30 newest events are read, and event types the app doesn't describe (comments, reviews, deletions, releases) are skipped, so a user whose recent activity is mostly comments can show fewer than five items or none.
