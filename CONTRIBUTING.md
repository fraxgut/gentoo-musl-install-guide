<!--
CONTRIBUTING.md
@fraxgut
CC-BY-SA-4.0
Contribution rules, commit conventions and translation synchronisation
-->

# Contributing

Thank you for your interest in this guide. This document gives the conventions
that the repository follows.

English is the infrastructure language of the repository. Directory names, file
names, commit messages and this document are in English. Inside `i18n/`, each
language is equal.

## What makes a good contribution

This guide documents commands that erase disks and rebuild systems. An error in
the text can destroy the data of a reader. Therefore:

- Test a command before you propose it. Say which system you tested it on.
- Give the reason for a change, not only the change. The guide explains why
  each decision is correct, and a new step needs the same treatment.
- Cite the source for a technical claim. Prefer the Gentoo wiki, the kernel
  documentation, the Btrfs documentation and the manual pages.
- Report a fact that you did not test. A note that says "not tested on BIOS" is
  useful. A silent guess is not.

## Commit messages

The repository uses [Conventional Commits
1.0.0](https://www.conventionalcommits.org/en/v1.0.0/):

```
<type>(<scope>): <description>
```

Write the description in the imperative. Start it with a capital letter. Do not
put a full stop at the end. Keep the subject line at about 50 characters, and
at 72 characters as the limit.

### Types

| Type       | Use                                                        |
|------------|------------------------------------------------------------|
| `feat`     | A new section, a new document or a new procedure.          |
| `fix`      | A correction to a command, a package name or a parameter.  |
| `docs`     | A change to the text that does not change a procedure.     |
| `refactor` | A reorganisation that keeps the same content.              |
| `chore`    | Repository maintenance: assets, licence, structure.        |
| `revert`   | A reversal of an earlier commit.                           |

### Scopes

Use the scope that names the part that you changed:

```
docs(es): Correct the storage instructions
docs(en): Make the LUKS setup clearer
fix(storage): Correct the Btrfs swapfile configuration
fix(toolchain): Update the LLVM package atoms
refactor(i18n): Reorganise the translated documentation
chore(licence): Adopt CC BY-SA 4.0
```

The scopes `en` and `es` mark a change to one language alone. The scopes
`storage`, `toolchain`, `install` and `troubleshooting` mark a change to that
subject in each language.

### The body

Add a body when the difference does not show the intention. Separate it from
the subject with one empty line. Wrap the text at about 72 characters. Write
what changed and why it was necessary.

### Breaking changes

Put `!` before the colon, or add a `BREAKING CHANGE:` footer. A breaking change
in this repository is a change that makes an existing installation invalid.

## Translations

Each language has the same file tree under `i18n/`. The file names stay in
English in every language. Only the content is translated. You can thus compare
the two directories file by file.

A change to the content of a document should also change the other language:

```
i18n/en/storage.md    →    i18n/es/storage.md
```

If you cannot translate your change, say so in the pull request. A maintainer
then does the translation. An untranslated change is better than no change.

Use `docs(en)` or `docs(es)` when a commit touches one language alone.

## Language rules

English text follows ASD-STE100 Simplified Technical English: the active voice,
the simple tenses, one instruction in one sentence, and short sentences.

Spanish text follows a formal register. It uses complete words and the correct
accents.

Both languages keep the same technical content. A translation that removes a
warning or a condition is incorrect.

## How to propose a change

1. Fork the repository.
2. Make a branch: `git checkout -b fix/btrfs-mount-options`
3. Make your change and commit it with a conventional message.
4. Push the branch: `git push origin fix/btrfs-mount-options`
5. Open a pull request. Say what you tested and on which system.

For a question or an idea, open an issue first. A discussion before the work
saves time for everybody.

## Licence

Your contributions are licensed under CC BY-SA 4.0, the licence of this
documentation. See [LICENCE.md](LICENCE.md).
