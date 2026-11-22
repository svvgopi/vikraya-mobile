# Restaurant Billing App Implementation Plan

[PRD v1.0](PRD.md) defines the product scope. Use sample data until the restaurant supplies its real menu and receipt details. Each feature below has a prerequisite and an observable acceptance check. Use fixtures or a simulated printer to verify features before connecting them.

| ID | Feature | Needs | Acceptance check |
| --- | --- | --- | --- |
| F01 | Flutter project and Android emulator | Windows Flutter and Android tooling | Launch the app on an Android emulator and run analysis and tests. |
| F02 | Menu catalog and editor | F01 | Create categories and items, change prices, hide an item, and restart. Edits persist and hidden items disappear from the order menu. |
| F03 | Order and totals | F01, sample menu fixture | Add an item twice, change quantities, and remove items. Verify totals against fixed examples using integer minor currency units. An empty order cannot proceed. |
| F04 | Finalize and save bills | F03 | Save an order offline with its own bill number and a snapshot of item names and prices. Repeated checkout taps produce one bill. Restart and verify the saved bill. |
| F05 | Receipt layout and preview | F01, finalized bill fixture | Preview a 58 mm receipt with restaurant details, bill number, timestamp, items, and totals. Long names wrap and totals stay legible. Payment method appears when supplied. |
| F06 | Simulated printing and recovery | F04, F05 | **Save & Print** saves before sending. Simulate success, a known failure, and a mid-job disconnect. Retry the saved bill without creating a second sale or automatically printing another copy. |
| F07 | Android PSF-588 direct printing | F01, F05, real phone and printer | Select and remember the printer, then print a sample receipt offline. Test consecutive receipts, a printer power cycle, and legibility on paper. Verify RawBT separately as a fallback. |
| F08 | Bill history and reprints | F04, F05, F06 for print verification | Use the simulated printer to verify that saved bills open after restart, retain the original prices after a menu edit, and reprint with the original number and **COPY**. |
| F09 | Backup and restore | F02, F04 | Restore an exported backup into a clean installation. Verify menu data, saved bills, and the next bill number. A failed restore preserves existing data. |
| F10 | Android integration and pilot | F02–F09 and confirmed receipt requirements | Use the real menu and printer for one shift alongside existing billing. Reconcile bill count and totals and repeat the disconnect, retry, and reprint scenarios on hardware. |
| F11 | iPhone printer compatibility | F01, F05, macOS with Xcode, real iPhone and PSF-588 | Print the same sample offline, check consecutive prints and reconnection, and record the result. Run when hardware is available. A passing result qualifies iPhone printing for integration and verification with the shared billing flow. |

## Build order

Start with F01. Menu editing, order calculations, and receipt preview can then move independently using fixtures. Start F07 as soon as F05 can produce a sample and the hardware is available. Continue bill saving, simulated printing, history, and backup while hardware is unavailable.

## Feature implementation

### F01 Flutter project and Android emulator

1. Check which Flutter SDK, Android SDK, and emulator tools are installed on Windows. Record the SDK versions and commands used to run the app.
2. Scaffold the Flutter app with Android as the first runnable target. Organize code by feature and keep the app entry point limited to wiring dependencies and navigation.
3. Verify a cold launch on the emulator, `flutter analyze`, and `flutter test`. Keep the scaffold usable by every later feature.

### F02 Menu catalog and editor

1. Define categories and menu items with stable IDs. Store prices as integer minor units and give each item an availability flag.
2. Implement local catalog reads and writes. Share the same persistent database with F04 so F09 can export a consistent snapshot. Confirm the database choice supports transactions and migrations before adopting it.
3. Build category browsing and editing in settings. Validate names and prices and keep hidden items out of new orders.
4. Verify persistence after restart, invalid input handling, and the visible catalog after a price or availability change.

### F03 Order and totals

1. Build the order from a sample catalog first. Capture the selected item's ID, display name, unit price, and quantity in the draft.
2. Add quantity controls and calculate line totals and the bill total from the draft using integer arithmetic. Connect the real catalog after F02.
3. Verify totals with fixed expected amounts, repeated additions, removal of the last item, and a disabled checkout for an empty order.

### F04 Finalize and save bills

1. Define the finalized bill record with an internal ID, display number, timestamp, immutable line snapshots, totals, and optional payment method. Keep one checkout identifier per draft through retries.
2. Save the bill and allocate its number in one local transaction. Enforce unique checkout identifiers and bill numbers in storage. A retry of a committed checkout returns the existing bill.
3. Keep the draft available until the save succeeds. Provide the saved bill to receipt rendering and print orchestration after the commit.
4. Verify duplicate submissions, a failed write followed by retry, sequential bill numbers, and reopening a bill after restart.

### F05 Receipt layout and preview

1. Use a finalized bill fixture and sample restaurant details to build a shared receipt layout. Add settings to edit the shop details from the PRD. Capture receipt details with finalized bills so later settings changes preserve the original receipt.
2. Render the layout as an on-screen 58 mm preview. Expose the same receipt content to the print implementation and support a **COPY** marker and optional payment method.
3. Verify long names, aligned amounts, timestamps, and required fields. Confirm the printer's actual printable width and character support in F07. Apply confirmed tax fields before F10.

### F06 Simulated printing and recovery

1. Define a print interface that accepts a prepared receipt and reports completed transmission, failure before sending, or uncertain output. Implement a simulator with deterministic examples of all three outcomes.
2. Wire **Save & Print** to F04, then F05, then the print interface. Guard repeated taps while a save or print is in progress. Show the result and allow a manual retry using the existing bill.
3. Persist the latest attempt status. Treat an interrupted in-progress attempt as uncertain after restart. Verify that saving errors prevent printing and that retries leave the bill count and total unchanged.

### F07 Android PSF-588 direct printing

1. Build a standalone test-print entry point that uses the F05 sample receipt. Confirm the physical unit's Bluetooth transport and print commands through device discovery and a real sample print before committing to a library or driver.
2. Implement Android permissions, printer selection, remembered selection, and the verified delivery path. Send jobs sequentially and report interrupted jobs using the F06 outcome rules when integrating.
3. Test permission denial, printer off or out of range, consecutive offline prints, and recovery after a power cycle. Record the phone and printer tested and inspect the paper output.
4. Verify the vendor's RawBT workflow separately. Exercise it as a manual fallback where direct delivery is unavailable. Keep the same bill number and treat an uncertain direct attempt as requiring staff review before retrying.

### F08 Bill history and reprints

1. Read finalized bills from local storage and show their bill numbers, timestamps, and totals. Add a detail screen with the saved receipt and latest available print status.
2. Route the reprint action through the print interface with a **COPY** marker. Use the saved snapshot and original bill number.
3. Verify a reprint after restart and after menu or restaurant detail edits. Compare bill count and sales totals before and after reprinting.

### F09 Backup and restore

1. Export a versioned local backup containing the catalog, restaurant settings, bills, and numbering state. Read these from a consistent database snapshot.
2. Validate the file before writing. Restore into a clean installation in one transaction and recover the next bill number from the restored state. For the first release, reject restoration over existing sales data.
3. Verify a round trip with bills, hidden items, and settings. Check that invalid files and failed restores preserve existing data and that the next checkout gets a unique bill number.

### F10 Android integration and pilot

1. Connect the real menu, saved bills, printer settings, history, and backup through the app's normal screens. Confirm the restaurant's receipt requirements and use its actual details, menu, and currency.
2. Run the full workflow on the counter phone offline. Repeat printer disconnect, manual retry, reprint, app restart, and backup restore with test data.
3. Pilot alongside existing billing for one shift. Compare finalized bill counts and totals. Record discrepancies and resolve them before relying on the app for live billing.

### F11 iPhone printer compatibility and integration

1. On macOS with Xcode, build the Flutter app for a physical iPhone. Test discovery and direct delivery of the F05 sample on the PSF-588. Record the OS, observed transport, and result.
2. If the hardware check passes, implement printer selection and reconnect behavior using the same print interface as Android. Connect it to checkout, retry, and reprint.
3. Verify offline consecutive prints, reconnection, receipt layout on paper, and bill preservation through failures. Record iPhone support as pending until these checks pass.

## Rules for each feature

- Start from the PRD and the feature’s acceptance check. Keep unrelated scope out of the change.
- Use settled interfaces between features. The order supplies quantities and prices, the saved bill holds a snapshot, the receipt renderer reads that snapshot, and printing sends the formatted receipt. A printer retry always uses an existing bill.
- Record the checks actually run and the observed results. Mark emulator and simulated results separately from physical printer results.
- Verify unfamiliar framework or printer behavior against official documentation or a real device. Record unverified behavior as pending. Flag any product decision that would change the PRD before implementing it.
