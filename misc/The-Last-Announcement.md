# The Last Announcement

**Category:** Misc  

---

## Challenge description

> An Indian Railways announcement recording appears to have been tampered with. Something seems off, but the announcement itself offers no obvious answers.
>
> Investigate the recording and uncover what someone tried to hide.

## Initial Thoughts & Inspection

We were given an audio file called `Train_Announcement.wav`.

When played, it sounds like a normal railway station announcement in Hindi ("Yatri kripiya dihyan de...").

However, there is a strange, robotic, high-pitched ringing noise in the background.

But as we know, in ctfs, unusual background sounds can indicate hidden information. Two possibilities worth investigating are:

1. Morse code hidden in the audio.
2. Audio spectrogram steganography, where text or images are hidden in specific sound frequencies.

## Finding the Flag

To inspect the audio frequencies visually, I used Audacity and changed the track view from Waveform to Spectrogram.

Another option is to generate a spectrogram using Python and Matplotlib.

```python
import scipy.io.wavfile as wav
import matplotlib.pyplot as plt

rate, data = wav.read("Train_Announcement.wav")

plt.figure(figsize=(15, 5))
plt.specgram(data[:, 0], Fs=rate, NFFT=1024, noverlap=512, cmap='inferno')
plt.savefig("spectrogram.png")
```

After generating the spectrogram, large glowing letters appeared in the higher frequencies, between approximately 5,000 Hz and 20,000 Hz.

The letters spelled out the flag.

## Flag

`InIt{h1d1ng_f1l3s_1n_4ud10}`

## Key Takeaway

Audio files can contain hidden information that is not easily noticeable when listening normally.

A spectrogram helps visualize the frequencies present in an audio recording, making it possible to discover hidden text, patterns or images embedded in the sound.
