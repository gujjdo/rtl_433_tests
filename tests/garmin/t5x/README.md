# Garmin T5x

Garmin T5x dog-tracking collar (EU variant), uplink on 155.450 MHz.

## Description

The collar transmits a 68 ms burst every 2.5 s. The modulation is 2-FSK with
±3 kHz tones around the channel center, 1500 baud NRZ. Each burst is about 26
alternating preamble symbols followed by a single 70-bit frame.

The frequency deviation is small relative to the discriminator range of the
default demodulator settings. The samples need `-Y minmax` and a narrowed FM
filter (`-Y filter=180`, about 5.6 kHz cutoff); both are in the `demod` file
of each set and are passed automatically by the test runner.

## Signal

- Frequency: 155.450 MHz
- Modulation: FSK, tones at ±3 kHz
- Baud: 1500 (667 us per symbol)
- Burst: 2.5 ms unmodulated carrier, ~26 alternating sync symbols, 70 data
  symbols (67.95 ms total)
- Period: one burst every 2.5 s

## Row format

The frame boundary is tone-invisible: the last preamble symbol equals the
first ID bit, so the frame starts at the first symbol pair that breaks the
preamble alternation.

    IIIIIIII IIIIIIII NNNNNNNN NNNNNNNN EEEEEEEE EEEEEEEE EEEEEEEE TTTTTTT S CCCCCCCC

- I: 14 bit collar id
- N: 16 bit northing window, low 16 bits of the latitude in Garmin
  semicircles (sc = 180/2^31 degrees)
- E: 24 bit easting window, low 24 bits of the longitude in semicircles
- T: 7 bit sequence tag, cycling A (0x41), B (0x00), C (0x1b) with each
  transmission; rarely D (0x11)
- S: 1 bit status, meaning not established
- C: 8 bit checksum, GF(2)-linear over the preceding 62 bits; D-tag frames
  carry a constant 0xb6 offset

Position is windowed: latitude = (N * 2^8 + k * 2^24) sc and
longitude = (E * 2^3 + m * 2^27) sc, with the wrap counts k, m resolved from
a nearby reference position (e.g. the receiver's own GPS fix). The windows
wrap every 1.40625 degrees of latitude and 11.25 degrees of longitude.

## Decoding

The checksum is a GF(2)-linear code (not a CRC): each of the 8 checksum bits
is the parity of a fixed subset of the 62 payload bits. The eight 62-bit row
masks are in the decoder source. The code detects all single-bit errors; the
sequence-tag whitelist catches most random rows.

## Samples

All samples are from one collar (id 0x2996, on-air id 10646) recorded in
Sweden in August-September 2026.

- `01`: strong clean signal, collar at rest, A/B/C cycle
- `02`: includes the rare D sequence tag (21.3 s into the source capture)
- `03`: collar under tree cover, weaker signal
- `04`: collar moving at vehicle range
- `05`: continuous raw recording (not gate-saved)

Manufacturer page: https://www.garmin.com/en-US/p/798107
