---
title: Wiki log
type: log
updated: 2026-10-21
sources: []
---

# Wiki log

Append-only. `grep "^## \[" wiki/log.md | tail -5` shows the last five entries.

## [2026-10-21] decision | Registration program and wallet

Checked on devnet: `TRBZ...KWDM` is the registration program named in the
challenge. Its last deploy is in slot 411635705. The IDL in the upstream
template repo is byte-identical to `idls/registration.json`. Other applicants
registered on the same program on 2026-10-21. Wallet for the run:
`6M3LySJ4D9y8PwwgeCMCs56MHAUa8ZJh3PYn5vmxGghE` (`~/.config/solana/id.json`).

## [2026-10-21] step | New program ID

`solana-keygen new` wrote a new program keypair:
`ANQwwsZiNSYMxUPZaNxNT5PxubRzRgHkqTSUzBtdVCC5`. `anchor keys sync` updated
`declare_id!` and `[programs.devnet]`; `[programs.localnet]` was set by hand.
`Anchor.toml` wallet set to `~/.config/solana/id.json`. `grep DBoobRVq` finds
nothing.

## [2026-10-21] step | Pre-flight on devnet

`["prereqs", 6M3L...]` under `TRBZ...` = `Gw9xH4cir8sefpY2TLKpAss9DDBabQYctizuuVktAbTs`:
does not exist. `["state", 6M3L...]` = `4cczhrZfTZ5NwE6dpWmCoAxtDdnNXt1YPdLRpSnQnehx`
and `["vault", vault_state]` = `3NNg4orReN2VGVwyNfCti1CwkYBsDrm5LYXaFMCSpimB`:
do not exist. Wallet balance: 16.03 SOL.

## [2026-10-21] step | Build

`arch -arm64 env PATH="$HOME/.cargo/bin:$PATH" anchor build` passed after two
failures (see `error.md`). Artifacts: `pre_req_vault.so` (176 888 bytes, SBPF
v0), IDL with address `ANQww...`, `target/types/pre_req_vault.ts`. Devnet
accepts SBPF v0: SIMD-0500 is inactive, and an applicant program deployed the
same day has ELF flags 0x0.

## [2026-10-21] step | Project folder

The project lives in `~/Dev/turbin3-q4-prereq-vault`: the sources,
`Anchor.toml`, `idls/`, `tests/`, and the docs. No build cache, no
`node_modules`, and no git history came with it. The user creates the GitHub
repo and commits.

## [2026-10-21] step | Deploy

`anchor deploy` from the new folder at 06:26 UTC: `Deploy success`, IDL
account `8g48zKFr593DTfm5PLTaRtDiCi2goU5oF42UGceJr2mq` written. Program
`ANQww...` deployed in slot 513431680, upgrade authority `6M3L...`. Cost:
0.907685 SOL.

## [2026-10-21] step | Tests and proof

`anchor test --skip-build --skip-deploy`: 4 passing (12 s). Withdraw + CPI
signature `4QnkxFF8...xa76`, slot 513431795, finalized. The logs show
`ANQww... invoke [1]` then `TRBZ... invoke [2]` with `Instruction: Initialize`.
The application account `Gw9xH4...AbTs` decodes to `github = "nada-r"`,
`user = 6M3L...`. `vault_state` and `vault` are closed. Details in
devnet-run.md. README.md rewritten with the proof.

## [2026-10-21] step | GitHub repo

The user created `github.com/nada-r/pre-reqQ4`. First commit from this folder,
with the remote `origin` set to that repo. `.gitignore` keeps `target/`,
`node_modules/`, `tests/fixtures/*.so`, and `.claude/settings.local.json` out
of git. No keypair is tracked.
