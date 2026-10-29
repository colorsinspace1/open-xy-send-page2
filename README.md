# Open-XY SDK patch: two-page send effects, and Dry on send B

For the Open-XY SDK at commit `c10d61b` (v0.0.2), OP–XY OS 1.2.1. Written while
building the Grainfield send effect and tested on one OP–XY. Offered to the SDK
author for review; MIT, like Grainfield. By Colors In Space.

`open-xy-send-effect-page2.patch` changes four files:
`sdk/platform/opxy-1.2.1/synth_controls.cpp`, `fx_object.cpp.in`,
`screen_hooks.s` and `tooling/adapters.py`. It adds no new firmware edit
sites: everything goes through hooks the profile already reviews.

## What it does

1. **Page 2 for send effects with five to eight controls.** Holding encoder 1
   and turning it switches the send page between slots 0–3 and 4–7. Turns on
   Page 2 go through stock's own turn (`8d5ad5bc`) with the slot index 4–7, so
   stock stores, steps and publishes them to the audio vector as usual.
2. **Stepping for slots 4–7.** Stock's step-count query (`8c0b2468`) has no
   entries for a send's slots 4–7, so continuous ones did not move. The hook
   answers "continuous" (-1) for an owned send effect's slots 4 and up.
3. **Page 2 values and bars.** The send page draws its value texts and then
   its bars from the vector at `[P5 + 4]` (`8d53b5d8`–`8d53b9f4`). The existing
   value-text hook (`8d53b636`) returns a stand-in vector holding the stored
   slot 4–7 values; the stub puts it in P5 for the rest of that draw, so both
   the numbers and the bars show Page 2. Stock's own vector is never written.
4. **Dry on send B.** The mixer queries dry levels from two call sites,
   `8d510d5e` (send A) and `8d510df4` (send B), and applies send B's answer as
   1 - value. The adapter's dry query now passes its return address to
   `sdk_fx_dry`, which inverts a declared Dry only when called from send B's
   site. Machines without `dryParameter` are unchanged.

## What we found on the hardware, in case it helps

- Values written from GUI-side hook code into the SDK's own statics did not
  reach the audio callback. Routing Page 2 through stock's parameter path
  (point 1) is what worked.
- Comparing the dry query's parameter pointer with a lookup of track 15's did
  not identify send B; the call-site return address does.
- Writing the GUI vector's words 0–3 directly flickered: stock rewrites them.
