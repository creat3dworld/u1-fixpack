# Draft comment for PR #869 (DO NOT POST — awaiting Marlena's go-ahead)

Re-verified this on the 2.1.2 develop build (develop at 8d8af0d, stock base 2.1.2.159). The bug is still there as built, and the patch still fixes it.

I ran the skip-pause-resume cases in Klipper batch mode with the shipped printer.cfg and fluidd.cfg, on u1-klipper 7ac453b plus the overlay's patches 01–08 as the 2.1.2 build has them — the overlay hasn't changed since the November 5 check. Without patch 10, all four cases fail: pause-then-resume stops with "Move out of range on Y axis: 33.000 448.500 5.800", the end-of-print park plus E-only move stops with "Move out of range on Y axis: 35.000 450.500 1.200", and the park-inside-skipped-object and tool-change cases move the head where only E or Z should move — the tool-change prime lands 10 mm off in X. With patch 10 applied, all four pass and the prints complete.

One thing worth knowing for anyone applying the patch file from this PR by hand: the file as stored in the overlay is skipped silently by `git apply` outside a repo — exit 0, nothing applied. The build itself uses `patch -F 0`, which applies it fine, and the same code as a plain git patch works too; that's how I ran the patched tree.

The triage, the patch and these batch-mode runs were done by Muse, the AI assistant, working under Marlena and Breandan's direction — Breandan directed the AI step by step, tested the results, and built the platform for live testing. A run on our physical U1 follows once it's on 2.1.2, and I'll report back here with that result.

Full logs: docs/evidence/extended-2.1.2/ in the fixpack repo.
