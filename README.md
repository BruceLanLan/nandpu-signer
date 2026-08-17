# NANDPU Mobile Signer (fixed surface)

The permanent human-in-the-loop signing page for NANDPU tapeouts.
Live: **https://brucelanlan.github.io/nandpu-signer/**

See `nandpu/docs/decisions.md` D-013 and `docs/agent-signing-handoff.md`
in the private `BruceLanLan/nandpu` repository for the full protocol.

## Per release window

1. In `nandpu`: run the window's preflight + `npm run prepare:<circuit>`.
2. Copy `nandpu/artifacts/unsigned-transactions.json` → `manifest.json` here.
3. Edit `index.html`:
   - the `EXPECTED` block (chainId, mode, currentNextId, expectedCircuitId,
     circuit key, NAND burn, Circuits address, creator);
   - every visible label (circuit name, ID, burn, lede text).
4. Validate: `grep -c "eth_sendTransaction" index.html` must be 1; the
   manifest must contain exactly one transaction with matching fields.
5. `git add -A && git commit && git push` — Pages publishes automatically.
6. `curl` the live URL and manifest; confirm the new circuit is shown.
7. Give the owner: `https://brucelanlan.github.io/nandpu-signer/?v=<key>`.

## After the receipt

1. Verify on chain (circuitInfo + eval vectors).
2. Delete `manifest.json`; replace `index.html` with the CLOSED notice.
3. Push. The page must expose no wallet action after the window.

## Rules

- One transaction per window, never batch (D-011).
- The page is public by design during the window (Binance Wallet
  requirement); it is closed immediately after the receipt.
- No secrets live here: the manifest is an unsigned transaction.
