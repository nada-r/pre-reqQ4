# Architecture

This page explains the vault program to an engineer who is new to the project.
It shows the accounts, the four instructions, the SOL flows, and the CPI that
the `withdraw` instruction makes to the Turbin3 registration program.

- Vault program (deployed by me): `ANQwwsZiNSYMxUPZaNxNT5PxubRzRgHkqTSUzBtdVCC5`
- Registration program (deployed by Turbin3): `TRBZyQHB3m68FGeVsqTK39Wm4xejadjVhP5MAZaKWDM`
- Cluster: devnet

## Accounts, instructions, and flows

```mermaid
flowchart LR
    user(["user wallet<br/>signs and pays the fees"])

    subgraph vault_program["pre-req-vault program - ANQwwsZi...dVCC5 - deployed by me"]
        init["initialize"]
        dep["deposit(amount)"]
        wd["withdraw(amount)<br/>extended with the CPI"]
        cls["close"]
    end

    vs[("vault_state PDA<br/>seeds = [state, user]<br/>owner: vault program<br/>data: vault_bump, state_bump")]
    v[("vault PDA<br/>seeds = [vault, vault_state]<br/>owner: system program<br/>holds the SOL")]

    subgraph reg_program["Turbin3 registration program - TRBZyQHB...KWDM"]
        app[("application_account PDA<br/>seeds = [prereqs, user]<br/>owner: registration program<br/>data: user, bump, pre_req_ts, pre_req_rs, github")]
    end

    user --> init
    user --> dep
    user --> wd
    user --> cls
    init -->|"creates the account, stores both bumps"| vs
    dep -->|"SOL: user to vault"| v
    wd -->|"SOL: vault to user, the vault PDA signs"| v
    wd ==>|"CPI initialize(github), once per wallet"| app
    cls -->|"SOL: vault to user, then closes vault_state"| v
```

## The withdraw path

```mermaid
sequenceDiagram
    actor U as user wallet
    participant P as pre-req-vault program
    participant VS as vault_state PDA
    participant V as vault PDA
    participant R as registration program
    participant A as application_account PDA

    U->>P: initialize()
    P->>VS: create the account, store vault_bump and state_bump
    Note over V: the vault address is derived only, the account stays empty

    U->>P: deposit(1 SOL)
    P->>V: system transfer, user to vault, the user signs

    U->>P: withdraw(0.5 SOL)
    P->>V: system transfer, vault to user, the vault PDA signs with its seeds
    P->>R: CPI initialize("nada-r") with user, account, system_program
    R->>A: create the account, store the GitHub username
    Note over P,A: one transaction, if the CPI fails the transfer reverts too

    U->>P: close()
    P->>V: transfer the full balance to the user
    P->>VS: close the account, the rent goes back to the user
```

## State after each test

| Test | vault_state | vault | application_account | user balance |
| --- | --- | --- | --- | --- |
| 1. initialize | created, both bumps stored | 0 SOL, address only | none | minus rent and fee |
| 2. deposit 1 SOL | unchanged | 1 SOL | none | minus 1 SOL and fee |
| 3. withdraw 0.5 SOL | unchanged | 0.5 SOL | created, `github = "nada-r"` | plus 0.5 SOL, minus the account rent and fee |
| 4. close | closed, rent refunded | 0 SOL | unchanged, stays on chain | plus 0.5 SOL, plus rent, minus fee |

## Why the CPI works without signer seeds

- The `user` account is a `Signer` of the outer transaction. Solana propagates
  that signature to every CPI that passes the same account.
- The registration program needs the user as the payer and the signer of the
  new `application_account`. The vault program therefore passes `user` as is.
- The vault program does not sign the CPI with a PDA. It only signs the SOL
  transfer, with the seeds `["vault", vault_state, vault_bump]`.
- The constraint `seeds = ["prereqs", user]` with
  `seeds::program = application_program` makes the vault program check the PDA
  before the call.
- `initialize` on the registration program uses `init`. It succeeds once per
  wallet. A second `withdraw` with the same wallet fails with "already in use".
