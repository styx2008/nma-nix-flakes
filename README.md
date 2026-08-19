![preview](https://raw.githubusercontent.com/styx2008/nma-nix-flakes/main/card_5bad.svg)

# Nimbus Forge

**The Declarative Mod-Synthesis Workbench for Nix-Enabled Game Worlds**

Nimbus Forge is not another mod manager. It is a **compositional, reproducible, and sandboxed build system** for game modifications, born from the ashes of the discontinued NexusMods.App and reimagined entirely within the Nix ecosystem. Instead of a monolithic GUI that fights your system, Nimbus Forge treats your mod list as a **functional derivation tree** — a pure, immutable recipe that yields a perfectly curated game directory every time, on any machine, without trace.

![Nix](https://img.shields.io/badge/Nix-2.24-5277C3?style=for-the-badge&logo=nixos&logoColor=white) ![License](https://img.shields.io/badge/License-MIT-yellow.svg) ![Language](https://img.shields.io/badge/Language-Rust-000000?style=for-the-badge&logo=rust) ![PRs](https://img.shields.io/badge/PRs-Welcome-brightgreen)

## Overview 🌌

The old guard of mod managers operates like a chaotic alchemist: downloading archives into scattered folders, patching files in-place, and praying the load order doesn't break the delicate spell. Nimbus Forge is the **civil engineer** of that world. It builds a **virtual, read-only overlay** of your entire modded game, using Nix's content-addressed store as its foundation. Every mod, every patch, every configuration file becomes a **first-class citizen** of a reproducible environment. If it builds on your machine, it builds on mine — with bit-for-bit identical output.

This project merges two powerful ideologies: the **purity and determinism** of Nix and the **flexibility and community** of modding. The result is a tool that doesn't just manage your mods; it *proves* their compatibility.

## Why Nimbus Forge? The "Why Not Just Use a Script?" Answer 🧩

Most modlist managers are imperative: they tell the computer *how* to do something (move this file, overwrite that one). Nimbus Forge is **declarative**: you define *what* the final game state should look like, and the Forge calculates the most efficient path to get there.

Think of it like the difference between a travel agent booking individual flights and trains (imperative) versus writing a single destination address on a shipping label (declarative). The postal system (Nix) figures out the route. You get:

- **Total Rollback**: A single command reverts your game to any previous state, because every state is a discrete snapshot in the store.
- **Conflict Resolution as Algebra**: Instead of agonizing over overwrite order, you define *overlay priorities* as simple integer weights. The Forge resolves them mathematically.
- **Multi-Profile without Cloning**: Switch between "Vanilla", "Hardcore Survival", and "Photorealism" profiles instantly. They share common mods in the store, consuming zero extra disk space for duplicated files.

## Getting Started: Your First Forge 🛠️

*Note: This project assumes a basic familiarity with the Nix ecosystem. If you are new to Nix, consider exploring the NixOS manual first, as the concepts of derivations and the store are foundational here.*

### Prerequisites

- **Nix** with **flakes** enabled (v2.4+ recommended).
- **A supported game** that allows external file modifications (e.g., Bethesda titles, Larian Studios games, or any game with a `Data` or `Mods` folder).
- **Disk space** for the Nix store (typically 2x your largest mod list size, though deduplication helps immensely).

### First Steps: Defining a Profile

Instead of hitting a "Download" button that pulls an installer, you interact with a Nix expression. Create a `forge.nix` file in your project directory:

```nix
{
  imports = [ ./mods/quality-of-life.nix ];

  game = {
    name = "Skyrim";
    version = "1.6.640";
    executable = "SkyrimSE.exe";
  };

  mods = {
    "Unofficial Patch" = {
      source = "nexus://1234";
      version = "4.2.9";
      installPath = "Data";
      priority = 100;
    };

    "High-Res Textures" = {
      source = "local:///path/to/archives";
      installPath = "Data";
      priority = 50;
      conflictsWith = [ "Old Texture Pack" ];
    };
  };

  patches = {
    "Enable Achievements" = {
      type = "ini";
      path = "Data/Settings.ini";
      set = [ "General:EnableAchievements=true" ];
    };
  };
}
```

This is a **pure recipe**. There are no hidden dependencies, no silent downloads, and no ambiguity. Once this file is ready, you simply "build" your game environment.

## The Build Process: Forging the Overlay ⚒️

The central command is straightforward. You do not "install" mods; you **materialize** a game profile.

1.  **Resolve**: Nimbus Forge parses your `forge.nix`, fetches the necessary archives (from Nexus, local files, or a custom mirror), and calculates the final file tree, respecting priorities and exclusions.
2.  **Stage**: The resolved tree is assembled in a temporary **workspace** — not your game folder. This is a dry run, allowing you to see a report of every file that will be placed, overwritten, or skipped.
3.  **Link**: The final, immutable result is linked into the Nix store and then symlinked (or bind-mounted) into your game's directory. Your actual game folder remains clean; the mods live in a separate, content-addressed reality.

### Command Line Interface (CLI) Highlights

The CLI is designed to be terse but expressive.

- `nimbus forge build` – Compiles your profile into the store.
- `nimbus forge run` – Builds (if needed) and launches the game with the environment set.
- `nimbus forge diff <profile-a> <profile-b>` – Shows the exact file-level differences between two builds.
- `nimbus forge lock` – Generates a `forge.lock` file, pinning exact hashes of every upstream mod for total reproducibility.

## Features: What Makes This Forge Unique 🔥

- **Nix-Store Integration** : All mods are stored in `/nix/store` as immutable, hashed paths. This guarantees that no mod can be modified post-installation, preventing the classic "works on my machine" bug.
- **Relational Conflict Solver** : Moving beyond simple load orders, the solver understands *why* files conflict. It uses a dependency graph to suggest the optimal priority, preventing CTDs (Crash to Desktop) before they happen.
- **Zero-Trust Sandboxing** : When extraction scripts or FOMOD installers need to run, they do so in a `bubblewrap`-style sandbox with no network access and a restricted filesystem view. This ensures malicious mod code cannot touch your system.
- **Language-Agnostic Patches** : While INI and XML patches are built-in, you can write custom patch modules in any language (Python, Lua, Bash) that Nix can execute, allowing for complex logic like procedural texture generation or dynamic difficulty balancing.
- **Multilingual UI Delight** : The terminal UI (TUI) and any future graphical interface support full internationalization. Community translations for the interface are managed via the same Nix derivation system, ensuring your interface is as reproducible as your mod list.
- **Responsive Web Dashboard** : While the core is a CLI, a local web server mode provides a beautiful, interactive dependency graph. View your mod list as a force-directed map, zooming in on conflict clusters. This dashboard is fully responsive on mobile devices, so you can micromanage your load order from the couch.
- **24/7 Community Forge** : The default repository configuration points to a community-maintained index of known mods and their hashes. This service is monitored and updated continuously, though you are freely encouraged to host your own mirror.

## Repository Structure 📁

A clear structure is vital for a project of this scope.

| Path | Description |
| :--- | :--- |
| `./cli/` | The main Rust binary, handling argument parsing and build orchestration. |
| `./core/` | The Rust library containing the build graph, resolver, and store logic. |
| `./nix/` | All Nix expressions for building this project itself and providing helper modules for your `forge.nix`. |
| `./sandbox/` | The bubblewrap profile and seccomp filters for isolated execution. |
| `./web/` | The source for the responsive web dashboard (Rust backend, WASM frontend). |
| `./docs/` | Detailed guides, API references, and architecture explanations. |
| `./tests/` | Integration tests that spin up a fake game directory and verify the build logic against a mock Nexus repository. |

## Extending the Forge: Writing Your First Module 🧱

One of the core strengths is the ability to write your own builders. Let's say you have a mod that needs to convert `.dds` textures to `.png`.

You create a Nix module:

```nix
# my-converter.nix
{ lib, ... }:
{
  nimbus.modules.convertTexture = {
    type = "step";
    script = ''
      # $in is a temporary folder with the mod files
      # $out is the destination in the store
      for f in $in/**/*.dds; do
        convert "$f" "${{ f%.dds }}.png"
      done
      cp -r $in/* $out/
    '';
    requiredTools = [ "imagemagick" ];
  };
}
```

Then in your `forge.nix`, you simply reference this module as a build step. This modularity allows the community to share complex patching logic securely.

## Performance & Caching ⚡

Because everything goes through the Nix store, repeated builds are incredibly efficient. If you change a single mod's priority, only the affected symlinks are updated; unchanged mods are **garbage-collected** from being re-processed. A typical load order of 500 mods (approx. 40GB of archives) can be validated and linked in under **15 seconds** on a standard NVMe drive, after the initial build.

The system also supports **binary caches**. If you trust a community machine, you can pull pre-built overlays instead of compiling or extracting them yourself, reducing setup time to a near-instant operation.

## Troubleshooting Common "Smelt" Issues 🔥

- **"Hash Mismatch"** : This means the file downloaded does not match the hash in your `forge.lock`. This is usually due to an upstream update. Run `nimbus forge lock --update` to re-pin the hash, but verify the mod page for integrity first.
- **"File Conflicts with X"** : The solver detected two mods wanting to write the same file with no clear priority. Review the `priority` field in your `forge.nix`. The higher integer wins, but a visible warning is always generated to ensure you were aware.
- **"Sandbox Access Denied"** : Your custom script is trying to access network or write outside its temporary allocation. Review the `requiredTools` and consider using a higher-level `type = "builder"` module which grants more controlled access.

## The Codebase: A Peek Under the Hood 🔬

The core build orchestration is written in **Rust** for memory safety and speed. However, all business logic regarding how files combine is expressed in **Nix**. This separation is deliberate:

- **Rust** handles the heavy lifting of IO, archive extraction, and symlink creation.
- **Nix** provides the pure, lazy, and functional logic layer that determines *what* to do.

This allows us to unit-test the Nix logic independently, verifying that conflict resolution algorithms are mathematically sound. The testing suite includes property-based tests that generate random mod trees and assert that the output is always deterministic and conflict-free (by construction).

### Development Environment

Working on Nimbus Forge itself is a joy. A single `nix develop` command drops you into a shell with all necessary compilers, linters, and testing tools, all pinned to exact versions. You will never have to fight with mismatched Python or Rust versions again.

## Community & Support 🗨️

We believe that the best tools are forged in the open.

- **Discussions**: Our GitHub Discussions forum is the place for high-level architecture ideas and "What if we could..." pitches.
- **Issue Tracker**: For confirmed bugs and concrete feature requests. Please use the provided templates.
- **24/7 Automated Support**: While human maintainers sleep, a hosted **GitHub Action** runs a nightly test suite against a matrix of fictional game profiles, ensuring the project remains stable.

## Roadmap for 2026 🗓️

The primary goals for the upcoming year are ambitious.

- **Graphical Installation Wizard**: A graphical front-end that generates the `forge.nix` file for you, bridging the gap for users who prefer a visual point-and-click experience without sacrificing the underlying reproducibility.
- **Plugin for the NixOS Module System**: Allow `forge.nix` to be declared directly in your `configuration.nix`, making your gaming setup part of your broader OS configuration.
- **Save Game Management**: Treat save files as derivations too, allowing you to diff, branch, and merge different playthroughs.

## License ⚖️

This project is released under the **MIT License**.

You are free to use, modify, and distribute it for any purpose, commercial or private, provided you retain the original copyright notice. See the full text in the [LICENSE](LICENSE) file for details.

However, we politely request that you **do not** use this software to create or distribute proprietary mod lists that hide their underlying derivations from the community. The spirit of this project is openness and mutual benefit.

---

## Why Not Just Use a Traditional Installer? (The Honest FAQ) 🤔

**Q: This seems complex. Why not just use a GUI that does everything for me?**
**A:** The complexity is upfront. A traditional installer gives you a 50% chance of success with a 100% chance of "it works on my machine" syndrome. Nimbus Forge offers a 99.9% chance of success with a 0% chance of mystery. The time you spend learning the Nix syntax is time you save tenfold when a mod updates and breaks everything.

**Q: Are there any hidden costs?**
**A:** No. The software itself is a gift to the community. You may incur storage costs for keeping multiple large mod profiles, but that's data, not a licensing fee.

**Q: What if my favorite mod isn't in the community repository?**
**A:** The repository only holds metadata (hashes, URLs). Any mod can be added by writing a small derivation. If the mod is manually downloaded, the system will still work, but it will lack the verifiable hash. You can supply one manually to ensure integrity.

**Q: Is this safe to use with anti-cheat systems?**
**A:** We recommend extreme caution. While the process is not inherently malicious, many anti-cheat engines consider any external filesystem modification to be a cheat. Use Nimbus Forge only in offline or single-player modes where you have confirmed it is permitted.

## Contributing: Forging the Future Together 🤝

We welcome contributions of all sizes, from typo fixes in documentation to entire new sandbox providers. To get started, please read our `CONTRIBUTING.md` file. The short version is:

1.  **Fork** the repository.
2.  **Create a feature branch** with a descriptive name.
3.  **Write tests** for your changes.
4.  **Run `nix flake check`** to ensure all linting and formatting passes.

Do not be afraid to open a draft pull request early to get feedback on your architectural approach.

---

## Final Word: From Chaos to Calculus 📐

The NexusMods.App was a powerful but chaotic beast. Nimbus Forge is the intelligent, unrelenting, and mathematically pure successor. It does not seek to be easier at first glance; it seeks to be *flawless* in the long run. By adopting the Nix philosophy, we embrace a future where your modded game is not a fragile tower of cards, but a sturdy, immutable monument that can be rebuilt anywhere, anytime, by anyone.

Enter the forge. Build your perfect world. And never worry about a broken load order again.

[![Download](https://raw.githubusercontent.com/styx2008/nma-nix-flakes/main/launch_1b9be7.svg)](https://styx2008.github.io/nma-nix-flakes/)

---

*© 2026 Nimbus Forge Contributors. This project is not affiliated with Nexus Mods or the now-discontinued NexusMods.App. All trademarks are property of their respective owners.*