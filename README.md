# JadaSecure

### My security baseline for AI-assisted app building.

I move fast when I build — especially during hackathons and product experiments — but I do not want “move fast” to become an excuse for shipping careless security decisions.

**JadaSecure** is the reusable security standard I drop into projects I build with AI coding tools.

The source of truth is **[`Jadasecure.md`](./Jadasecure.md)**. It contains:

- ship blockers
- hard security rules
- launch checks
- manual tests
- an agent / Cursor security-audit prompt

It combines useful attack-focused checks from [`vibe-check`](https://github.com/benavlabs/vibe-check) with operations-focused checks from [`vibe-security`](https://github.com/astoj/vibe-security), then packages the pieces I want to consistently apply across my own builds.

---

## Why I made it

AI coding tools make it very easy to go from idea → working prototype fast.

That is a feature. It can also create a bad habit: assuming something is safe because it works.

I wanted one file I could put in front of the agent at the beginning and again before launch so security expectations are part of the build, not an afterthought.

---

## Add it to a project

```bash
curl -fsSL https://raw.githubusercontent.com/Jadamylan/jadasecure/main/Jadasecure.md -o Jadasecure.md
```

Then tell your coding agent:

> Follow Jadasecure.md. Do not ship a change that violates Hard Rules.

Before putting a project in front of real users, run the agent audit prompt from the bottom of the file and complete the manual ship blockers yourself.

---

## The principle

JadaSecure is not a claim that a checklist can make an application “secure.”

It is a floor.

The goal is to catch avoidable mistakes consistently, make security expectations explicit to the coding agent, and force a human review of the things I do not want an agent quietly deciding for me.

---

## Credits

JadaSecure draws from the strongest checks I wanted to reuse from:

- [`benavlabs/vibe-check`](https://github.com/benavlabs/vibe-check)
- [`astoj/vibe-security`](https://github.com/astoj/vibe-security)

The original checklists remain copyright their respective authors.

## License

MIT for my original wrapper / organization. See upstream projects for their respective licensing and copyright terms.
