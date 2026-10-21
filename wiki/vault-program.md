---
title: Vault program
type: page
updated: 2026-10-21
sources: [programs/pre-req-vault/src, idls/registration.json, tests/pre-req-vault.ts]
---

# Vault program

Anchor 1.1.2 program. Program ID on devnet:
`ANQwwsZiNSYMxUPZaNxNT5PxubRzRgHkqTSUzBtdVCC5`. Source:
`programs/pre-req-vault/src`. The diagram is in `docs/architecture.md`.

## Instructions

| Instruction | Accounts | Effect |
| --- | --- | --- |
| `initialize` | user (signer, mut), vault_state (init, payer = user), vault (PDA, system account), system_program | Creates `vault_state` and stores `vault_bump` and `state_bump`. |
| `deposit(amount)` | user, vault (mut), vault_state, system_program | System transfer of `amount` lamports from user to vault. The user signs. |
| `withdraw(amount)` | user, vault (mut), vault_state, application_account (mut), application_program, system_program | System transfer of `amount` lamports from vault to user, signed with the vault seeds. Then a CPI `initialize(github)` to the registration program. |
| `close` | user, vault (mut), vault_state (mut, close = user), system_program | Transfers the full vault balance to the user, then closes `vault_state` and refunds its rent. |

## PDAs

| Account | Seeds | Derived under | Owner | Data |
| --- | --- | --- | --- | --- |
| `vault_state` | `["state", user]` | vault program | vault program | `vault_bump: u8`, `state_bump: u8` (8 + 2 bytes) |
| `vault` | `["vault", vault_state]` | vault program | system program | lamports only |
| `application_account` | `["prereqs", user]` | registration program (`seeds::program`) | registration program | `user: pubkey`, `bump: u8`, `pre_req_ts: bool`, `pre_req_rs: bool`, `github: string` |

## The CPI (task 2)

- `declare_program!(registration)` reads `idls/registration.json`. It
  generates `registration::cpi::accounts::Initialize`,
  `registration::cpi::initialize`, and `registration::program::Q3PreReqsRs`.
- `withdraw` builds `Initialize { user, account: application_account, system_program }`,
  a `CpiContext::new(application_program, accounts)`, and calls
  `initialize(ctx, GITHUB_USERNAME.to_string())`.
- `GITHUB_USERNAME` is the constant `"nada-r"` in `constants.rs`. The
  constant keeps the TypeScript signature `withdraw(amount)` unchanged.
- No signer seeds: `user` signs the outer transaction, and the signature
  propagates through the CPI. The registration program needs the user as the
  payer and the signer of the new account.
- The constraint `seeds = [b"prereqs", user.key()]` with
  `seeds::program = application_program.key()` checks the PDA before the call.
- `initialize` on the registration program uses `init`. It succeeds once per
  wallet. A second `withdraw` with the same wallet fails with "already in use".
- The instruction is atomic. A failed CPI reverts the SOL transfer.

## Registration program

`TRBZyQHB3m68FGeVsqTK39Wm4xejadjVhP5MAZaKWDM` on devnet. IDL name:
`q3_pre_reqs_rs`. Instructions: `initialize(github)`, `update(github)`,
`close`, `submit_ts`, `submit_rs`, `create_collection`. Its last deploy is in
slot 411635705.

## Test flow (`tests/pre-req-vault.ts`)

1. `initialize`: asserts both bumps in `vault_state`.
2. `deposit` 1 SOL: vault balance +1 SOL, user balance below start minus 1 SOL.
3. `withdraw` 0.5 SOL: vault balance -0.5 SOL, user balance above start. Then
   fetches `application_account` with the registration IDL and asserts
   `github == "nada-r"`.
4. `close`: vault balance 0, `vault_state` null, user balance above start.

## Unused template code

`COUNTER_SEED`, `HELLO_WORLD_LAMPORTS`, `MAX_COUNT`, and the two `ErrorCode`
variants come from the template. No instruction uses them.
