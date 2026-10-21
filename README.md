# pre-reqQ4

Turbin3 Q4 2026 Builders Cohort, prerequisite Part 1. An Anchor vault program
whose `withdraw` instruction makes a CPI to the Turbin3 registration program
on devnet and records my GitHub username (`nada-r`).

- Program ID (deployed by me): `ANQwwsZiNSYMxUPZaNxNT5PxubRzRgHkqTSUzBtdVCC5`
- Wallet: `6M3LySJ4D9y8PwwgeCMCs56MHAUa8ZJh3PYn5vmxGghE`
- Withdraw + registration CPI transaction:
  `4QnkxFF8AbCRiC7aVAVgYtVqFx2GZuyrNZjagP7x5RbhTu4k1B9UZHQgHSNb1Y4CU3utK3tt3sdQaRzrNVskxa76`
- Architecture diagrams: [docs/architecture.md](docs/architecture.md)

Explorer links (devnet):

- [Withdraw + CPI transaction](https://explorer.solana.com/tx/4QnkxFF8AbCRiC7aVAVgYtVqFx2GZuyrNZjagP7x5RbhTu4k1B9UZHQgHSNb1Y4CU3utK3tt3sdQaRzrNVskxa76?cluster=devnet)
- [Application account](https://explorer.solana.com/address/Gw9xH4cir8sefpY2TLKpAss9DDBabQYctizuuVktAbTs?cluster=devnet)
- [Program](https://explorer.solana.com/address/ANQwwsZiNSYMxUPZaNxNT5PxubRzRgHkqTSUzBtdVCC5?cluster=devnet)

## What I changed and why

`programs/pre-req-vault/src/instructions/withdraw.rs`:

- After the SOL transfer, the instruction builds
  `Initialize { user, account: application_account, system_program }`, a
  `CpiContext::new(application_program, accounts)`, and calls
  `initialize(ctx, GITHUB_USERNAME.to_string())`.
- The `user` is the payer and the signer of the new account. The user already
  signed the outer transaction, and a signature propagates through a CPI. The
  vault program passes no signer seeds for this call.
- The `system_program` account is passed because the registration program
  creates the `application_account`.
- The whole instruction is atomic. If the CPI fails, the transfer reverts too.

`programs/pre-req-vault/src/constants.rs`: a constant `GITHUB_USERNAME = "nada-r"`.
A constant keeps the TypeScript signature `withdraw(amount)` unchanged, so the
given tests run as written.

`programs/pre-req-vault/src/lib.rs` and `Anchor.toml`: my own program ID, and a
`[programs.devnet]` entry.

`tests/pre-req-vault.ts`: the withdraw test also fetches the application
account with the registration IDL and asserts `github == "nada-r"`.

## Accounts

| Account | Type | Seeds | Role |
| --- | --- | --- | --- |
| `user` | `Signer` | wallet keypair | Owns the vault. Signs and pays for every instruction. |
| `vault_state` | `Account<VaultState>` | `["state", user]` | Stores `vault_bump` and `state_bump`. |
| `vault` | `SystemAccount` | `["vault", vault_state]` | A system account PDA that holds the deposited SOL. |
| `application_account` | `UncheckedAccount` | `["prereqs", user]` with `seeds::program = application_program` | Created by the registration program during the CPI. Stores the GitHub username. |
| `application_program` | `Program` | n/a | The Turbin3 registration program `TRBZyQHB3m68FGeVsqTK39Wm4xejadjVhP5MAZaKWDM`. |
| `system_program` | `Program` | n/a | Creates accounts and moves lamports. |

## Instructions

- `initialize`: creates `vault_state` and stores both PDA bumps. The `vault`
  address is derived, but the account stays empty until the first deposit.
- `deposit(amount)`: system transfer of `amount` lamports from `user` to
  `vault`. The user signs, so no PDA seeds are needed.
- `withdraw(amount)`: system transfer of `amount` lamports from `vault` to
  `user`, signed with the seeds `["vault", vault_state, vault_bump]`. Then the
  CPI to the registration program.
- `close`: transfers the full vault balance back to the user, then closes
  `vault_state` and refunds its rent.

## Proof on devnet (2026-10-21)

Program deployed in slot 513431680 with `6M3L...GghE` as the upgrade authority.
Deploy transaction:
`3RXgwZ2HaTYzUtvUxB82hytLaxRfcso3uD77qMVT5Sd6ZeN4T59aEGRjFQY9JCoTZMB5MKhQDW5yktcqUhJ5wqeV`.

Test output of `anchor test --skip-build --skip-deploy` (RPC retry lines removed):

```
  pre-req-vault
Transaction signature: 4u9M2GQmB7GbDkzwVH6est8V5dRLKJtm74kiVpDFDNkHsFEVesVb2C3K4HUUgEYshzA6gBqDRvfAMmRZncKTsDib
    ✔ Initialize the vault (1605ms)
Transaction signature: 4mburjmGFBCKHQktsXB5pHatmMesDKWexoYyvug1QzoPqJGycNc7z6LmKz39hUqtJXH156oNQLU99h7NzxNt4efr
    ✔  Deposilt 1 Sol in to the vault (1261ms)
Transaction signature: 4QnkxFF8AbCRiC7aVAVgYtVqFx2GZuyrNZjagP7x5RbhTu4k1B9UZHQgHSNb1Y4CU3utK3tt3sdQaRzrNVskxa76
Registered github: nada-r application account: Gw9xH4cir8sefpY2TLKpAss9DDBabQYctizuuVktAbTs
    ✔  Withdraw 0.5 Sol from the vault (2218ms)
Transaction signature: 4ByvVZCehCaza2yJYCuom3z5RZCWwF8FMJgULStcYtbsQNvz3fMcEpHv3WbpRFBDzis4fSJUq2DFGN2DwH4F4QaT
    ✔  Close the vault and withdraw all the funds (6802ms)

  4 passing (12s)
```

Program logs of the withdraw transaction (`solana confirm -v <sig> -ud`):

```
Program ANQwwsZiNSYMxUPZaNxNT5PxubRzRgHkqTSUzBtdVCC5 invoke [1]
Program log: Instruction: Withdraw
Program 11111111111111111111111111111111 invoke [2]
Program 11111111111111111111111111111111 success
Program TRBZyQHB3m68FGeVsqTK39Wm4xejadjVhP5MAZaKWDM invoke [2]
Program log: Instruction: Initialize
Program 11111111111111111111111111111111 invoke [3]
Program 11111111111111111111111111111111 success
Program TRBZyQHB3m68FGeVsqTK39Wm4xejadjVhP5MAZaKWDM consumed 8220 of 189286 compute units
Program TRBZyQHB3m68FGeVsqTK39Wm4xejadjVhP5MAZaKWDM success
Program ANQwwsZiNSYMxUPZaNxNT5PxubRzRgHkqTSUzBtdVCC5 consumed 19307 of 200000 compute units
Program ANQwwsZiNSYMxUPZaNxNT5PxubRzRgHkqTSUzBtdVCC5 success
```

The application account `Gw9xH4cir8sefpY2TLKpAss9DDBabQYctizuuVktAbTs`, decoded
with `idls/registration.json`:

```
owner      TRBZyQHB3m68FGeVsqTK39Wm4xejadjVhP5MAZaKWDM
user       6M3LySJ4D9y8PwwgeCMCs56MHAUa8ZJh3PYn5vmxGghE
bump       255
pre_req_ts false
pre_req_rs false
github     "nada-r"
```

After the `close` test, `vault_state` and `vault` no longer exist.

## How to run

Versions used: anchor-cli 1.1.2, anchor-lang 1.1.2, Agave (Solana CLI) 4.2.1,
Rust 1.89+ (`rust-version` in `Cargo.toml`), pnpm 10, Node 25.

```bash
pnpm install
anchor keys sync        # switch to your own program ID
anchor build            # on a Mac with a Rosetta shell, see error.md
anchor deploy           # cluster = "devnet" in Anchor.toml
anchor test --skip-build --skip-deploy
```

Caution: the registration is single-shot per wallet. The first successful
`withdraw` creates the `["prereqs", user]` account. A second run with the same
wallet fails with "already in use".

Not run: the LiteSVM test in `programs/pre-req-vault/tests/test_initialize.rs`.
It predates the CPI and is out of scope for this challenge.
