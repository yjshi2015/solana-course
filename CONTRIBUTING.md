# Contributing to the Course

Hey, thanks for contributing!

## How Can I Contribute?

Content presentation is controlled by the config file `course-structure.json`. All content is in the `content` directory sorted by the slug in `course-structure.json`. Static assets are in the `assets` directory.

### Adding Content

If you'd like to add content, please start by [creating an issue](https://github.com/Unboxed-Software/solana-course/issues/new) and tagging @jamesrp13 to discuss your reasoning, plan, and timeline.

Once a plan has been discussed and agreed to, you can start working on content. 

When you're done, create a PR to the `main` branch.

Create new modules in the same format as the existing modules - see [Getting Started](./content/getting-started.md).

This structure leans into a pedagogical technique call IWY loops. IWY stands for "I do, We do, You do." Each step along the way increases the audiences exposure to the topic _and_ reduces the amount of handholding you're given.

### Editing Existing Content

If you want to fix a typo or otherwise improve on existing content, follow a similar process as with adding content:

1. [Create an issue](https://github.com/Unboxed-Software/solana-course/issues/new) and/or comment on an existing issue to state you've started working
2. Create a PR to the `draft` branch during or when complete

### Guidelines

The guidelines below are consistent with Solana Foundation Style, in order to ensure consistency with other content on solana.com. There's also a few additional items aimed at technical documents. 

Use language consistent with [TERMINOLOGY](https://github.com/solana-foundation/developer-content/blob/main/docs/terminology.md). 

In particular:

- Use sentence case for headlines (”Solana Foundation announces new initiative” instead of “Solana Foundation Announces New Initiative”).
- Use 'secret key' rather than 'private key' to be consistent with web3.js. __Note__: this will change in a future version of web3.js. 
- Use 'wallet app' for software. 'wallet' for the address that holds value.
- Use 'onchain' (not on-chain, definitely not smart contract) when referring to onchain apps.
- Use 'SOL' rather than 'Sol' to refer to Solana's native token. Definitely don't call it Solana!
- PDAs are not public keys. It is not possible to have a public key without a secret key. A public key is derived from a secret key, and it is not possible to generate a public key without first generating a secret key.
- Use the terms 'blockchain' or 'web3' rather than 'crypto'.
- Be careful about the term 'token account'. A ['token account' is any account formatted to hold tokens](https://solana.stackexchange.com/questions/7507/what-is-the-difference-between-a-token-account-and-an-associated-token-account), and being specific (rather than, for example, swapping between 'associated token account' and 'token account' makes this clearer. 
  - Use the specific term 'associated token account' rather than just 'token account' if you're referring to an account at an associated token address.  
  - Use 'token mint account' to refer to the address a token is minted at. E.g., the [USDC mainnet token mint account](https://explorer.solana.com/address/EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v).
- Use apostrophe of possession, [including for inanimate objects](https://english.stackexchange.com/questions/1031/is-using-the-possessive-s-correct-in-the-cars-antenna). Eg 'the account's balance' is correct.
- Don't use 'here' links. They make the course hard to scan and ['here' links are bad for SEO](https://www.smashingmagazine.com/2012/06/links-should-never-say-click-here/).

Code examples should be formatted as follows:

### JS/TS

 - two spaces per prettier defaults, StandardJS, node style guide, idiomatic JS, AirBnB style guide, MDN, Google Style guide, codepen, jsfiddle, etc.

 - We are aiming to use `esrun`, which supports top level `await`, doesn't require a `tsconfig.json`, etc. There is no need for `async function main()` wrappers or [IIFEs](https://developer.mozilla.org/en-US/docs/Glossary/IIFE). `await` just works. If you see these wrappers, delete them and use `esrun`.

 - Likewise, use async/await and use try / catch everywhere, rather than `.then()` and `.catch()`
 
### Rust
 - four spaces per rustfmt

Diagrams:

- If you draw Solana elliptic curves, these are [Edwards curves](https://en.wikipedia.org/wiki/Edwards_curve)
- Use [Whimsical](https://whimsical.com/) for diagrams
- Use SVG where possible (they'll look better on different screens). You can get an SVG export from Whimsical by appending `/svg` to the end of a Whimsical URL.
 
Note that while `prettier` can format Markdown, [prettier doesn't support language-specific settings inside Markdown files](https://github.com/prettier/prettier/issues/5378) so you'll need to format the code yourself for now.

## Committing

We are using [conventional commits](https://www.conventionalcommits.org/en/v1.0.0/)
for this repository.

General flow for making a contribution:

1. Fork the repo on GitHub
2. Clone the project to your own machine
3. Make a new branch and add your contributions
4. Push your work back up to your fork on GitHub
5. Submit a Pull Request so that we can review your changes

## Config

Content is controlled by the config file `course-structure.json`. All content is in the `content` dir sorted by the slug in `course-structure.json`.

## Writing Content for this Guide

Use the terms at https://docs.solana.com/terminology

A **Track** is a provable skill. Right now the tracks are simply front end Solana development and on-chain Solana development.

A **Unit** is a group of lessons. 

Each **Lesson** is a block of added understanding, starting from scratch and building on the previous lesson to take students to mastery of a topic. 

Lessons should follow the format:

 - **Overview** section is a conceptual overview. The equivalent would be when a teacher in a classroom says "don't do this yet, just watch.". The overview is intentionally not meant to be something readers code along with.

 - **Lab** section is when students code along and should follow a step-by-step process.

## Static Assets

Static assets are in `assets`. This is set in `svelte.config.js` to match the older soldev-ui directory. 

To include an image:

```markdown
![Some alt text](../assets/somefile.svg)
```

## Components

Components are individual Svelte files, in the `/src/lib/components` directory.

Routes (pages) are in `/src/lib/routes`

## Development

```
npm run dev -- --open
```

### Developing

Once you've created a project and installed dependencies with `npm install`, start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

### Building

To create a production version of your app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://kit.svelte.dev/docs/adapters) for your target environment.

### Localization

In order for the course structure to be maintained, localized files need to adhere to the following rules:

1. Localized lesson files should be in a subdirectory of `content` named after the language abbreviation. For example, lessons translated into Spanish should be housed in `content/es`.
2. Localized asset files should be in a subdirectory of `assets` named after the language abbreviation. For example, assets localized into Spanish should be housed in `assets/es`.
3. File names for localized files must be identical to their English counterpart. To be clear, _do not translate file names_. The file name is used as the slug for the article and must be identical between languages.
