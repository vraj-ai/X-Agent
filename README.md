# X-Agent — vraj ai digest

**A personal AI-news reading pipeline that drafts posts in your own voice, and
never posts without you asking.**

It reads AI sources, rewrites each item into plain technical language, and
builds a local copy-and-paste page with drafted originals, opinions, replies,
repost comments, and image ideas — written from a `profile.json` that describes
your voice, themes, and opinions. Posting to X is a separate, explicit,
one-item-at-a-time command.

Zero dependencies. Python 3 standard library only.

## Design rules

These are the constraints the whole thing is built around:

- **Copy-first, not auto-post.** Posting is off by default and requires an
  explicit `--post`. It never posts a batch.
- **Official API, no scraping.** X data comes from the X API.
- **No invented facts.** It won't fabricate source stats or engagement numbers.
- **Degrades instead of failing.** Missing keys turn features off, they don't
  break the run.

## What one run does

1. Reads AlphaSignal's RSS feed
2. Reads recent AI posts from X — *if* `X_BEARER_TOKEN` (or a user token) is set
3. Rewrites each item in plain technical language — *if* `OPENAI_API_KEY` is set
4. Writes an RSS feed to `out/ai-digest.xml`
5. Writes a copy-and-paste dashboard to `out/index.html`
6. Drafts originals, opinions, replies, repost comments, and image ideas from
   `profile.json`
7. Optionally posts **one** reviewed item, only with `--post`

## Run it

```sh
cp .env.example .env
# fill in the keys you actually have — none are required
python3 ai_digest.py
open out/index.html          # or: python3 -m http.server 8080 --directory out
```

With no keys at all it still works: AlphaSignal's public feed plus deterministic
personalized fallback packs. An LLM key improves wording and variety; an X token
adds X posts as a source.

## Configure it

| File | What you change |
|------|-----------------|
| `profile.json` | Your voice, themes, opinions, projects, and things to avoid — this is what makes the drafts sound like you |
| `config.json` | Feed URL, X search queries, item limits, output paths, model |
| `.env` | API keys and tokens |

**Tuning X topics** — edit `x_queries` in `config.json`. Keep queries focused;
one good query beats scraping the whole platform:

```json
"x_queries": [
  "(agents OR \"code agents\" OR MCP) -is:retweet lang:en",
  "(LLM OR reasoning OR evals OR RLHF) -is:retweet lang:en",
  "(inference OR GPUs OR RAG OR \"prompt injection\") -is:retweet lang:en"
]
```

## Connect X (one time)

Create an X developer app with this exact callback URL:

```text
http://127.0.0.1:8765/callback
```

Put the app's client ID in `.env`, then run:

```sh
python3 x_auth.py
```

It opens X in your browser, waits for approval, and writes the access and
refresh tokens into `.env` with mode `600`. It never prints the token.
Requested scopes: `tweet.read users.read tweet.write offline.access`.

Verify with one clearly labeled test message:

```sh
python3 ai_digest.py --test-post
```

Then, after reviewing `out/index.html`, post at most one new digest item:

```sh
python3 ai_digest.py --post
```

## Schedule it

Hourly via cron — use the absolute path to *this* repo:

```cron
0 * * * * cd /absolute/path/to/X-Agent && /usr/bin/python3 ai_digest.py >> out/job.log 2>&1
```

## Checks

```sh
python3 -m unittest -v
python3 ai_digest.py --self-test
```

## Project docs

Decision history, issues, and handoffs live in [`CONTEXT/`](CONTEXT/) — start
with [`CONTEXT/index.md`](CONTEXT/index.md).
