# Setup — five minutes

You need two things working: **Claude Code** and this **skills pack**.

## 1 · Install Claude Code

Follow the official install for your OS: https://claude.com/claude-code

Then open a terminal and log in:

```sh
claude
# first run walks you through login — a Claude account (Pro/Max) or an API key both work
```

## 2 · Install the skills pack

From a terminal **inside the project you want the skills in** — the installer scopes skills to the project it's run from (add `-g` if you'd rather install them user-global):

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
