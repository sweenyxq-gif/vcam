# VCam Studio

VCam Studio is an Android 10+ broadcast-style media controller. It normalizes still images, animated GIFs, Media3 video and CameraX frames into one persistent OpenGL ES rendering pipeline. The preview app is deliberately useful without a system virtual-camera backend.

## Current milestone

- Premium dark Material 3 shell with Studio, Scenes, Media and Settings navigation
- Immutable `StateFlow` UI and sealed actions/errors
- SAF picker with persisted read permissions
- Shared `FrameSource` contract with image, GIF, Media3 and CameraX implementations
- Persistent GLES preview renderer with GPU textures and transform matrix
- Scene persistence (Room), user defaults (DataStore), capability presets and performance models
- Strict `IDLE → PREPARING → READY → STARTING → STREAMING → STOPPING` output machine
- Foreground service and ongoing resolution/FPS notification
- Isolated `VirtualCameraBackend` with safe `UnsupportedBackend`

The system camera-injection backend is intentionally not implemented. Add one as a separate implementation of `VirtualCameraBackend`; no media, renderer, scene or UI API should depend on its mechanism.

## Build

Open the root folder in Android Studio (JDK 17, Android SDK 35) and sync. Run the `app` configuration on Android 10/API 29 or newer.

## Source layout

The requested boundaries are package-level modules in the initial deployable app module. This avoids dozens of almost-empty Gradle modules during pipeline stabilization. Every boundary is dependency-oriented and can be extracted into an Android/Kotlin Gradle module later without changing public interfaces. See `docs/ARCHITECTURE.md`.

## Performance rules

- The renderer survives source changes; only its input texture is rebound.
- Images decode and upload once. CPU pixel memory is released after upload.
- Video and camera decode directly to `SurfaceTexture`/`Surface`.
- GIF timing belongs to the decoder; the render scheduler samples the current frame at output cadence.
- No frame path creates `Bitmap` objects.
- UI state is immutable, and output lifecycle has one state enum rather than competing flags.
