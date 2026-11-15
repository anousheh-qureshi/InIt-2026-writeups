# Beepy

| | |
|---|---|
| **Category** | Misc |
| **Points** | 100 |
| **Author** | Abdullah Hamdan |
| **Artifact** | `audio.wav` |
| **Flag** | `InIt{B33PY_M0RS3_1S_C00L}` |

## Challenge

> What is this weird beep beeeeeep sound?
>
> Wrap the answer in `InIt{A11_CAP5}`

## TL;DR

The beeps are Morse code. Measure the tone on/off durations, decode them, and wrap the all-caps result in `InIt{}`.

## Solution

### 1. Look at the audio

No audio tools were needed beyond Python's `wave` module. The file is a tone that switches on and off, so the message is in the timing, not the pitch.

I computed a short-window RMS envelope (5 ms windows), thresholded it at 30% of the peak, and turned it into a list of on/off run lengths:

```python
import wave, struct, math

w = wave.open('audio.wav')
sr, n, ch = w.getframerate(), w.getnframes(), w.getnchannels()
s = struct.unpack('<%dh' % (n * ch), w.readframes(n))[::ch]

win = int(sr * 0.005)                       # 5 ms windows
env = [math.sqrt(sum(x*x for x in s[i:i+win]) / win)
       for i in range(0, len(s) - win, win)]
on = [e > max(env) * 0.3 for e in env]      # tone present?

runs, cur, c = [], on[0], 0                 # (state, duration in ms)
for o in on:
    if o == cur: c += 1
    else: runs.append((cur, c * 5)); cur, c = o, 1
runs.append((cur, c * 5))
```

### 2. Work out the timing

The durations fall into clean groups, so the unit is easy to read off:

| Duration | Meaning |
|---|---|
| ~60 ms tone | dot (`.`) |
| ~180 ms tone | dash (`-`) |
| ~60 ms silence | gap between elements of a letter |
| ~180 ms silence | gap between letters |
| longer silence (300+ ms) | gap between words |

### 3. Decode

Map each tone run to `.` or `-` (under or over 120 ms), split letters on the long gaps, and look each group up in a Morse table:

```text
B33PY?M0RS3?1S?C00L
```

The `?` marks are the sequence `..--.-`, which is the **underscore** in extended Morse. The challenge asks for all caps, so the result is:

```text
B33PY_M0RS3_1S_C00L
```

## Flag

```
InIt{B33PY_M0RS3_1S_C00L}
```

## Takeaways

- "Beeps" in an audio file usually means Morse code. Look at the on/off timing rather than the frequency.
- Standard Morse tables have no underscore. If a decoder gives unknown groups, check extended Morse (`..--.-` is `_`).
