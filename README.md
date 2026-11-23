# Keyword Recorder (Android)

A native Android app (Kotlin, not a web app) that records audio and transcribes it live. It saves every recording as an **MP3** plus a **Word (.docx)** transcript. **The first word you say is the keyword.** Recordings are grouped and sorted by that keyword, and it names their folders and files.

## What it does

| Feature | Details |
|---|---|
| Recording buttons | **Record**, **Pause / Resume** and **Stop**, with a live timer and input-level meter |
| Live transcript | Text appears as you speak, using the phone's own speech engine (on-device when available) |
| Keyword | The first word of the transcript, e.g. *"Groceries, milk eggs bread…"* → **Groceries**. You can edit it before saving |
| Review | After Stop you can correct the transcript and keyword, then **Save** or **Discard** |
| MP3 | 64 kbps mono MP3, encoded on the phone with LAME |
| Word document | Keyword as the title, then the date, length and full transcript |
| Downloads | Both files are copied to `Downloads/KeywordRecorder/<Keyword>/<Keyword>_<date>.mp3` / `.docx` |
| Library | Recordings grouped under keyword headings (A–Z, newest first), each with **Play**, **Word** (open), **Share** and **Delete** |
| Player controls | Play/Pause, seek bar, back 10 s, forward 10 s, speed 0.75× / 1× / 1.5× / 2× |
| Background recording | A foreground service with a "Recording…" notification keeps it going with the screen off |

## Permissions

The app asks for everything it needs on first launch, and again when you tap Record if anything is missing:

* **Microphone** (`RECORD_AUDIO`): required to record.
* **Notifications** (`POST_NOTIFICATIONS`, Android 13+): shows the "Recording…" notification.
* `FOREGROUND_SERVICE` and `FOREGROUND_SERVICE_MICROPHONE` are granted automatically and keep recording alive in the background.

If you permanently deny the microphone, the app explains why it needs it and offers an **Open Settings** button. Saving to Downloads uses Android's MediaStore, so no storage permission is needed.

## Requirements

* Android 10 or newer to install and record.
* Android 13 or newer for live transcription. On older phones you can type the transcript or keyword after stopping. The MP3 and Word files are still produced.
* A speech-recognition service on the phone. Google's is built into almost all phones. If the offline speech model for your language isn't downloaded yet, the app asks the phone to download it and uses the standard online recognizer in the meantime.

## Building the APK

### Option A: Android Studio (on your computer)
1. Install [Android Studio](https://developer.android.com/studio).
2. **File → Open** and choose this `KeywordRecorder` folder. Let Gradle sync. Accept any prompt to install the **NDK** and **CMake**.
3. Plug in your phone with USB debugging on and press **Run ▶**, *or* use **Build → Build App Bundle(s) / APK(s) → Build APK(s)** and copy `app/build/outputs/apk/…/app-*.apk` to the phone.

### Option B: no install, build on GitHub
1. Create a new GitHub repository and upload this folder's contents.
2. GitHub Actions runs `.github/workflows/build.yml` automatically. Open the **Actions** tab, then the finished run, and download **KeywordRecorder-apk**.
3. Copy the APK to your phone and open it. Allow "install unknown apps" if Android asks.

### Notes
* On its first build, the project downloads the LAME 3.100 MP3-encoder source from Debian (with SourceForge as a fallback) and compiles it. If that download is blocked, get `lame-3.100.tar.gz` yourself and extract it into `app/src/main/cpp/` so that `app/src/main/cpp/lame-3.100/libmp3lame/lame.c` exists.
* The release APK is signed with the debug key so it can be sideloaded straight away. Use your own keystore before publishing to Google Play.
* LAME is LGPL-licensed.

## Project layout

```
app/src/main/
  AndroidManifest.xml            permissions, foreground service, file sharing
  cpp/CMakeLists.txt, lame_jni.c LAME MP3 encoder + JNI bridge
  java/com/voicekeyword/recorder/
    MainActivity.kt              UI, permission requests, library, player
    RecordingEngine.kt           microphone capture → MP3 encoder + transcriber
    Transcriber.kt               live speech-to-text fed from the same audio
    Mp3Encoder.kt                Kotlin wrapper around LAME
    DocxWriter.java              writes the .docx (no external libraries)
    RecordingRepository.kt       storage, keyword folders, Downloads export, sorting
    RecordingService.kt          foreground "Recording…" service
    RecordingsAdapter.kt         keyword-grouped list
```
