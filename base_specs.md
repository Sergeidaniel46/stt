Yes, Whisper Base English INT8 should work on common Windows computers and modern Android devices. It is a reasonable middle ground, but slower and more memory-intensive than Tiny.

The official sherpa-onnx project provides exported `base.en` models and supports both Flutter and Android. It does not publish a strict minimum hardware specification, so the hardware numbers below are practical deployment guidance rather than official requirements. [Sherpa-onnx Whisper models](https://k2-fsa.github.io/sherpa/onnx/pretrained_models/whisper/export-onnx.html)

## Practical specifications

| Platform | Minimum practical | Recommended |
|---|---|---|
| Windows OS | Windows 10, 64-bit | Windows 10/11, 64-bit |
| Windows CPU | Modern dual-core x64 | 4+ physical cores, Intel Core i5/Ryzen 5 or newer |
| Windows RAM | 4 GB | 8 GB+ |
| Android OS | Android 7/API 24 | Android 10+ |
| Android CPU | 64-bit ARM, 4 cores | ARM64, 6–8 cores, Cortex-A73/A76 class or newer |
| Android RAM | 3 GB | 4–6 GB+ |
| Free storage | At least 250 MB | 400 MB+ for model, native libraries and temporary installation space |

Your project explicitly requires Android API 24 and packages:

- Debug: `arm64-v8a` and `x86_64`
- Release: `arm64-v8a`

Therefore, the release APK will not run on 32-bit-only Android devices.

## Expected behavior

The installed Base INT8 files total approximately 161 MB:

```text
encoder.onnx     29.1 MB
decoder.onnx    130.7 MB
tokens.txt        0.8 MB
```

Runtime memory will be higher than the file size because ONNX Runtime allocates model, audio-feature, decoder and working buffers. Allowing several hundred megabytes of available RAM is sensible.

Approximate practical expectations:

| Device class | Expected result |
|---|---|
| Recent Windows desktop/laptop | Should work comfortably |
| Older dual-core Windows PC | Works, but transcription may be noticeably slower |
| Modern mid-range ARM64 Android | Should work for short recordings |
| High-end Android | Good expected performance |
| Low-end 3 GB Android | May be slow; memory pressure is possible |
| x86_64 Android emulator | Works in principle, but often much slower than a physical ARM64 phone |
| 32-bit Android | Not supported by this project’s release build |

## Important limitation

“Works” does not necessarily mean real-time. Whisper Base is an offline batch recognizer:

```text
recording ends → complete audio decoded → result returned
```

For a 3–10 second utterance, this is usually acceptable. Long recordings can become slow, particularly on emulators or low-end phones. Your code also imposes a 60-second inference timeout.

For deployment, I would set:

- Maximum recording duration: 10–15 seconds
- Android target: ARM64, Android 10+, 4 GB RAM
- Windows target: 64-bit Windows 10+, 8 GB RAM
- Keep Tiny available as a fallback for low-end Android hardware

The final proof still requires measuring on representative devices. Log the existing `duration`, inference time, `peak`, and `rms`; a good acceptance target is that inference finishes faster than or close to the recording duration.