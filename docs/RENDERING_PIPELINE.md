# Rendering pipeline contract

`FrameSource → GPU texture → orientation correction → crop → scale → position → mirror → rotation → overlays → output Surface`

`FrameSource` owns decoding/capture and playback control. It does not know whether a frame is previewed or sent to a system backend. `PreviewRenderer` owns GL resources and never asks a source for a per-frame bitmap. External producers signal `SurfaceTexture.OnFrameAvailableListener`; still content uploads once.

GIF source timestamps remain authoritative. Output at 30/60 FPS repeats the current GIF frame until its native delay expires rather than modifying GIF timing. Video uses Media3 hardware decode when available. CameraX uses Camera2 and provides a Surface directly.

All GL creation, replacement and destruction should occur on the GL command queue. The current milestone provides the contracts and preview renderer; shared EGL output and overlay passes are intentionally the next renderer milestone before any platform backend work.
