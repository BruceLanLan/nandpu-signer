# NANDPU Mobile Signer (fixed surface)

The permanent human-in-the-loop signing page for NANDPU tapeouts.
Live: **https://brucelanlan.github.io/nandpu-signer/**

See `nandpu/docs/decisions.md` D-013 and `docs/agent-signing-handoff.md`
in the private `BruceLanLan/nandpu` repository for the full protocol.

## Per release window

1. In `nandpu`: run the window's preflight + `npm run prepare:<circuit>`.
2. Copy `nandpu/artifacts/unsigned-transactions.json` → `manifest.json` here.
3. Copy `template/index.html` over `index.html` and edit it:
   - the `EXPECTED` block (chainId, mode, currentNextId, expectedCircuitId,
     circuit key, NAND burn, Circuits address, creator);
   - every visible label (circuit name, ID, burn, lede text);
   - the confirmation code (contract address chars 3-6) in BOTH the
     visible hint and the `send()` check.
4. Validate: `grep -c "eth_sendTransaction" index.html` must be 1; the
   manifest must contain exactly one transaction with matching fields.
5. `git add -A && git commit && git push` — Pages publishes automatically.
6. `curl` the live URL and manifest; confirm the new circuit is shown.
7. Give the owner: `https://brucelanlan.github.io/nandpu-signer/?v=<key>`.

## After the receipt

1. Verify on chain (circuitInfo + eval vectors).
2. Delete `manifest.json`; replace `index.html` with the CLOSED notice.
3. Push. The page must expose no wallet action after the window.

## Security posture (R11 hardening, 2026-08-17)

The console template (`template/index.html`) enforces, in order:
1. manifest invariant checks (mode/chainId/nextId/one-tx/circuit-key/
   burn/contract/selector `0x7bd3ac1d` — the page refuses to sign any
   other method);
2. wallet-side side-by-side comparison (chain 0x38, contract prefix,
   0 BNB) before the checkbox;
3. a 4-character confirmation code (contract address chars 3-6) typed by
   the signer;
4. live `nextId` re-check immediately before `eth_sendTransaction`.

No external CDN dependencies (self-contained HTML), no URL parameters are
ever read — nothing on the page can be redirected by a crafted link.

## Rules

- One transaction per window, never batch (D-011).
- The page is public by design during the window (Binance Wallet
  requirement); it is closed immediately after the receipt.
- No secrets live here: the manifest is an unsigned transaction.
