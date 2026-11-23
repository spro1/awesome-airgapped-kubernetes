# Contribution Guidelines

Thanks for helping make this list better. The value of this list is the *Air-gap* note on every entry, so contributions are judged mostly on that note.

## Inclusion criteria

An entry must:

- Be useful for installing or operating Kubernetes **without internet access**.
- Be actively maintained: not archived, and with commits or releases within the last 12 months. If a project is abandoned but a maintained fork exists, add the fork.
- Have an *Air-gap* note that is backed by the project's **official documentation or repository**. Put the evidence link in your pull request description.

We do not accept:

- Entries whose air-gap behavior you have not checked yourself. A wrong air-gap note is worse than no entry.
- Marketing pages, paywalled content, or closed-source products without public documentation.
- Content that refers to a specific company, customer, or internal system. Gotchas must be general technical facts.

## Entry format

```md
- [Name](https://link) - What it does, in one sentence. *Air-gap:* One sentence on how it behaves offline.
```

The *Air-gap* sentence should state whichever of these apply:

- Whether offline/air-gapped installation is officially supported, and how (bundle, image tarball, private registry).
- Whether there is a way to list or export the images and binaries it needs.
- Whether the default configuration makes outbound calls (telemetry, update checks, public registries) and how to turn them off.
- Whether it depends on data that must be refreshed periodically (for example, a vulnerability database) and how to do that offline.

Style:

- Link to the project's main repository or official documentation, not to a fork or a mirror.
- The description starts with a capital letter and ends with a period.
- Keep both sentences short and factual. No "awesome", "best", or "easy".
- Add the entry at the end of the relevant section.

## Translations

`README.ko.md` mirrors `README.md`. If you change an entry, update both files, or say in your pull request that the Korean translation still needs updating.

## Pull requests

- One entry or one fix per pull request.
- Run `npx awesome-lint` locally before opening the pull request.
- Link checks run in CI with [lychee](https://github.com/lycheeverse/lychee).
