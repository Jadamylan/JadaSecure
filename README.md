# JadaSecure

### My security baseline for AI-assisted app building.

I move fast when I build — especially during hackathons, MVP sprints, and product experiments — but I do not want **“move fast”** to quietly turn into **“ship something careless.”**

**JadaSecure** is the reusable security standard I bring into projects I build with AI coding tools.

The source of truth is **[`Jadasecure.md`](./Jadasecure.md)**.

It gives the coding agent clear rules before the build gets too far, then gives me a human checklist before I call the project ready.

---

## Why I made it

AI coding tools make it ridiculously easy to go from:

```text
idea → working prototype → deployed app
```

That speed is useful. It can also hide bad assumptions.

Something compiling does **not** mean authentication is safe.  
Something working locally does **not** mean secrets are protected.  
Something looking polished does **not** mean the data flow makes sense.

I wanted one security floor I could reuse across my own builds instead of remembering the same launch questions from scratch every time.

---

## What it covers

JadaSecure is organized around the mistakes I most want to catch before launch:

| Area | What I am checking |
|---|---|
| **Secrets** | API keys, tokens, environment variables, accidental client-side exposure |
| **Authentication** | identity checks, session handling, protected routes |
| **Authorization** | whether users can reach data or actions they should not |
| **Input + APIs** | validation, unsafe assumptions, rate limits, error handling |
| **Data + privacy** | collection, storage, logs, sensitive fields, unnecessary retention |
| **Dependencies** | risky packages, outdated assumptions, unnecessary attack surface |
| **AI-assisted code** | places where the agent made a decision that still needs human judgment |
| **Launch blockers** | manual checks I want completed before real users touch the product |

The full rules, prompts, and launch checks live in **[`Jadasecure.md`](./Jadasecure.md)**.

---

## Drop it into a project

```bash
curl -fsSL https://raw.githubusercontent.com/Jadamylan/jadasecure/main/Jadasecure.md -o Jadasecure.md
```

Then tell the coding agent:

> Follow Jadasecure.md. Do not ship a change that violates Hard Rules.

Before launch, I run the security-audit prompt from the bottom of the file and complete the manual ship blockers myself.

---

## How I use it in AI-assisted development

My preferred flow is:

```text
1. Add JadaSecure at the beginning
2. Build the feature
3. Ask the agent to audit against JadaSecure
4. Fix anything that violates a Hard Rule
5. Run the human launch checks
6. Then ship
```

That matters to me because I use AI heavily when I build. The answer is not to pretend the agent is either perfectly safe or completely unusable.

The answer is to make the expectations explicit.

---

## What JadaSecure is not

This is **not** a certification, penetration test, compliance framework, or promise that a checklist can make an application “secure.”

It is a floor.

The goal is to:

- catch avoidable mistakes consistently
- make security expectations visible to the coding agent
- keep secrets and permissions from becoming afterthoughts
- force a human decision on the things I do not want an agent quietly deciding for me

---

## Credits

JadaSecure draws from checks I wanted to reuse from:

- [`benavlabs/vibe-check`](https://github.com/benavlabs/vibe-check)
- [`astoj/vibe-security`](https://github.com/astoj/vibe-security)

The original projects remain copyright their respective authors.

## License

MIT for my original wrapper / organization. See the upstream projects for their respective licensing and copyright terms.
