# Acoustic OFDM Transceiver

A full OFDM link in MATLAB, transmitted and received over a real acoustic channel with a
laptop speaker and microphone. QPSK on the subcarriers, IFFT/FFT with a cyclic prefix,
preamble-based frame synchronisation, pilot-based channel estimation and equalisation, and
two-microphone receive diversity. It carries arbitrary data end to end: a 128x128 image was
transmitted and reconstructed with 0% bit error.

## Chain

**Transmitter** — QPSK mapping, training insertion, serial-to-parallel, IFFT, cyclic prefix,
resampling to the sound card rate, upconversion to carrier, concatenated behind a shaped
preamble.

**Receiver** — downconversion, lowpass, frame synchronisation, resampling, cyclic-prefix
removal, FFT, channel equalisation, training removal, demapping.

**Frame synchronisation** — the received signal passes an RRC matched filter, then a sliding
window correlates it against the locally generated LFSR preamble. The correlation is
normalised by the window energy, `T = |c|^2/E`, so the detection threshold does not depend
on received level, and the frame start is taken at the peak.

## Channel estimation

Two methods, and the comparison is the interesting part.

**Block**: one training symbol before the data, `H = Y/X`, estimated once per frame. Cheap,
but only valid while the channel holds still.

**Comb**: one pilot every four data subcarriers (L = 5), with linear interpolation across the
rest. Tracks a channel that changes within a frame.

On a channel with carrier frequency offset, block estimation stays clean for about 8 OFDM
symbols and then the constellation rotates apart: BER 0.10 by symbol 10. Adding
Viterbi-Viterbi phase tracking extends that to roughly 30 symbols (BER 0.0022), still
degrading by 40 (0.0612). Comb estimation handles the same channel at **BER 0.0003**,
against 0.1555 for block plus Viterbi.

## Results

Sensitivity: BER stays under 1% down to **SNR = 0 dB**, and breaks above it at -1 dB.

Throughput against cyclic-prefix length, at 16 kHz bandwidth:

| Cyclic prefix | Throughput | BER |
|---|---|---|
| 128 | 25 600 bps | 6.76% |
| 32 | 30 118 bps | 9.95% |
| 0 | 32 000 bps | 13.78% |

Shortening the CP buys throughput and pays for it in inter-symbol interference, which is the
whole trade-off in one table.

Receive diversity by Maximum Ratio Combining across a laptop microphone and a USB
microphone: BER 3.04% and 2.69% individually, **0.97% combined**.

Over real acoustic channels, an indoor corridor with strong echo produced a much longer
power delay profile than the emulated channels, while outdoors the delay profile was short
and the transmission ran at **0% BER**.

## Running it

`audiotrans.m` for the standard link, `imagetrans.m` for the image demo,
`diversity_trans.m` for the two-microphone version, `audiotrans_throughput.m` for the
bandwidth and CP sweeps.

## Files

```
audiotrans.m                 - Main script for standard OFDM audio transmission and channel simulation.
audiotrans_throughput.m      - Main script testing the maximum throughput and BER across different bandwidth and CP.
channel_emulator.p           - Given file for the emulation of the channel.
channel_tracking.m           - Performs channel estimation and equalization using Block or Comb methods.
channel_tracking_diversity.m - Implements Maximum Ratio Combining (MRC) for diversity reception.
comb_training.m              - Inserts pilot tones into OFDM symbols for Comb-based channel estimation.
demapper.m                   - Demaps complex QPSK symbols into a binary bit stream.
diversity_trans.m            - Main script for diversity transmission using two microphones (the one of the PC and the USB).
frame_sync.m                 - Detects the start of the data frame using matched filtering using preamble.
image.jpg                    - Alternative source image file used for the image transmission demonstration.
image2.jpg                   - Source image file used for the image transmission demonstration.
image_decoder.m              - Decodes a received bit stream into an image and displays it.
imagetrans.m                 - Main script for transmitting and reconstructing an image file.
matched_filter.m             - Filters the signal with a root-raised cosine pulse for synchronization.
ofdm_rx_resampling.m         - Downsamples the received signal from the audio card rate to the OFDM baseband rate. (Given)
ofdm_tx_resampling.m         - Upsamples the OFDM baseband signal to the audio card sampling rate for transmission. (Given)
ofdmlowpass.m                - Applies a lowpass filter. (Given)
preamble_generate.m          - Generates a pseudo-noise sequence for frame synchronization using lfsr.
QPSK_mapping.m               - Maps binary data bits to complex QPSK constellation symbols.
remove_comb.m                - Removes pilot tones from the received symbol matrix to extract data when using Comb.
rrc.m                        - Generates Root Raised Cosine filter coefficients for pulse shaping. (Given)
rxofdm.m                     - Standard receiver function performing sync, FFT, equalization, and demapping.
rxofdm_diversity.m           - Diversity receiver function combining signals from two sources using MRC.
training_generate.m          - Generates the pseudo-random sequence used for pilot symbols.
txofdm.m                     - Standard transmitter function generating the full OFDM signal structure.
```

---

EPFL Wireless Receivers, two-person project with Matteo Barberis. Files marked "(Given)"
above were provided by the course.
