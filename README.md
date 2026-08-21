# JadaSecure

Personal security standard for every app I build.

The file you copy into a project is **[Jadasecure.md](./Jadasecure.md)**. That is the source of truth: ship blockers, hard rules, launch checklist, manual tests, and the Cursor audit prompt.

It combines the strongest attack checks from [vibe-check](https://github.com/benavlabs/vibe-check) with the strongest ops checks from [vibe-security](https://github.com/astoj/vibe-security).

## Add it to a new app

```bash
curl -fsSL https://raw.githubusercontent.com/Jadamylan/jadasecure/main/Jadasecure.md -o Jadasecure.md
```

Tell Cursor:

> Follow Jadasecure.md. Do not ship a change that violates Hard Rules.

Before real users, paste the **Agent prompt** from the bottom of `Jadasecure.md`, then run the five **Ship blockers** yourself.

## License

MIT. Use it in any of my repos. The original checklists remain copyright their authors.
