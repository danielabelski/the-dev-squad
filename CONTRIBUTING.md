# Contributing to The Dev Squad

Thanks for your interest in contributing! Here's how to get involved.

## Getting Started

1. Fork the repo
2. Clone your fork: `git clone https://github.com/YOUR-USERNAME/the-dev-squad.git`
3. Install dependencies: `pnpm install`
4. Start the dev server: `pnpm dev`
5. Make your changes
6. Test that the viewer loads and the pipeline works
7. Submit a pull request

## Requirements

- Node.js 22+
- pnpm
- Claude Code CLI (`claude` command)
- Active Claude subscription

## What to Contribute

**Good first contributions:**
- Bug fixes
- UI improvements to the viewer
- Better pixel-art furniture/props for the office scene
- Documentation improvements
- New idle behaviors for agents

**Bigger contributions:**
- Additional pipeline stages
- New agent roles
- Alternative orchestration strategies
- Integration with other AI models

## Code Guidelines

- This is a Next.js app with TypeScript and Tailwind CSS
- The pipeline backend lives in `pipeline/` — orchestrator, hooks, role files
- The viewer lives in `src/` — standard Next.js app router structure
- Keep the pixel art style consistent if adding visual elements
- Run `pnpm test:hook` after changing hook or orchestrator behavior
- Run `pnpm test:signals` after changing structured signal parsing
- Test your changes with an actual pipeline run before submitting

## Architecture

Read [ARCHITECTURE.md](ARCHITECTURE.md) for a full breakdown of the system design.

Key principles:
- **Scripts enforce, markdown suggests** — agent restrictions are enforced by hooks, not prompts
- **The plan IS the code** — plans contain complete, copy-pasteable code for every file
- **Structured signals** — agents communicate via JSON, not free text
- **The orchestrator is deterministic** — it's code, not an LLM

## Asset Guidelines

- All assets in the public repo must be CC0, MIT, or original work
- Do not add assets from paid sprite packs
- Match the existing mixed approach: original pixel sprites and CSS-drawn props are both fine if the scene stays visually consistent
- Character sprites must include walk cycles (front, back, left, right)

## Pull Requests

- Keep PRs focused — one feature or fix per PR
- Include a clear description of what changed and why
- If it's a visual change, include a screenshot
- Make sure the app builds without new errors

## Issues

Found a bug? Have an idea? [Open an issue](https://github.com/johnkf5-ops/the-dev-squad/issues).

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
