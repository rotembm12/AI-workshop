# Workshop setup — five minutes, before you arrive

You need two things working: **Claude Code** and this **skills pack**. Do this before the
workshop day — the afternoon hands-on assumes it's done.

## 1 · Install Claude Code

Follow the official install for your OS: https://claude.com/claude-code

Then open a terminal and log in:

```sh
claude
# first run walks you through login — a Claude account (Pro/Max) or an API key both work
```

If your team was given a shared key for the event, the organizers will send it separately.

## 2 · Install the skills pack

From any terminal:

```sh
npx skills@latest add rotembm12/AI-workshop
```

Pick the skills you want when prompted (taking all of them is fine). There is no setup step —
every skill works with plain markdown files in your repo. No issue tracker, no auth, nothing
to configure.

## 3 · Smoke test — thirty seconds

```sh
cd <your project>
claude
```

Type `/grilling` — if it autocompletes, you're done. Ask it anything or `/exit`.

## If something's broken

Don't burn your evening on it. Come as you are — you'll pair with a neighbor for the
hands-on (one working setup per pair is all it takes), and I'll be around at the breaks.
