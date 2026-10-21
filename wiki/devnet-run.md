---
title: Devnet run
type: page
updated: 2026-10-21
sources: [anchor deploy output, anchor test output, solana confirm -v, decoded application account]
---

# Devnet run (2026-10-21)

The single deploy and the single test run that produced the submission proof.

## Identifiers

| Item | Value |
| --- | --- |
| Program ID | `ANQwwsZiNSYMxUPZaNxNT5PxubRzRgHkqTSUzBtdVCC5` |
| Upgrade authority | `6M3LySJ4D9y8PwwgeCMCs56MHAUa8ZJh3PYn5vmxGghE` (the test wallet) |
| Deployed in slot | 513431680, 176 888 bytes, SBPF v0 |
| Deploy transaction | `3RXgwZ2HaTYzUtvUxB82hytLaxRfcso3uD77qMVT5Sd6ZeN4T59aEGRjFQY9JCoTZMB5MKhQDW5yktcqUhJ5wqeV` (06:26:50 UTC) |
| IDL account (written by `anchor deploy`) | `8g48zKFr593DTfm5PLTaRtDiCi2goU5oF42UGceJr2mq` |
| Wallet | `6M3LySJ4D9y8PwwgeCMCs56MHAUa8ZJh3PYn5vmxGghE` = `~/.config/solana/id.json` |
| `vault_state` (`["state", user]`) | `4cczhrZfTZ5NwE6dpWmCoAxtDdnNXt1YPdLRpSnQnehx`, closed by test 4 |
| `vault` (`["vault", vault_state]`) | `3NNg4orReN2VGVwyNfCti1CwkYBsDrm5LYXaFMCSpimB`, empty after test 4 |
| `application_account` (`["prereqs", user]`) | `Gw9xH4cir8sefpY2TLKpAss9DDBabQYctizuuVktAbTs`, 72 bytes, 0.001016 SOL |

## Commands and results

1. `anchor deploy` at 06:26 UTC: `Deploy success`, plus `IDL initialized`.
2. `anchor test --skip-build --skip-deploy` at 06:27 UTC: `4 passing (12s)`.
   The devnet RPC answered `429 Too Many Requests` several times during the
   close test. The client retried and the test passed.

## Signatures

| Test | Signature |
| --- | --- |
| Initialize | `4u9M2GQmB7GbDkzwVH6est8V5dRLKJtm74kiVpDFDNkHsFEVesVb2C3K4HUUgEYshzA6gBqDRvfAMmRZncKTsDib` |
| Deposit 1 SOL | `4mburjmGFBCKHQktsXB5pHatmMesDKWexoYyvug1QzoPqJGycNc7z6LmKz39hUqtJXH156oNQLU99h7NzxNt4efr` |
| Withdraw 0.5 SOL + CPI | `4QnkxFF8AbCRiC7aVAVgYtVqFx2GZuyrNZjagP7x5RbhTu4k1B9UZHQgHSNb1Y4CU3utK3tt3sdQaRzrNVskxa76` |
| Close | `4ByvVZCehCaza2yJYCuom3z5RZCWwF8FMJgULStcYtbsQNvz3fMcEpHv3WbpRFBDzis4fSJUq2DFGN2DwH4F4QaT` |

Explorer: <https://explorer.solana.com/tx/4QnkxFF8AbCRiC7aVAVgYtVqFx2GZuyrNZjagP7x5RbhTu4k1B9UZHQgHSNb1Y4CU3utK3tt3sdQaRzrNVskxa76?cluster=devnet>

## Withdraw transaction (`solana confirm -v`)

Slot 513431795, block time 2026-10-21T06:27:17Z, status Ok, finalized.

Accounts: user (fee payer, signer), vault (writable), application_account
(writable), system program, vault_state (read), the vault program
(executable), the registration program.

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

## Decoded application account

Decoded with `BorshAccountsCoder` from `@anchor-lang/core` and
`idls/registration.json`:

```
owner      TRBZyQHB3m68FGeVsqTK39Wm4xejadjVhP5MAZaKWDM
user       6M3LySJ4D9y8PwwgeCMCs56MHAUa8ZJh3PYn5vmxGghE
bump       255
pre_req_ts false
pre_req_rs false
github     "nada-r"
```

## Balances

Wallet: 16.031132 SOL before, 15.123447 SOL after the deploy, 15.122411 SOL
after the tests. The deploy cost 0.907685 SOL (program rent 0.899470, IDL
account 0.006441, fees). The four tests cost 0.001036 SOL, of which 0.001016
SOL is the rent of the application account.
