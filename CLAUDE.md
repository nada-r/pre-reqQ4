# CLAUDE.md

Turbin3 Q4 2026 prerequisite: an Anchor vault program whose `withdraw`
instruction makes a CPI to the Turbin3 registration program on devnet.

## Commands

- `pnpm install`
- `anchor build` (on this Mac: `arch -arm64 env PATH="$HOME/.cargo/bin:$PATH" anchor build`, see `error.md`)
- `anchor keys sync`
- `anchor deploy` (`cluster = "devnet"` in `Anchor.toml`)
- `anchor test --skip-build --skip-deploy`

## Rules

- Use your own program ID only. Never call the registration program directly from the tests.
- The registration is single-shot per wallet. Never run the withdraw test twice on the same wallet.
- No keypairs in git. `target/` and `*-keypair.json` stay ignored.

## Analysis validation

- Validate the methodology first. Challenge the baselines.
- Cross-check every claim against real on-chain data: `solana confirm -v`, the explorer, decoded accounts.
- If something looks off, stop and verify the fundamentals before the next step.
- Never compound a faulty assumption across steps.

## Memory

- `wiki/index.md`: the index of the wiki pages.
- `wiki/log.md`: a dated, append-only log of each step.
- `error.md`: each error with its cause and its fix.
