# Errors

Record each error here: the date, the command, the error text, the cause and the fix.

## 2026-10-21 - `anchor build`: no such command `+1.89.0-sbpf-solana-v1.52`

- Command: `anchor build`
- Error: `error: no such command: '+1.89.0-sbpf-solana-v1.52'`
- Cause: Homebrew `cargo` (`/usr/local/bin/cargo`, rustc 1.99.0) comes before
  `~/.cargo/bin` in PATH. `cargo-build-sbf` runs `cargo +<toolchain>`, and only
  the rustup proxy understands the `+toolchain` switch.
- Fix: put the rustup proxy first: `env PATH="$HOME/.cargo/bin:$PATH" anchor build`.

## 2026-10-21 - `anchor build`: `ld` segfault during the IDL build

- Command: `env PATH="$HOME/.cargo/bin:$PATH" anchor build`
- Error: `clang: error: unable to execute command: Segmentation fault: 11`
  while linking the `syn` build script, then `Error: Building IDL failed`.
- Cause: the shell runs under Rosetta 2. `cc` and `ld` are universal binaries.
  A translated parent process starts them as x86_64, and the x86_64 `ld`
  crashes on the arm64 link.
- Fix: run the whole build natively:
  `arch -arm64 env PATH="$HOME/.cargo/bin:$PATH" anchor build`.
  The SBF program had already compiled. The retry produced the IDL and the
  TypeScript types.

## 2026-10-21 - `anchor keys sync` updated one cluster entry only

- Command: `anchor keys sync`
- Symptom: `[programs.devnet]` got the new ID, `[programs.localnet]` kept the
  old one, and Anchor 1.1.2 rewrote `Anchor.toml` with a default `[test]` block.
- Cause: `anchor keys sync` syncs the entry of the current cluster only.
- Fix: set both entries by hand and restore the file layout.
