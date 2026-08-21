# NANDPU / TapeOut mobile signer

Permanent human-in-the-loop signing pages. Live on GitHub Pages:

**https://brucelanlan.github.io/nandpu-signer/**

Two independent windows. Wrong processor = NAND burned on the wrong contract. Do not mix.

| Window | Live URL | Processor | Confirm code |
|---|---|---|---|
| NANDPU #45–#48 | https://brucelanlan.github.io/nandpu-signer/ | `0x8633…28aA` | `332F` |
| TapeOut mining pack | **https://brucelanlan.github.io/nandpu-signer/tapeout/** | `0xb102…a14C` | `024B` |

TapeOut aliases (same pack): `/tapeout-mine.html`, `/mine/`.

See `nandpu/docs/decisions.md` D-013 and `docs/agent-signing-handoff.md`
in the private `BruceLanLan/nandpu` repository for the NANDPU protocol.

---

## TapeOut mining pack (this repo, `/tapeout/`)

Five unsigned `tapeout()` transactions against the **official TapeOut**
processor (`0xb1024b89886b9a34Aa4ff5F31C411D708b20a14C`), not NANDPU.

- Burns **485 TapeOut NAND** (wallet holds 500). Tasks 61 / 31 / 149 / 147 / 222.
- TapeOut IDs are permissionless: the page **retargets to live `nextId`**
  on load and again before each signature. Frozen IDs in the HTML are a
  snapshot; the live chain wins.
- Confirm code is contract chars 3–6: **`024B`**. Wallet popup must show
  chain `0x38`, contract prefix `b102`, value `0 BNB`.
- Page does not hold keys and does not broadcast (D-001). Human signs
  in Binance Wallet.

Do **not** overwrite root `index.html` / `manifest.json` with this pack.
Those files are the NANDPU window.

After the owner signs: verify `circuitInfo` + `eval` on TapeOut, then
replace `/tapeout/index.html` with a CLOSED notice (same as NANDPU).

---

## NANDPU per release window

1. In `nandpu`: run the window's preflight + `npm run prepare:<circuit>`.
2. Copy `nandpu/artifacts/unsigned-transactions.json` → `manifest.json` here.
3. Copy `template/index.html` over `index.html` and edit it:
   - the `EXPECTED` block (chainId, mode, currentNextId, expectedCircuitId,
     circuit key, NAND burn, Circuits address, creator);
   - every visible label (circuit name, ID, burn, lede text);
   - the confirmation code (contract address chars 3-6) in BOTH the
     visible hint and the `send()` check.
4. Validate: NANDPU root must stay pointed at `0x8633…`; TapeOut files
   must stay pointed at `0xb102…`.
5. `git add -A && git commit && git push` — Pages publishes automatically.
6. `curl` the live URL and manifest; confirm the new circuit is shown.
7. Give the owner the matching URL from the table above, with `?v=<key>`
   for cache-bust on the NANDPU root.

## After the NANDPU receipt

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

- NANDPU default: one transaction per window, never batch (D-011). The
  current NANDPU root is an explicit multi-tx window; TapeOut mining is
  also multi-tx, on a **different** processor.
- The page is public by design during the window (Binance Wallet
  requirement); it is closed immediately after the receipt.
- No secrets live here: the manifest is an unsigned transaction.
- Never paste a club password, private key, or mnemonic into this repo.
