# 🔊 Weird Sound

| | |
|---|---|
| **Category** | Misc |
| **Points** | 100 |
| **Flag** | `InIt{W0ah_Y0u_S33_M3!}` |

## 📝 Challenge Analysis

We get an audio file (`audio.wav`) full of buzzing, screeching and frequency shifts. In CTFs, audio with artificial frequency patterns is almost always **spectrogram steganography**.

## 🛠️ Solution

### 1. Spectrogram inspection
- Open the file in **Audacity** or **Sonic Visualiser**, or plot it in Python with `scipy.io.wavfile` and `matplotlib.pyplot.specgram`.
- In Audacity: track dropdown menu → switch the view from **Waveform** to **Spectrogram**.

### 2. Reading the hidden frequencies
Between **3.7 kHz and 10.4 kHz**, the high-intensity tones form a 7-row matrix that looks like a **5×7 bitmap font**. Reading the glyphs left to right spells:

```text
W 0 a h _ Y 0 u _ S 3 3 _ M 3 !
```

### 3. Character clarification
The `S` stays a capital `S`, and the vowels are standard leetspeak (`o → 0`, `e → 3`). Wrapped in the flag format:

```text
InIt{W0ah_Y0u_S33_M3!}
```

## 🏁 Flag

```text
InIt{W0ah_Y0u_S33_M3!}
```

## 🧰 Tools
Audacity · Sonic Visualiser · Python (scipy, matplotlib)
