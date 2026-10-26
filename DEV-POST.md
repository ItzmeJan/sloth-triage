---
title: "I let Gemma pick my top 3 tasks, and the rest of the list can wait"
published: false
tags: hf26challenge, ai, javascript, opensource
---

My to-do list is not a plan. It is a museum of things I felt guilty about once.

So for Hacktoberfest Week 3 ("Slow and Steady"), I built the smallest thing that fixes
that: **sloth-triage**, a ~70-line Bun CLI that hands your task list to an open-weight
model and asks the only question that matters — *what are three things actually worth
doing today?* Everything else is explicitly allowed to wait.

```text
$ bun run index.js tasks.txt

🦥 sloth-triage (gemma4:cloud) read 9 tasks and chose 3

1. Finish the quarterly report — due tomorrow at 9am.
2. Renew the domain — it expires Friday.
3. Reply to the landlord — a leaking tap gets worse.

Skip the conference recording and the photo library. The downloads folder can wait forever.
```

That is the whole product. No accounts, no dashboard, no streaks, no "productivity
system". It reads a text file, makes one request to a model on my machine, and prints
a short plan. Then it stops.

## The build

There is nothing to install:

```json
{
  "name": "sloth-triage",
  "type": "module",
  "scripts": { "start": "bun run index.js tasks.txt" }
}
```

The core is one prompt and one `fetch` to an OpenAI-compatible endpoint:

```js
const prompt = `You are a very lazy but sensible assistant. Below is the user's task list.

Pick the ${top} tasks that genuinely deserve attention today. Prefer deadlines and things
that get worse if ignored. Then write a short "lazy plan": order the tasks, one line each,
and say what can be deliberately skipped or deferred. Be calm and brief.

Tasks:
${tasks.map((task) => `- ${task}`).join("\n")}`;

const res = await fetch(`${baseUrl}/chat/completions`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    model,
    messages: [{ role: "user", content: prompt }],
    temperature: 0.3,
  }),
});
```

`SLOTH_MODEL` and `SLOTH_BASE_URL` are the only configuration. The default is Ollama on
`127.0.0.1:11434`, running Gemma. Lines can be plain text or `- [ ]` checkboxes; `--top=N`
changes how many survivors you get.

## Why open matters here

A task list is one of the most personal files on a laptop. It contains health
appointments, money problems, names of people I owe replies to. Sending that to a closed
API so it can rank nine bullets is a bad trade.

With an open-weight model served by Ollama, the list stays on the machine:

- **No account and no API key.** The tool works the same on a plane.
- **No meter running.** A 1B model answering one short prompt costs effectively nothing.
- **The model is a setting, not a vendor.** Swap Gemma for Llama or Qwen with one env var,
  and the prompt is right there in the file if you want to change how it thinks.
- **The whole thing is auditable.** Two files and no dependencies. You can read every line
  before it reads your diary.

That last point is the real one. "Open source AI" is not just a license on weights. It is
being able to see the prompt, see the request, and know there is no hidden second call.
A closed product cannot offer that, no matter how good its summary is.

## What I would fix next

- Small models sometimes confuse "urgent" with "loud". The prompt asks for reasons so the
  bad calls are at least visible.
- It does not know my calendar. A real version would pass in the next two days of events.
- I tested with Gemma through Ollama; the code expects any local OpenAI-compatible server.

The project is deliberately boring: one file, one prompt, one output. That is the point.
A tool that helps you do less should not itself become another thing to maintain.

**Repo:** https://github.com/ItzmeJan/sloth-triage

Built for the [Hacktoberfest Open-Source AI Challenge: Week 3](https://dev.to/challenges/hacktoberfest-week3-2026-10-19).
