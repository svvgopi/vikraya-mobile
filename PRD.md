# Restaurant Billing App Product Requirements

**Version 1.0 · Locked for implementation · 22 November 2026**

The app lets counter staff select menu items, finalize a bill offline, and print a 58 mm receipt on the restaurant’s Posiflow PSF-588 handheld printer. Version 1 serves one restaurant with one billing phone and one printer. Android is the first release. iPhone printing depends on a successful test with the same printer model on a physical iPhone.

## Product requirements

| Area | Requirement |
| --- | --- |
| Menu | Staff can browse items by category and tap to add them to a bill. The owner can add and edit categories, item names, and prices, and hide unavailable items. |
| New bill | Staff can change quantities, remove items, and see the total update immediately. An empty bill cannot be finalized. |
| Checkout | Staff can review the bill and tap **Save & Print**. Recording the payment method is optional. The app assigns a unique bill number on the device and saves the bill before attempting to print. Repeated taps during checkout finalize the bill once. |
| Receipt | The 58 mm layout shows the restaurant details, bill number, date and time, item names, quantities, unit prices, line totals, and final total. Include payment method when entered and any fields required for the restaurant’s confirmed receipt and tax rules. |
| Printer settings | Staff can select the PSF-588, retain the selection across restarts, and run a test print. Direct Bluetooth printing is the checkout goal. RawBT is the documented Android fallback. |
| Print recovery | A failed or interrupted print leaves the saved bill available in history. Staff can retry the same bill. When physical output cannot be confirmed, show **Sent to printer** if transmission completed and an explicit uncertain status if the connection dropped mid-job. Leave repeat printing to staff. |
| Bill history | Staff can open saved bills and print a **COPY** with the original bill number. Saved bills retain their original items, prices, and totals after menu changes. |
| Offline operation and backup | Billing, history, and direct printing work without internet. Provide a backup and verify restoration of menu data, finalized bills, and bill numbering before the app becomes the restaurant’s only sales record. |

## Delivery and acceptance

1. **Flutter prototype on Windows.** Run the app in the Android emulator with a simulated printer and a 58 mm preview. Demonstrate selecting and editing items, correct totals, saving a bill, surviving an app restart, and recovering from a simulated print failure without duplicating the bill.
2. **Physical Android print.** Once the app can generate one sample receipt, test it on the counter phone and the PSF-588. Verify direct printing with internet off, several consecutive receipts, and reconnection after a printer power cycle. Check alignment, long names, totals, and required currency or local-language characters on paper. Run this milestone before polishing the full app.
3. **Android pilot and release.** Run one shift alongside the current billing process. Reconcile the shift’s finalized bill count and total against the existing records. Verify that a saved bill survives a printer disconnect, that a reprint uses its original number, and that a restored backup preserves bill numbering. Confirm the receipt and tax fields before live use.
4. **iPhone compatibility.** Reuse the Flutter billing screens and receipt layout. Build and test with macOS and Xcode. Release iPhone printing after a real iPhone and the PSF-588 pass the same offline printing, alignment, and reconnection checks.

## Launch inputs

Use sample menu items, prices, and restaurant details during development. Before live use, enter the real menu and restaurant details and confirm the currency, receipt language, and applicable receipt and tax requirements. Validate any required local-language text on the printer.

## Later scope

Multi-device billing and sync, kitchen tickets, table management, inventory, online ordering, and payment processing are outside version 1.

## Hardware and platform references

- [PSF-588 Android setup and RawBT test print](https://www.shreyanspos.com/pages/driver-for-mobile-printer-srs588-srs-583)
- [Flutter iOS development setup](https://docs.flutter.dev/platform-integration/ios/setup)
