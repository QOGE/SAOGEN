# Symbiont Wallet GUI Sidebar and Theme Implementation Report

Prepared for: Claude Sonnet, project manager  
Date: 2026-09-02  
Repository: `/home/ion/symbiont-wallet`  
Branch: `main`

## Executive summary

The existing Fyne GUI was restyled and reorganized around a permanent left-hand navigation sidebar, a QOGE-branded dark theme, and a wider application canvas. This was an aesthetic and layout pass over existing wallet capabilities; it did not add wallet, signing, transaction, address-decoding, or RPC protocol behavior.

The implementation is pushed to `origin/main`. The current local and remote HEAD is:

`c6221ec548a3c4d3f89fa96044d1a872eff11cc1`

## Landed commits

### 1. `37e6e8ce9fec8bb3e03a1163c4a3b8c0e415d32d`

`feat: refresh GUI layout and apply QOGE theme`

Scope:

- Added the custom QOGE theme in `cmd/gui/theme.go` and installed it immediately after application creation with `a.Settings().SetTheme(NewQogeTheme())`.
- Added and embedded Space Grotesk Regular for application-wide typography.
- Added the SIL Open Font License alongside the font asset.
- Increased the initial window size from `620x640` to `1040x840` after iterative layout review.
- Replaced the visible top-tab navigation with a 144-pixel left sidebar containing Wallet, My Addresses, Send, and Network.
- Kept the existing internal `AppTabs` as the authoritative selection/gating model so existing enable/disable behavior and tab tests remain applicable.
- Wallet and Network remain available before a wallet is loaded. My Addresses and Send remain disabled until a wallet opens or is created.
- Sidebar buttons show a magenta background only for the active page; inactive buttons use the low-importance/transparent treatment.
- Sidebar button corner radius is scoped to 3 pixels without changing ordinary application-button radii.
- Added icons to sidebar entries and widened the sidebar so icon and label fit on one row.
- Made ordinary action buttons compact instead of full-width and centered them where appropriate, while preserving the deliberate Wallet-tab alignment requested during visual review.
- Applied increased horizontal content padding without adding padding to the sidebar.
- Changed the application content and footer background to `#0e0e1a` for a continuous corporate dark surface.
- Added a branded lifecycle palette for FRESH, FUNDED, SPEND_PENDING, SPENT, and RETIRED address chips.
- Reworked My Addresses rows into a structured ledger with index, lifecycle chip, address, balance, and an icon-only copy action.
- Added summary cards for spendable balance, pending balance, and total address count.
- Converted My Addresses to a border layout so the scrollable address list expands into available vertical space. Its minimum height remains a small floor rather than a fixed target.
- Replaced the My Addresses text Refresh button with a compact refresh icon placed alongside its status row.
- Simplified the address status prompt to `Press Refresh to see your addresses.` because the page is inaccessible until a wallet is loaded.
- In Send, reduced the amount field width and placed Preview Transaction on the same row.
- Replaced the Send text refresh control with a real icon button wired to the existing dropdown-population handler.
- Positioned the Send refresh icon beside the upper From dropdown. The lower wallet-address dropdown reserves matching blank width for visual symmetry without presenting a second action.
- Enlarged the transaction-preview scroll area to keep long destination addresses easier to inspect.
- Preserved the existing danger styling of Broadcast Transaction.
- Reorganized Wallet controls through several visual iterations, including deliberate spacing between Open Existing Wallet and Create New Wallet and row placement for creation/backup controls.

Files in the commit:

- `cmd/gui/main.go`
- `cmd/gui/theme.go`
- `cmd/gui/fonts/SpaceGrotesk-Regular.ttf`
- `cmd/gui/fonts/OFL.txt`

Git summary: 537 insertions, 50 deletions across four files.

### 2. `c6221ec548a3c4d3f89fa96044d1a872eff11cc1`

`chore: remove redundant Generate New Address display from My Addresses`

The follow-up removed the My Addresses `Generate New` control, its dedicated display field, and its separate copy action. `NextReceiveAddress()` only returned the current front address from the already-generated pool, so the UI suggested generation that was not actually occurring. Any visible FRESH address can receive funds, and every address row already has a working copy icon.

Effects:

- The address ledger now occupies the reclaimed area.
- The concentration warning remains unchanged.
- Per-row copy controls remain the receive-address copy mechanism.
- The Send flow's independent use of `NextReceiveAddress()` for change-address selection remains intact.
- No wallet logic was changed.

File in the commit:

- `cmd/gui/main.go`

Git summary: 1 insertion, 30 deletions.

## Theme implementation details

The theme is fixed-dark and uses the QOGE palette. Important implementation points include:

- Main background and surface: `#0e0e1a`
- Elevated control surface: `#24243a`
- Primary accent: `#d400ff`
- Near-black foreground on the magenta accent for contrast
- Separate semantic colors for lifecycle states and danger actions
- Space Grotesk Regular returned for every Fyne text style, giving the application consistent normal-weight typography
- Default Fyne icons retained through `theme.DefaultTheme()`
- Default theme fallback retained for color and size names not explicitly overridden

## Architecture and behavior preserved

This work changes presentation and navigation wiring only. It does not alter:

- Wallet creation/opening or seed authentication
- Five-state address lifecycle rules
- Balance or confirmation scanning
- P2QPK signing
- Transaction construction or serialization
- Destination address validation
- Mempool testing or broadcast RPC behavior
- Local-cookie or manual RPC authentication

The custom sidebar selects the same four existing `TabItem` instances. Existing wallet-loaded gating is mirrored onto the sidebar buttons, preventing navigation into wallet-dependent pages before a wallet is loaded.

## Verification performed

Before the layout/theme commit, the following completed successfully:

- `git diff --check`
- `go vet ./cmd/gui`
- `go build ./cmd/gui`
- `go test ./...`

After the redundant address-display removal, both of the following completed successfully:

- `go test ./cmd/gui/...`
- `go test ./...`

The visual layout was also iteratively reviewed in the running application by the user. Automated tests establish that existing behavior did not regress, but they do not comprehensively validate theme appearance, pixel spacing, or container rendering.

## Current repository state

- Branch: `main`
- Local HEAD: `c6221ec548a3c4d3f89fa96044d1a872eff11cc1`
- `origin/main`: `c6221ec548a3c4d3f89fa96044d1a872eff11cc1`
- The original HTML layout mockup was intentionally deleted after it had served as the visual reference.
- This report is local and uncommitted unless the project manager later decides it belongs in repository history.

## Suggested review focus

For project-management review, the most useful checks are:

1. Verify navigation and disabled-state behavior before and after wallet loading.
2. Inspect the four pages at the default `1040x840` size and at smaller manually resized dimensions.
3. Confirm long addresses and status messages remain readable without colliding with refresh/copy icons.
4. Confirm the active sidebar item is visually unambiguous and inactive entries remain quiet.
5. Confirm the Broadcast Transaction action retains its distinct danger styling.
6. Decide whether this local implementation report should remain out of Git or be added as project documentation.
