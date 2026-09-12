# Architecture

## Dependency flow

`feature → domain ← data`, while `media → renderer → output Surface`. `output/backend` depends only on the output contract and never leaks device-specific behavior into media or scenes.

## Requested boundaries

| Boundary | Responsibility |
|---|---|
| `app` | Process entry points and DI composition |
| `core/common`, `core/ui`, `core/device`, `core/permissions`, `core/logging` | Cross-cutting primitives, design system, capability checks, permission and diagnostics policy |
| `domain/model`, `domain/repository`, `domain/usecase` | Stable business contracts |
| `data/database`, `data/datastore`, `data/repository` | Room, preferences and repository adapters |
| `media/image`, `media/gif`, `media/video`, `media/camera` | Frame producers behind `FrameSource` |
| `renderer/gl`, `renderer/shader`, `renderer/texture`, `renderer/transform`, `renderer/scheduler` | Persistent zero-copy render graph and pacing |
| `scene/engine`, `scene/model`, `scene/persistence` | Scene switching, representation and storage |
| `output/engine`, `output/service`, `output/backend` | State machine, foreground lifetime and pluggable system sink |
| `feature/studio`, `feature/scenes`, `feature/media`, `feature/settings`, `feature/onboarding` | Compose screens and immutable presentation state |

## Frame lifecycle

1. A coordinator prepares a replacement `FrameSource` against either an external texture or one-time texture-upload input.
2. After preparation succeeds, `SceneEngine.commit` atomically exposes the new source and releases the previous source.
3. `PreviewRenderer` remains attached to its GLSurfaceView/EGL context.
4. The shader applies orientation metadata, crop/scale, position, mirroring, rotation and later overlay passes.
5. The same renderer can target preview and a future backend-provided output surface.

## Backend contract

`VirtualCameraBackend` has support detection, observable state and suspendable start/stop. `UnsupportedBackend` is the only default binding. A future privileged, OEM, ROM or platform implementation belongs in its own build variant/module and must accept rendered frames through this interface.

## Next stabilization work

- Move source preparation onto a renderer-owned command queue and add EGL shared-context output rendering.
- Add native GIF texture upload/compositing (the decoder timing contract is already isolated).
- Persist the complete media catalog, scene transforms and playback config using typed Room converters.
- Add benchmark tests on low-, mid- and high-tier hardware for every advertised preset.
- Add instrumentation tests for process death, URI revocation, camera lifecycle and foreground-service restrictions.
