# Windows HLSL compatibility

This is the finalized static and runtime-qualified compatibility ledger for Kodi RetroPlayer on Windows. The static audit maps renderer blockers and fidelity limitations across the complete HLSL preset tree; the exposed Windows catalog was then qualified in Kodi for compilation, resources, switching, and visual output on the audited runtime described below.

- Audited libretro commit: `524835ca394e5fd1bf459c5090222d7159f5a5dc`
- HLSL presets audited: 599
- Audit scope: every `libretro/hlsl/**/*.cgp` preset and every shader `#include` reachable from its `shaderN` entries

## Methodology

The audit parses every preset assignment for shader passes, LUT declarations and sampler fields, aliases, feedback/history, per-pass filter and wrap controls, mipmap inputs, and float/sRGB framebuffer requests. Shader paths are resolved relative to each preset. Includes are then followed recursively relative to the including source, with cycles deduplicated. All report paths are normalized relative to `libretro/`.

Comment-free transitive source is scanned for exact, case-sensitive RetroArch resource identifiers: `ORIG`, `PASSPREV` (optionally numbered), numbered `PASS`, and `PREV` (optionally numbered), plus `COMPAT_*` macro use. Identifier boundaries exclude arbitrary longer names such as `ORIG_LINEARIZED`. A finite allowlist separately recognizes the resource accessors declared by `compat_orig_struct.inc` and `compat_prev_struct.inc` (for example, `ORIG_Sample`, `ORIG_texture`, `PASSPREV_Sample`, and `PASSPREV_texture`); declaration-only `INITIALIZE_*` macros do not create a blocker. Those generic include declarations are recorded as sources but excluded from resource-use detection; only a shader's actual use creates a blocker.

A non-empty pass alias is a hard blocker only when its exact identifier is referenced by the comment-free transitive source of a later shader pass; empty and unreferenced alias declarations do not require alias binding. Any frame history/feedback, original or non-immediate pass resource, or downstream reference to a declared pass alias is a hard blocker because Kodi D3D binds only the current pass input (`decal`) plus parameters and LUTs. Filter/wrap controls, intermediate mipmaps, and LUT wrap requests are fidelity limitations, not static rejections; affected candidates remain eligible for visual runtime qualification. A preset can appear in every applicable group below.

## Renderer capability matrix

| Preset/shader feature | Kodi Windows D3D status | Static treatment |
| --- | --- | --- |
| HLSL compiled through FX11 | Supported | Eligible |
| Current input (`decal`) | Supported | Eligible |
| Parameters and basic LUT binding | Supported | Eligible |
| Relative shader includes | Supported | Eligible |
| Sequential multipass rendering | Supported | Eligible when each pass needs only its immediate input |
| Float and sRGB framebuffers | Supported | Eligible |
| Source and viewport scaling | Supported | Eligible |
| Absolute scaling on an intermediate pass | Supported | Eligible |
| Explicit absolute FBO scaling on the final pass | Unsupported; the requested fixed output size is discarded | Hard runtime blocker for affected border presets |
| Frame count | Supported | Eligible |
| Frame history, feedback, or `PREV` resources | Unsupported | Hard blocker |
| `ORIG`, `PASSPREV`, or arbitrary numbered `PASS` resources | Unsupported | Hard blocker |
| Downstream references to declared pass aliases | Unsupported | Hard blocker |
| Preset-controlled pass filter or wrap mode | Not faithfully applied | Runtime fidelity qualification |
| Intermediate mipmaps | Not faithfully supplied | Runtime fidelity qualification |
| LUT wrap mode | Not faithfully applied | Runtime fidelity qualification |

## Runtime blocker: explicit absolute final-pass FBO scaling

5 presets.

Kodi's Windows D3D path supports fixed-size intermediate render targets, but it does not preserve an explicit absolute size when that scaled pass is the final preset pass. The post-UV1 SGB retest made the border visible but still produced a tiny, streaked game image because the requested 256x224 final-pass FBO size was discarded. Implementing final-pass FBO scaling would make these five presets eligible for runtime qualification:

- `hlsl/borders/gameboy-player/gameboy-player.cgp`
- `hlsl/borders/gameboy-player/gameboy-player-gba-color.cgp`
- `hlsl/borders/sgb/sgb.cgp`
- `hlsl/borders/sgba/sgba.cgp`
- `hlsl/borders/sgba/sgba-gba-color.cgp`

## Hard blocker: frame history or feedback

76 presets.

- `hlsl/cgp/gameboy-screen-grid+motionblur.cgp`
- `hlsl/cgp/lowquality-lcd+motionblur.cgp`
- `hlsl/crt/phosphor-trails.cgp`
- `hlsl/handheld/console-border/dmg-2x.cgp`
- `hlsl/handheld/console-border/dmg-3x.cgp`
- `hlsl/handheld/console-border/dmg-4x.cgp`
- `hlsl/handheld/console-border/dmg-5x.cgp`
- `hlsl/handheld/console-border/dmg-6x.cgp`
- `hlsl/handheld/console-border/gb-dmg-alt-2x.cgp`
- `hlsl/handheld/console-border/gb-dmg-alt-3x.cgp`
- `hlsl/handheld/console-border/gb-dmg-alt-4x.cgp`
- `hlsl/handheld/console-border/gb-dmg-alt-5x.cgp`
- `hlsl/handheld/console-border/gb-dmg-alt-6x.cgp`
- `hlsl/handheld/console-border/gb-dmg-alt-7x.cgp`
- `hlsl/handheld/console-border/gb-light-alt-2x.cgp`
- `hlsl/handheld/console-border/gb-light-alt-3x.cgp`
- `hlsl/handheld/console-border/gb-light-alt-4x.cgp`
- `hlsl/handheld/console-border/gb-light-alt-5x.cgp`
- `hlsl/handheld/console-border/gb-light-alt-6x.cgp`
- `hlsl/handheld/console-border/gb-light-alt-7x.cgp`
- `hlsl/handheld/console-border/gb-pocket-2x.cgp`
- `hlsl/handheld/console-border/gb-pocket-3x.cgp`
- `hlsl/handheld/console-border/gb-pocket-4x.cgp`
- `hlsl/handheld/console-border/gb-pocket-5x.cgp`
- `hlsl/handheld/console-border/gb-pocket-6x.cgp`
- `hlsl/handheld/console-border/gb-pocket-alt-2x.cgp`
- `hlsl/handheld/console-border/gb-pocket-alt-3x.cgp`
- `hlsl/handheld/console-border/gb-pocket-alt-4x.cgp`
- `hlsl/handheld/console-border/gb-pocket-alt-5x.cgp`
- `hlsl/handheld/console-border/gb-pocket-alt-6x.cgp`
- `hlsl/handheld/console-border/gb-pocket-alt-7x.cgp`
- `hlsl/handheld/console-border/gba-lcd-grid-v2-2x.cgp`
- `hlsl/handheld/console-border/gba-lcd-grid-v2-3x.cgp`
- `hlsl/handheld/console-border/gba-lcd-grid-v2-4x.cgp`
- `hlsl/handheld/console-border/gba-lcd-grid-v2-5x.cgp`
- `hlsl/handheld/console-border/gba-lcd-grid-v2-6x.cgp`
- `hlsl/handheld/console-border/gba-retro-v2-2x.cgp`
- `hlsl/handheld/console-border/gba-retro-v2-3x.cgp`
- `hlsl/handheld/console-border/gba-retro-v2-4x.cgp`
- `hlsl/handheld/console-border/gba-retro-v2-5x.cgp`
- `hlsl/handheld/console-border/gba-retro-v2-6x.cgp`
- `hlsl/handheld/console-border/gbc-lcd-grid-v2-2x.cgp`
- `hlsl/handheld/console-border/gbc-lcd-grid-v2-3x.cgp`
- `hlsl/handheld/console-border/gbc-lcd-grid-v2-4x.cgp`
- `hlsl/handheld/console-border/gbc-lcd-grid-v2-5x.cgp`
- `hlsl/handheld/console-border/gbc-lcd-grid-v2-6x.cgp`
- `hlsl/handheld/console-border/gbc-lcd-grid-v2-7x.cgp`
- `hlsl/handheld/console-border/gbc-retro-v2-2x.cgp`
- `hlsl/handheld/console-border/gbc-retro-v2-3x.cgp`
- `hlsl/handheld/console-border/gbc-retro-v2-4x.cgp`
- `hlsl/handheld/console-border/gbc-retro-v2-5x.cgp`
- `hlsl/handheld/console-border/gbc-retro-v2-6x.cgp`
- `hlsl/handheld/console-border/gbc-retro-v2-7x.cgp`
- `hlsl/handheld/gb-light-shader.cgp`
- `hlsl/handheld/gb-pocket-shader.cgp`
- `hlsl/handheld/gb-shader.cgp`
- `hlsl/handheld/lcd-cgwg.cgp`
- `hlsl/handheld/lcd-grid-v2-gba-color-motionblur.cgp`
- `hlsl/handheld/lcd-grid-v2-gbc-color-motionblur.cgp`
- `hlsl/handheld/lcd-grid-v2-motionblur.cgp`
- `hlsl/handheld/lcd-grid-v2-nds-color-motionblur.cgp`
- `hlsl/handheld/lcd-grid-v2-palm-color-motionblur.cgp`
- `hlsl/handheld/lcd-grid-v2-psp-color-motionblur.cgp`
- `hlsl/handheld/lcd-grid-v2-vba-color-motionblur.cgp`
- `hlsl/handheld/lcd-shader-gba-color.cgp`
- `hlsl/handheld/lcd-shader-gbc-color.cgp`
- `hlsl/handheld/lcd-shader-nds-color.cgp`
- `hlsl/handheld/lcd-shader-palm-color.cgp`
- `hlsl/handheld/lcd-shader-psp-color.cgp`
- `hlsl/handheld/lcd-shader-vba-color.cgp`
- `hlsl/handheld/lcd-shader.cgp`
- `hlsl/motionblur/braid-rewind.cgp`
- `hlsl/motionblur/motionblur-blue.cgp`
- `hlsl/motionblur/motionblur-color.cgp`
- `hlsl/motionblur/motionblur-simple.cgp`
- `hlsl/motionblur/response-time.cgp`

## Hard blocker: original or non-immediate pass resources

118 presets.

- `hlsl/borders/bigblur.cgp`
- `hlsl/borders/sgba/sgba-gba-color+crt-easymode-halation.cgp`
- `hlsl/cgp/tvout/tvout+mdapt.cgp`
- `hlsl/cgp/tvout+interlacing/tvout+mdapt+interlacing.cgp`
- `hlsl/cgp/xsoft+scalefx-level2aa.cgp`
- `hlsl/cgp/xsoft+scalefx-level2aa+sharpsmoother.cgp`
- `hlsl/crt/crt-easymode-halation.cgp`
- `hlsl/crt/crt-hyllian-glow.cgp`
- `hlsl/crt/crt-hyllian-multipass.cgp`
- `hlsl/crt/crt-interlaced-halation.cgp`
- `hlsl/crt/crt-lottes-multipass.cgp`
- `hlsl/crt/crt-super-xbr.cgp`
- `hlsl/crt/crtglow_gauss_ntsc_3phase.cgp`
- `hlsl/crt/crtglow_gauss.cgp`
- `hlsl/crt/crtglow_lanczos.cgp`
- `hlsl/crt/ntsc-shadowmask-interlacing.cgp`
- `hlsl/crt/ntsc-shadowmask-noblend.cgp`
- `hlsl/crt/ntsc-shadowmask.cgp`
- `hlsl/crt/phosphorlut-4k.cgp`
- `hlsl/crt/phosphorlut-shadowmask.cgp`
- `hlsl/crt/shaders/phosphorlut-v3/phosphorlut-4k-noblend.cgp`
- `hlsl/crt/shaders/phosphorlut-v3/phosphorlut-aperturemask-interlacing-noblend.cgp`
- `hlsl/crt/shaders/phosphorlut-v3/phosphorlut-aperturemask-interlacing.cgp`
- `hlsl/crt/shaders/phosphorlut-v3/phosphorlut-aperturemask-noblend.cgp`
- `hlsl/crt/shaders/phosphorlut-v3/phosphorlut-aperturemask.cgp`
- `hlsl/crt/shaders/phosphorlut-v3/phosphorlut-shadowmask-interlacing-noblend.cgp`
- `hlsl/crt/shaders/phosphorlut-v3/phosphorlut-shadowmask-interlacing.cgp`
- `hlsl/crt/shaders/phosphorlut-v3/phosphorlut-shadowmask-noblend.cgp`
- `hlsl/denoisers/crt-fast-bilateral-super-xbr.cgp`
- `hlsl/denoisers/fast-bilateral-super-2xbr-3d-3p.cgp`
- `hlsl/denoisers/fast-bilateral-super-xbr-4p.cgp`
- `hlsl/denoisers/fast-bilateral-super-xbr-6p.cgp`
- `hlsl/denoisers/fast-bilateral-super-xbr-natural-vision.cgp`
- `hlsl/denoisers/fast-bilateral-super-xbr.cgp`
- `hlsl/dithering/mdapt.cgp`
- `hlsl/dithering/mdapt+xbr-hybrid+aa.cgp`
- `hlsl/dithering/mdapt+xbr-hybrid+ddt.cgp`
- `hlsl/handheld/console-border/dmg-2x.cgp`
- `hlsl/handheld/console-border/dmg-3x.cgp`
- `hlsl/handheld/console-border/dmg-4x.cgp`
- `hlsl/handheld/console-border/dmg-5x.cgp`
- `hlsl/handheld/console-border/dmg-6x.cgp`
- `hlsl/handheld/console-border/gb-dmg-alt-2x.cgp`
- `hlsl/handheld/console-border/gb-dmg-alt-3x.cgp`
- `hlsl/handheld/console-border/gb-dmg-alt-4x.cgp`
- `hlsl/handheld/console-border/gb-dmg-alt-5x.cgp`
- `hlsl/handheld/console-border/gb-dmg-alt-6x.cgp`
- `hlsl/handheld/console-border/gb-dmg-alt-7x.cgp`
- `hlsl/handheld/console-border/gb-light-alt-2x.cgp`
- `hlsl/handheld/console-border/gb-light-alt-3x.cgp`
- `hlsl/handheld/console-border/gb-light-alt-4x.cgp`
- `hlsl/handheld/console-border/gb-light-alt-5x.cgp`
- `hlsl/handheld/console-border/gb-light-alt-6x.cgp`
- `hlsl/handheld/console-border/gb-light-alt-7x.cgp`
- `hlsl/handheld/console-border/gb-pocket-2x.cgp`
- `hlsl/handheld/console-border/gb-pocket-3x.cgp`
- `hlsl/handheld/console-border/gb-pocket-4x.cgp`
- `hlsl/handheld/console-border/gb-pocket-5x.cgp`
- `hlsl/handheld/console-border/gb-pocket-6x.cgp`
- `hlsl/handheld/console-border/gb-pocket-alt-2x.cgp`
- `hlsl/handheld/console-border/gb-pocket-alt-3x.cgp`
- `hlsl/handheld/console-border/gb-pocket-alt-4x.cgp`
- `hlsl/handheld/console-border/gb-pocket-alt-5x.cgp`
- `hlsl/handheld/console-border/gb-pocket-alt-6x.cgp`
- `hlsl/handheld/console-border/gb-pocket-alt-7x.cgp`
- `hlsl/handheld/gb-light-shader.cgp`
- `hlsl/handheld/gb-pocket-shader.cgp`
- `hlsl/handheld/gb-shader.cgp`
- `hlsl/handheld/lcd-shader-gba-color.cgp`
- `hlsl/handheld/lcd-shader-gbc-color.cgp`
- `hlsl/handheld/lcd-shader-nds-color.cgp`
- `hlsl/handheld/lcd-shader-palm-color.cgp`
- `hlsl/handheld/lcd-shader-psp-color.cgp`
- `hlsl/handheld/lcd-shader-vba-color.cgp`
- `hlsl/handheld/lcd-shader.cgp`
- `hlsl/hqx/hq2x.cgp`
- `hlsl/hqx/hq3x.cgp`
- `hlsl/hqx/hq4x.cgp`
- `hlsl/ntsc/artifact-colors.cgp`
- `hlsl/scalefx/scalefx-hybrid.cgp`
- `hlsl/scalefx/scalefx.cgp`
- `hlsl/scalefx/scalefx+rAA.cgp`
- `hlsl/scalefx/shaders/old/scalefx_hybrid.cgp`
- `hlsl/scalefx/shaders/old/scalefx.cgp`
- `hlsl/scalefx/shaders/old/scalefx9.cgp`
- `hlsl/scalefx/shaders/old/xsoft+scalefx_hybrid.cgp`
- `hlsl/scalefx/shaders/old/xsoft+scalefx.cgp`
- `hlsl/scalefx/shaders/old/xsoft+scalefx+sharpsmoother.cgp`
- `hlsl/scalefx/shaders/old/xsofter+scalefx_hybrid.cgp`
- `hlsl/scalefx/shaders/old/xsofter+scalefx.cgp`
- `hlsl/sharpen/super-xbr-super-res.cgp`
- `hlsl/xbr/shaders/xbr-lv2-multipass/xbr-lv2-a.cgp`
- `hlsl/xbr/shaders/xbr-lv2-multipass/xbr-lv2-b.cgp`
- `hlsl/xbr/shaders/xbr-lv2-multipass/xbr-lv2-c.cgp`
- `hlsl/xbr/shaders/xbr-lv2-multipass/xbr-lv2-d.cgp`
- `hlsl/xbr/shaders/xbr-lv2-multipass/xbr-lv2-dilation-a.cgp`
- `hlsl/xbr/shaders/xbr-lv2-multipass/xbr-lv2-dilation-b.cgp`
- `hlsl/xbr/shaders/xbr-lv2-multipass/xbr-lv2-dilation-c.cgp`
- `hlsl/xbr/shaders/xbr-lv2-multipass/xbr-lv2-dilation-d.cgp`
- `hlsl/xbr/shaders/xbr-lv2-multipass/xbr-lv2-noblend-dilation-c.cgp`
- `hlsl/xbr/super-2xbr-3d-2p.cgp`
- `hlsl/xbr/super-2xbr-3d-3p-smoother.cgp`
- `hlsl/xbr/super-4xbr-3d-4p.cgp`
- `hlsl/xbr/super-4xbr-3d-6p-smoother.cgp`
- `hlsl/xbr/super-8xbr-3d-6p.cgp`
- `hlsl/xbr/super-xbr-2p.cgp`
- `hlsl/xbr/super-xbr-3p-smoother.cgp`
- `hlsl/xbr/super-xbr-6p-adaptive.cgp`
- `hlsl/xbr/super-xbr-6p-small-details.cgp`
- `hlsl/xbr/super-xbr-6p.cgp`
- `hlsl/xbr/super-xbr-deposterize.cgp`
- `hlsl/xbr/xbr-lv2-accuracy-multipass.cgp`
- `hlsl/xbr/xbr-lv2-accuracy-smart-blur.cgp`
- `hlsl/xbr/xbr-lv2-multipass.cgp`
- `hlsl/xbr/xbr-lv3-multipass.cgp`
- `hlsl/xbr/xbr-mlv4-dilation.cgp`
- `hlsl/xbr/xbr-mlv4-multipass.cgp`
- `hlsl/xbrz/xbrz-freescale-multipass.cgp`

## Hard blocker: pass aliases

15 presets.

- `hlsl/borders/gameboy-player/gameboy-player+crt-royale.cgp`
- `hlsl/borders/sgb/sgb+crt-royale.cgp`
- `hlsl/cgp/crt-royale-kurozumi.cgp`
- `hlsl/crt/crt-lottes-multipass.cgp`
- `hlsl/crt/crt-royale-fake-bloom-intel.cgp`
- `hlsl/crt/crt-royale-fake-bloom.cgp`
- `hlsl/crt/crt-royale-gdapt.cgp`
- `hlsl/crt/crt-royale-intel.cgp`
- `hlsl/crt/crt-royale-ntsc-256px-composite.cgp`
- `hlsl/crt/crt-royale-ntsc-256px-svideo.cgp`
- `hlsl/crt/crt-royale-ntsc-320px-composite.cgp`
- `hlsl/crt/crt-royale-ntsc-320px-svideo.cgp`
- `hlsl/crt/crt-royale-pal-r57shell.cgp`
- `hlsl/crt/crt-royale-tvout+image-adjustment.cgp`
- `hlsl/crt/crt-royale.cgp`

## Fidelity limitation: preset-controlled filter or wrap mode

565 presets. This group is not automatically rejected; it requires visual runtime qualification.

- `hlsl/anti-aliasing/aa-shader-4.o-level2.cgp`
- `hlsl/anti-aliasing/aa-shader-4.o.cgp`
- `hlsl/anti-aliasing/advanced-aa.cgp`
- `hlsl/anti-aliasing/fx-aa.cgp`
- `hlsl/anti-aliasing/fxaa-edge-detect.cgp`
- `hlsl/anti-aliasing/reverse-aa.cgp`
- `hlsl/bicubic/bicubic-fast.cgp`
- `hlsl/bicubic/bicubic-normal.cgp`
- `hlsl/bicubic/bicubic-sharp.cgp`
- `hlsl/bicubic/bicubic-sharper.cgp`
- `hlsl/bilinear.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur10x10shared-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur10x10shared-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur10x10shared-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur10x10shared-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur11fast-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur11fast-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur11fast-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur11fast-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur11resize-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur11resize-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur11resize-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur11resize-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur12x12shared-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur12x12shared-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur12x12shared-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur12x12shared-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3fast-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3fast-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3fast-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3fast-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3resize-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3resize-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3resize-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3resize-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3x3-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3x3-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3x3-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3x3-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3x3resize-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3x3resize-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3x3resize-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3x3resize-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur5fast-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur5fast-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur5fast-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur5fast-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur5resize-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur5resize-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur5resize-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur5resize-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur5x5-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur5x5-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur5x5-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur5x5-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur6x6shared-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur6x6shared-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur6x6shared-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur6x6shared-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur7fast-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur7fast-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur7fast-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur7fast-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur7resize-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur7resize-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur7resize-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur7resize-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur7x7-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur7x7-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur7x7-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur7x7-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur8x8shared-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur8x8shared-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur8x8shared-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur8x8shared-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur9fast-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur9fast-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur9fast-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur9fast-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur9resize-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur9resize-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur9resize-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur9resize-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur9x9-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur9x9-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur9x9-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur9x9-srgb-six-times.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur10x10shared-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur10x10shared-mipmap-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur10x10shared-mipmap-srgb.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur10x10shared-srgb.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur12x12shared-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur12x12shared-mipmap-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur12x12shared-mipmap-srgb.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur12x12shared-srgb.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur6x6shared-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur6x6shared-mipmap-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur6x6shared-mipmap-srgb.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur6x6shared-srgb.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur8x8shared-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur8x8shared-mipmap-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur8x8shared-mipmap-srgb.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur8x8shared-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur10x10shared-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur10x10shared-mipmap-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur10x10shared-mipmap-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur10x10shared-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur11fast-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur11fast-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur11resize-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur11resize-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur12x12shared-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur12x12shared-mipmap-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur12x12shared-mipmap-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur12x12shared-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur3fast-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur3fast-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur3resize-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur3resize-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur3x3-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur3x3-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur3x3resize-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur3x3resize-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur5fast-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur5fast-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur5resize-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur5resize-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur5x5-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur5x5-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur6x6shared-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur6x6shared-mipmap-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur6x6shared-mipmap-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur6x6shared-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur7fast-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur7fast-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur7resize-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur7resize-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur7x7-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur7x7-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur8x8shared-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur8x8shared-mipmap-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur8x8shared-mipmap-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur8x8shared-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur9fast-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur9fast-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur9resize-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur9resize-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur9x9-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/quality-test-presets/test-blur9x9-srgb.cgp`
- `hlsl/borders/bigblur.cgp`
- `hlsl/borders/gameboy-player/gameboy-player+crt-royale.cgp`
- `hlsl/borders/sgb/sgb+crt-royale.cgp`
- `hlsl/borders/sgba/sgba-gba-color.cgp`
- `hlsl/borders/sgba/sgba-gba-color+crt-easymode-halation.cgp`
- `hlsl/cgp/2x2xscalehq.cgp`
- `hlsl/cgp/2xbr-crt-hyllian.cgp`
- `hlsl/cgp/2xbr-hybrid-crt-hyllian.cgp`
- `hlsl/cgp/2xbr-jinc2-sharper-hybrid.cgp`
- `hlsl/cgp/2xbr-reverse-aa.cgp`
- `hlsl/cgp/crt-reverse-aa-ddt.cgp`
- `hlsl/cgp/crt-royale-kurozumi.cgp`
- `hlsl/cgp/lowquality-lcd.cgp`
- `hlsl/cgp/n64-aa-crt.cgp`
- `hlsl/cgp/n64-vifilter/vifilter-480i.cgp`
- `hlsl/cgp/n64-vifilter/vifilter-angrylion-force240p.cgp`
- `hlsl/cgp/n64-vifilter/vifilter-crt.cgp`
- `hlsl/cgp/n64-vifilter/vifilter-scanline.cgp`
- `hlsl/cgp/n64-vifilter/vifilter.cgp`
- `hlsl/cgp/nes-color-decoder+colorimetry+pixellate.cgp`
- `hlsl/cgp/nes-color-decoder+pixellate.cgp`
- `hlsl/cgp/retro-v2+gba-color.cgp`
- `hlsl/cgp/retro-v2+gbc-color.cgp`
- `hlsl/cgp/retro-v2+image-adjustment.cgp`
- `hlsl/cgp/retro-v2+nds-color.cgp`
- `hlsl/cgp/retro-v2+psp-color.cgp`
- `hlsl/cgp/retro-v2+vba-color.cgp`
- `hlsl/cgp/simple2x.cgp`
- `hlsl/cgp/tvout/gtu-famicom-240p.cgp`
- `hlsl/cgp/tvout/gtuv50-240p.cgp`
- `hlsl/cgp/tvout/tvout-pixelsharp.cgp`
- `hlsl/cgp/tvout/tvout+cbod.cgp`
- `hlsl/cgp/tvout/tvout+gdapt.cgp`
- `hlsl/cgp/tvout/tvout+mdapt.cgp`
- `hlsl/cgp/tvout/tvout+nes-color-decoder.cgp`
- `hlsl/cgp/tvout/tvout+ntsc-256px-composite.cgp`
- `hlsl/cgp/tvout/tvout+ntsc-256px-svideo.cgp`
- `hlsl/cgp/tvout/tvout+ntsc-2phase-composite.cgp`
- `hlsl/cgp/tvout/tvout+ntsc-2phase-svideo.cgp`
- `hlsl/cgp/tvout/tvout+ntsc-320px-composite.cgp`
- `hlsl/cgp/tvout/tvout+ntsc-320px-svideo.cgp`
- `hlsl/cgp/tvout/tvout+ntsc-3phase-composite.cgp`
- `hlsl/cgp/tvout/tvout+ntsc-3phase-svideo.cgp`
- `hlsl/cgp/tvout/tvout+ntsc-nes.cgp`
- `hlsl/cgp/tvout/tvout+snes-hires-blend.cgp`
- `hlsl/cgp/tvout+interlacing/gtu-famicom-240p+interlacing.cgp`
- `hlsl/cgp/tvout+interlacing/gtuv50-240p+interlacing.cgp`
- `hlsl/cgp/tvout+interlacing/tvout+cbod+interlacing.cgp`
- `hlsl/cgp/tvout+interlacing/tvout+gdapt+interlacing.cgp`
- `hlsl/cgp/tvout+interlacing/tvout+mdapt+interlacing.cgp`
- `hlsl/cgp/tvout+interlacing/tvout+nes-color-decoder+interlacing.cgp`
- `hlsl/cgp/tvout+interlacing/tvout+ntsc-256px-composite+interlacing.cgp`
- `hlsl/cgp/tvout+interlacing/tvout+ntsc-256px-svideo+interlacing.cgp`
- `hlsl/cgp/tvout+interlacing/tvout+ntsc-2phase-composite+interlacing.cgp`
- `hlsl/cgp/tvout+interlacing/tvout+ntsc-2phase-svideo+interlacing.cgp`
- `hlsl/cgp/tvout+interlacing/tvout+ntsc-320px-composite+interlacing.cgp`
- `hlsl/cgp/tvout+interlacing/tvout+ntsc-320px-svideo+interlacing.cgp`
- `hlsl/cgp/tvout+interlacing/tvout+ntsc-3phase-composite+interlacing.cgp`
- `hlsl/cgp/tvout+interlacing/tvout+ntsc-3phase-svideo+interlacing.cgp`
- `hlsl/cgp/tvout+interlacing/tvout+ntsc-nes+interlacing.cgp`
- `hlsl/cgp/tvout+interlacing/tvout+snes-hires-blend+interlacing.cgp`
- `hlsl/cgp/xbr-dilation-smart-blur-4xsoft.cgp`
- `hlsl/cgp/xbr-dtt-dilation-soft2-aa-gamma.cgp`
- `hlsl/cgp/xbr-hybrid-bicubic.cgp`
- `hlsl/cgp/xbr-hybrid-ddt.cgp`
- `hlsl/cgp/xbr-hybrid-lanczos.cgp`
- `hlsl/cgp/xbr-hybrid-sharp-lanczos.cgp`
- `hlsl/cgp/xbr-smart-blur.cgp`
- `hlsl/cgp/xsoft+scalefx-level2aa.cgp`
- `hlsl/cgp/xsoft+scalefx-level2aa+sharpsmoother.cgp`
- `hlsl/crt/4xbr-hybrid-crt-b.cgp`
- `hlsl/crt/4xbr-hybrid-crt.cgp`
- `hlsl/crt/crt-aperture.cgp`
- `hlsl/crt/crt-caligari.cgp`
- `hlsl/crt/crt-cgwg-fast.cgp`
- `hlsl/crt/crt-easymode-halation.cgp`
- `hlsl/crt/crt-easymode.cgp`
- `hlsl/crt/crt-geom.cgp`
- `hlsl/crt/crt-hyllian-3d.cgp`
- `hlsl/crt/crt-hyllian-fast.cgp`
- `hlsl/crt/crt-hyllian-glow.cgp`
- `hlsl/crt/crt-hyllian-multipass.cgp`
- `hlsl/crt/crt-hyllian.cgp`
- `hlsl/crt/crt-interlaced-halation.cgp`
- `hlsl/crt/crt-lottes-fast.cgp`
- `hlsl/crt/crt-lottes-halation.cgp`
- `hlsl/crt/crt-lottes-multipass.cgp`
- `hlsl/crt/crt-lottes.cgp`
- `hlsl/crt/crt-nes-mini.cgp`
- `hlsl/crt/crt-reverse-aa.cgp`
- `hlsl/crt/crt-royale-fake-bloom-intel.cgp`
- `hlsl/crt/crt-royale-fake-bloom.cgp`
- `hlsl/crt/crt-royale-gdapt.cgp`
- `hlsl/crt/crt-royale-intel.cgp`
- `hlsl/crt/crt-royale-ntsc-256px-composite.cgp`
- `hlsl/crt/crt-royale-ntsc-256px-svideo.cgp`
- `hlsl/crt/crt-royale-ntsc-320px-composite.cgp`
- `hlsl/crt/crt-royale-ntsc-320px-svideo.cgp`
- `hlsl/crt/crt-royale-pal-r57shell.cgp`
- `hlsl/crt/crt-royale-tvout+image-adjustment.cgp`
- `hlsl/crt/crt-royale.cgp`
- `hlsl/crt/crt-super-xbr.cgp`
- `hlsl/crt/crtglow_gauss_ntsc_3phase.cgp`
- `hlsl/crt/crtglow_gauss.cgp`
- `hlsl/crt/crtglow_lanczos.cgp`
- `hlsl/crt/dotmask.cgp`
- `hlsl/crt/GritsScanlines.cgp`
- `hlsl/crt/GTU-famicom-radeon.cgp`
- `hlsl/crt/GTU-famicom.cgp`
- `hlsl/crt/gtuv50-radeon.cgp`
- `hlsl/crt/gtuv50.cgp`
- `hlsl/crt/ntsc-shadowmask-interlacing.cgp`
- `hlsl/crt/ntsc-shadowmask-noblend.cgp`
- `hlsl/crt/ntsc-shadowmask.cgp`
- `hlsl/crt/phosphor-trails.cgp`
- `hlsl/crt/phosphorlut-4k.cgp`
- `hlsl/crt/phosphorlut-shadowmask.cgp`
- `hlsl/crt/shaders/phosphor.cgp`
- `hlsl/crt/shaders/phosphorlut-v3/phosphorlut-4k-noblend.cgp`
- `hlsl/crt/shaders/phosphorlut-v3/phosphorlut-aperturemask-interlacing-noblend.cgp`
- `hlsl/crt/shaders/phosphorlut-v3/phosphorlut-aperturemask-interlacing.cgp`
- `hlsl/crt/shaders/phosphorlut-v3/phosphorlut-aperturemask-noblend.cgp`
- `hlsl/crt/shaders/phosphorlut-v3/phosphorlut-aperturemask.cgp`
- `hlsl/crt/shaders/phosphorlut-v3/phosphorlut-shadowmask-interlacing-noblend.cgp`
- `hlsl/crt/shaders/phosphorlut-v3/phosphorlut-shadowmask-interlacing.cgp`
- `hlsl/crt/shaders/phosphorlut-v3/phosphorlut-shadowmask-noblend.cgp`
- `hlsl/crt/snes-hires-blend.cgp`
- `hlsl/crt/tv-highcontrast-hd-1152x672.cgp`
- `hlsl/crt/tv-highcontrast-hd.cgp`
- `hlsl/crt/tvout-tweaks.cgp`
- `hlsl/crt/zfast-crt.cgp`
- `hlsl/ddt/ddt-extended.cgp`
- `hlsl/ddt/ddt-sharp.cgp`
- `hlsl/ddt/ddt-waterpaint.cgp`
- `hlsl/ddt/ddt.cgp`
- `hlsl/denoisers/crt-fast-bilateral-super-xbr.cgp`
- `hlsl/denoisers/fast-bilateral-super-2xbr-3d-3p.cgp`
- `hlsl/denoisers/fast-bilateral-super-xbr-4p.cgp`
- `hlsl/denoisers/fast-bilateral-super-xbr-6p.cgp`
- `hlsl/denoisers/fast-bilateral-super-xbr-natural-vision.cgp`
- `hlsl/denoisers/fast-bilateral-super-xbr.cgp`
- `hlsl/denoisers/fast-bilateral.cgp`
- `hlsl/dithering/cbod_v1.cgp`
- `hlsl/dithering/gdapt.cgp`
- `hlsl/dithering/gdapt+xbr-hybrid+aa.cgp`
- `hlsl/dithering/gdapt+xbr-hybrid+ddt.cgp`
- `hlsl/dithering/gendither.cgp`
- `hlsl/dithering/mdapt.cgp`
- `hlsl/dithering/mdapt+xbr-hybrid+aa.cgp`
- `hlsl/dithering/mdapt+xbr-hybrid+ddt.cgp`
- `hlsl/eagle/super-eagle.cgp`
- `hlsl/gpu/3dfx_4x1.cgp`
- `hlsl/gpu/shaders/old/3dfx_4x1.cgp`
- `hlsl/handheld/bevel.cgp`
- `hlsl/handheld/console-border/dmg-2x.cgp`
- `hlsl/handheld/console-border/dmg-3x.cgp`
- `hlsl/handheld/console-border/dmg-4x.cgp`
- `hlsl/handheld/console-border/dmg-5x.cgp`
- `hlsl/handheld/console-border/dmg-6x.cgp`
- `hlsl/handheld/console-border/gb-dmg-alt-2x.cgp`
- `hlsl/handheld/console-border/gb-dmg-alt-3x.cgp`
- `hlsl/handheld/console-border/gb-dmg-alt-4x.cgp`
- `hlsl/handheld/console-border/gb-dmg-alt-5x.cgp`
- `hlsl/handheld/console-border/gb-dmg-alt-6x.cgp`
- `hlsl/handheld/console-border/gb-dmg-alt-7x.cgp`
- `hlsl/handheld/console-border/gb-light-alt-2x.cgp`
- `hlsl/handheld/console-border/gb-light-alt-3x.cgp`
- `hlsl/handheld/console-border/gb-light-alt-4x.cgp`
- `hlsl/handheld/console-border/gb-light-alt-5x.cgp`
- `hlsl/handheld/console-border/gb-light-alt-6x.cgp`
- `hlsl/handheld/console-border/gb-light-alt-7x.cgp`
- `hlsl/handheld/console-border/gb-pocket-2x.cgp`
- `hlsl/handheld/console-border/gb-pocket-3x.cgp`
- `hlsl/handheld/console-border/gb-pocket-4x.cgp`
- `hlsl/handheld/console-border/gb-pocket-5x.cgp`
- `hlsl/handheld/console-border/gb-pocket-6x.cgp`
- `hlsl/handheld/console-border/gb-pocket-alt-2x.cgp`
- `hlsl/handheld/console-border/gb-pocket-alt-3x.cgp`
- `hlsl/handheld/console-border/gb-pocket-alt-4x.cgp`
- `hlsl/handheld/console-border/gb-pocket-alt-5x.cgp`
- `hlsl/handheld/console-border/gb-pocket-alt-6x.cgp`
- `hlsl/handheld/console-border/gb-pocket-alt-7x.cgp`
- `hlsl/handheld/console-border/gba-2x.cgp`
- `hlsl/handheld/console-border/gba-3x.cgp`
- `hlsl/handheld/console-border/gba-4x.cgp`
- `hlsl/handheld/console-border/gba-5x.cgp`
- `hlsl/handheld/console-border/gba-6x.cgp`
- `hlsl/handheld/console-border/gba-lcd-grid-v2-2x.cgp`
- `hlsl/handheld/console-border/gba-lcd-grid-v2-3x.cgp`
- `hlsl/handheld/console-border/gba-lcd-grid-v2-4x.cgp`
- `hlsl/handheld/console-border/gba-lcd-grid-v2-5x.cgp`
- `hlsl/handheld/console-border/gba-lcd-grid-v2-6x.cgp`
- `hlsl/handheld/console-border/gba-retro-v2-2x.cgp`
- `hlsl/handheld/console-border/gba-retro-v2-3x.cgp`
- `hlsl/handheld/console-border/gba-retro-v2-4x.cgp`
- `hlsl/handheld/console-border/gba-retro-v2-5x.cgp`
- `hlsl/handheld/console-border/gba-retro-v2-6x.cgp`
- `hlsl/handheld/console-border/gbc-2x.cgp`
- `hlsl/handheld/console-border/gbc-3x.cgp`
- `hlsl/handheld/console-border/gbc-4x.cgp`
- `hlsl/handheld/console-border/gbc-5x.cgp`
- `hlsl/handheld/console-border/gbc-6x.cgp`
- `hlsl/handheld/console-border/gbc-lcd-grid-v2-2x.cgp`
- `hlsl/handheld/console-border/gbc-lcd-grid-v2-3x.cgp`
- `hlsl/handheld/console-border/gbc-lcd-grid-v2-4x.cgp`
- `hlsl/handheld/console-border/gbc-lcd-grid-v2-5x.cgp`
- `hlsl/handheld/console-border/gbc-lcd-grid-v2-6x.cgp`
- `hlsl/handheld/console-border/gbc-lcd-grid-v2-7x.cgp`
- `hlsl/handheld/console-border/gbc-retro-v2-2x.cgp`
- `hlsl/handheld/console-border/gbc-retro-v2-3x.cgp`
- `hlsl/handheld/console-border/gbc-retro-v2-4x.cgp`
- `hlsl/handheld/console-border/gbc-retro-v2-5x.cgp`
- `hlsl/handheld/console-border/gbc-retro-v2-6x.cgp`
- `hlsl/handheld/console-border/gbc-retro-v2-7x.cgp`
- `hlsl/handheld/console-border/gg-2x.cgp`
- `hlsl/handheld/console-border/gg-3x.cgp`
- `hlsl/handheld/console-border/gg-4x.cgp`
- `hlsl/handheld/console-border/gg-5x.cgp`
- `hlsl/handheld/console-border/gg-6x.cgp`
- `hlsl/handheld/console-border/ngpc-2x.cgp`
- `hlsl/handheld/console-border/ngpc-3x.cgp`
- `hlsl/handheld/console-border/ngpc-4x.cgp`
- `hlsl/handheld/console-border/ngpc-5x.cgp`
- `hlsl/handheld/console-border/ngpc-6x.cgp`
- `hlsl/handheld/console-border/psp-2x.cgp`
- `hlsl/handheld/console-border/psp-3x.cgp`
- `hlsl/handheld/console-border/psp-4x.cgp`
- `hlsl/handheld/console-border/psp-5x.cgp`
- `hlsl/handheld/console-border/psp-6x.cgp`
- `hlsl/handheld/dot.cgp`
- `hlsl/handheld/ds-hybrid-sabr.cgp`
- `hlsl/handheld/gb-light-shader.cgp`
- `hlsl/handheld/gb-palette-dmg.cgp`
- `hlsl/handheld/gb-palette-light.cgp`
- `hlsl/handheld/gb-palette-pocket.cgp`
- `hlsl/handheld/gb-pocket-shader.cgp`
- `hlsl/handheld/gb-shader.cgp`
- `hlsl/handheld/gba-color.cgp`
- `hlsl/handheld/gbc-color.cgp`
- `hlsl/handheld/gbc-gambatte-color.cgp`
- `hlsl/handheld/lcd-cgwg.cgp`
- `hlsl/handheld/lcd-grid-v2-gba-color-motionblur.cgp`
- `hlsl/handheld/lcd-grid-v2-gba-color.cgp`
- `hlsl/handheld/lcd-grid-v2-gbc-color-motionblur.cgp`
- `hlsl/handheld/lcd-grid-v2-gbc-color.cgp`
- `hlsl/handheld/lcd-grid-v2-motionblur.cgp`
- `hlsl/handheld/lcd-grid-v2-nds-color-motionblur.cgp`
- `hlsl/handheld/lcd-grid-v2-nds-color.cgp`
- `hlsl/handheld/lcd-grid-v2-palm-color-motionblur.cgp`
- `hlsl/handheld/lcd-grid-v2-palm-color.cgp`
- `hlsl/handheld/lcd-grid-v2-psp-color-motionblur.cgp`
- `hlsl/handheld/lcd-grid-v2-psp-color.cgp`
- `hlsl/handheld/lcd-grid-v2-vba-color-motionblur.cgp`
- `hlsl/handheld/lcd-grid-v2-vba-color.cgp`
- `hlsl/handheld/lcd-grid-v2.cgp`
- `hlsl/handheld/lcd-shader-gba-color.cgp`
- `hlsl/handheld/lcd-shader-gbc-color.cgp`
- `hlsl/handheld/lcd-shader-nds-color.cgp`
- `hlsl/handheld/lcd-shader-palm-color.cgp`
- `hlsl/handheld/lcd-shader-psp-color.cgp`
- `hlsl/handheld/lcd-shader-vba-color.cgp`
- `hlsl/handheld/lcd-shader.cgp`
- `hlsl/handheld/lcd3x.cgp`
- `hlsl/handheld/nds-color.cgp`
- `hlsl/handheld/nds.cgp`
- `hlsl/handheld/palm-color.cgp`
- `hlsl/handheld/psp-color.cgp`
- `hlsl/handheld/retro-v2.cgp`
- `hlsl/handheld/vba-color.cgp`
- `hlsl/handheld/zfast-lcd.cgp`
- `hlsl/hqx/hq2x.cgp`
- `hlsl/hqx/hq3x.cgp`
- `hlsl/hqx/hq4x.cgp`
- `hlsl/hqx/single-pass/hq2x.cgp`
- `hlsl/hqx/single-pass/hq3x.cgp`
- `hlsl/hqx/single-pass/hq4x.cgp`
- `hlsl/interpolation/aann.cgp`
- `hlsl/interpolation/pixellate.cgp`
- `hlsl/interpolation/quilez.cgp`
- `hlsl/interpolation/sharp-bilinear.cgp`
- `hlsl/interpolation/smootheststep.cgp`
- `hlsl/interpolation/smoothstep.cgp`
- `hlsl/motionblur/braid-rewind.cgp`
- `hlsl/motionblur/feedback.cgp`
- `hlsl/motionblur/motionblur-blue.cgp`
- `hlsl/motionblur/motionblur-color.cgp`
- `hlsl/motionblur/motionblur-simple.cgp`
- `hlsl/motionblur/response-time.cgp`
- `hlsl/mudlord/bloom.cgp`
- `hlsl/mudlord/blur.cgp`
- `hlsl/mudlord/emboss.cgp`
- `hlsl/mudlord/mud-mudlord.cgp`
- `hlsl/mudlord/noise-mudlord.cgp`
- `hlsl/mudlord/oldtv.cgp`
- `hlsl/mudlord/sharpen.cgp`
- `hlsl/mudlord/toon.cgp`
- `hlsl/mudlord/waterpaint-mudlord.cgp`
- `hlsl/nearest.cgp`
- `hlsl/nedi/fast-bilateral-nedi.cgp`
- `hlsl/nedi/nedi.cgp`
- `hlsl/neon/neon-variation-1.cgp`
- `hlsl/ntsc/ntsc-256px-gauss-scanline.cgp`
- `hlsl/ntsc/ntsc-256px-svideo-gauss-scanline.cgp`
- `hlsl/ntsc/ntsc-256px-svideo.cgp`
- `hlsl/ntsc/ntsc-256px.cgp`
- `hlsl/ntsc/ntsc-320px-gauss-scanline.cgp`
- `hlsl/ntsc/ntsc-320px-svideo-gauss-scanline.cgp`
- `hlsl/ntsc/ntsc-320px-svideo.cgp`
- `hlsl/ntsc/ntsc-320px.cgp`
- `hlsl/ntsc/ntsc-nes.cgp`
- `hlsl/ntsc/ntsc-svideo.cgp`
- `hlsl/ntsc/ntsc-vcr.cgp`
- `hlsl/ntsc/ntsc.cgp`
- `hlsl/pal/pal-r57shell-nes-svideo.cgp`
- `hlsl/pal/pal-r57shell.cgp`
- `hlsl/quad/biquad.cgp`
- `hlsl/quad/quad_interp.cgp`
- `hlsl/reshade/bsnes-gamma-ramp.cgp`
- `hlsl/reshade/gba.cgp`
- `hlsl/reshade/halftone-print.cgp`
- `hlsl/reshade/nds.cgp`
- `hlsl/reshade/vba.cgp`
- `hlsl/sabr/sabr-v1.1.cgp`
- `hlsl/sabr/sabr-v3.0.cgp`
- `hlsl/scalefx/scalefx-hybrid.cgp`
- `hlsl/scalefx/scalefx.cgp`
- `hlsl/scalefx/scalefx+rAA.cgp`
- `hlsl/scalefx/shaders/old/scalefx_hybrid.cgp`
- `hlsl/scalefx/shaders/old/scalefx.cgp`
- `hlsl/scalefx/shaders/old/scalefx9.cgp`
- `hlsl/scalefx/shaders/old/xsoft+scalefx_hybrid.cgp`
- `hlsl/scalefx/shaders/old/xsoft+scalefx.cgp`
- `hlsl/scalefx/shaders/old/xsoft+scalefx+sharpsmoother.cgp`
- `hlsl/scalefx/shaders/old/xsofter+scalefx_hybrid.cgp`
- `hlsl/scalefx/shaders/old/xsofter+scalefx.cgp`
- `hlsl/scalehq/2xScaleHQ.cgp`
- `hlsl/scalehq/4xScaleHQ.cgp`
- `hlsl/scalenx/scale2x.cgp`
- `hlsl/scalenx/scale2xplus.cgp`
- `hlsl/scalenx/scale2xSFX.cgp`
- `hlsl/scalenx/scale3x.cgp`
- `hlsl/scalenx/scale3xSFX.cgp`
- `hlsl/sharpen/adaptive-sharpen-multipass.cgp`
- `hlsl/sharpen/adaptive-sharpen.cgp`
- `hlsl/sharpen/super-xbr-super-res.cgp`
- `hlsl/waterpaint/water.cgp`
- `hlsl/waterpaint/waterpaint-hc.cgp`
- `hlsl/waterpaint/waterpaint.cgp`
- `hlsl/windowed/jinc2-sharp.cgp`
- `hlsl/windowed/jinc2-sharper-3d.cgp`
- `hlsl/windowed/jinc2-sharper.cgp`
- `hlsl/windowed/jinc2.cgp`
- `hlsl/windowed/lanczos12.cgp`
- `hlsl/windowed/lanczos16.cgp`
- `hlsl/windowed/lanczos2-sharp-3d.cgp`
- `hlsl/windowed/lanczos2-sharp.cgp`
- `hlsl/windowed/lanczos4.cgp`
- `hlsl/windowed/lanczos6.cgp`
- `hlsl/xbr/5xbr-retro.cgp`
- `hlsl/xbr/shaders/xbr-lv2-multipass/xbr-lv2-a.cgp`
- `hlsl/xbr/shaders/xbr-lv2-multipass/xbr-lv2-b.cgp`
- `hlsl/xbr/shaders/xbr-lv2-multipass/xbr-lv2-c.cgp`
- `hlsl/xbr/shaders/xbr-lv2-multipass/xbr-lv2-d.cgp`
- `hlsl/xbr/shaders/xbr-lv2-multipass/xbr-lv2-dilation-a.cgp`
- `hlsl/xbr/shaders/xbr-lv2-multipass/xbr-lv2-dilation-b.cgp`
- `hlsl/xbr/shaders/xbr-lv2-multipass/xbr-lv2-dilation-c.cgp`
- `hlsl/xbr/shaders/xbr-lv2-multipass/xbr-lv2-dilation-d.cgp`
- `hlsl/xbr/shaders/xbr-lv2-multipass/xbr-lv2-noblend-dilation-c.cgp`
- `hlsl/xbr/super-2xbr-3d-2p.cgp`
- `hlsl/xbr/super-2xbr-3d-3p-smoother.cgp`
- `hlsl/xbr/super-4xbr-3d-4p.cgp`
- `hlsl/xbr/super-4xbr-3d-6p-smoother.cgp`
- `hlsl/xbr/super-8xbr-3d-6p.cgp`
- `hlsl/xbr/super-xbr-2p.cgp`
- `hlsl/xbr/super-xbr-3p-smoother.cgp`
- `hlsl/xbr/super-xbr-6p-adaptive.cgp`
- `hlsl/xbr/super-xbr-6p-small-details.cgp`
- `hlsl/xbr/super-xbr-6p.cgp`
- `hlsl/xbr/super-xbr-deposterize.cgp`
- `hlsl/xbr/super-xbr-fast-3p.cgp`
- `hlsl/xbr/super-xbr-fast-6p.cgp`
- `hlsl/xbr/xbr-hybrid.cgp`
- `hlsl/xbr/xbr-lv1-noblend.cgp`
- `hlsl/xbr/xbr-lv2-3d.cgp`
- `hlsl/xbr/xbr-lv2-accuracy-multipass.cgp`
- `hlsl/xbr/xbr-lv2-accuracy-smart-blur.cgp`
- `hlsl/xbr/xbr-lv2-deposterize.cgp`
- `hlsl/xbr/xbr-lv2-fast.cgp`
- `hlsl/xbr/xbr-lv2-multipass.cgp`
- `hlsl/xbr/xbr-lv2-noblend.cgp`
- `hlsl/xbr/xbr-lv2-small-details.cgp`
- `hlsl/xbr/xbr-lv2.cgp`
- `hlsl/xbr/xbr-lv3-multipass.cgp`
- `hlsl/xbr/xbr-lv3-noblend.cgp`
- `hlsl/xbr/xbr-lv3.cgp`
- `hlsl/xbr/xbr-mlv4-dilation.cgp`
- `hlsl/xbr/xbr-mlv4-multipass.cgp`
- `hlsl/xbrz/2xbrz-linear.cgp`
- `hlsl/xbrz/3xbrz-linear.cgp`
- `hlsl/xbrz/4xbrz-linear.cgp`
- `hlsl/xbrz/5xbrz-linear.cgp`
- `hlsl/xbrz/6xbrz-linear.cgp`
- `hlsl/xbrz/xbrz-freescale-multipass.cgp`
- `hlsl/xbrz/xbrz-freescale.cgp`
- `hlsl/xsai/super-2xsai.cgp`
- `hlsl/xsal/2xsal-level2-crt.cgp`
- `hlsl/xsal/2xsal.cgp`
- `hlsl/xsal/4xsal-level2-crt.cgp`
- `hlsl/xsal/4xsal-level2-hq.cgp`
- `hlsl/xsal/4xsal-level2.cgp`
- `hlsl/xsal/4xsal-level3-xbr-fast.cgp`
- `hlsl/xsal/4xsal-level3-xbr-soft-fast.cgp`
- `hlsl/xsal/4xsal-level3-xbr-soft.cgp`
- `hlsl/xsal/4xsal-level3-xbr.cgp`
- `hlsl/xsoft/4xsoft.cgp`
- `hlsl/xsoft/4xsoftSdB.cgp`

## Fidelity limitation: intermediate mipmaps

68 presets. This group is not automatically rejected; it requires visual runtime qualification.

- `hlsl/blurs/benchmark-presets/benchmark-blur10x10shared-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur10x10shared-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur11fast-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur11fast-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur11resize-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur11resize-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur12x12shared-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur12x12shared-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3fast-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3fast-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3resize-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3resize-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3x3-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3x3-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3x3resize-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur3x3resize-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur5fast-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur5fast-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur5resize-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur5resize-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur5x5-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur5x5-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur6x6shared-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur6x6shared-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur7fast-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur7fast-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur7resize-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur7resize-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur7x7-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur7x7-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur8x8shared-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur8x8shared-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur9fast-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur9fast-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur9resize-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur9resize-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur9x9-mipmap-gamma-encode-every-fbo-six-times.cgp`
- `hlsl/blurs/benchmark-presets/benchmark-blur9x9-mipmap-srgb-six-times.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur10x10shared-mipmap-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur10x10shared-mipmap-srgb.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur12x12shared-mipmap-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur12x12shared-mipmap-srgb.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur6x6shared-mipmap-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur6x6shared-mipmap-srgb.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur8x8shared-mipmap-gamma-encode-every-fbo.cgp`
- `hlsl/blurs/large-input-quality-test-presets/test-large-input-blur8x8shared-mipmap-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur10x10shared-mipmap-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur12x12shared-mipmap-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur6x6shared-mipmap-srgb.cgp`
- `hlsl/blurs/quality-test-presets/test-blur8x8shared-mipmap-srgb.cgp`
- `hlsl/borders/gameboy-player/gameboy-player+crt-royale.cgp`
- `hlsl/borders/sgb/sgb+crt-royale.cgp`
- `hlsl/cgp/crt-royale-kurozumi.cgp`
- `hlsl/crt/crt-hyllian-glow.cgp`
- `hlsl/crt/crt-royale-fake-bloom-intel.cgp`
- `hlsl/crt/crt-royale-fake-bloom.cgp`
- `hlsl/crt/crt-royale-gdapt.cgp`
- `hlsl/crt/crt-royale-intel.cgp`
- `hlsl/crt/crt-royale-ntsc-256px-composite.cgp`
- `hlsl/crt/crt-royale-ntsc-256px-svideo.cgp`
- `hlsl/crt/crt-royale-ntsc-320px-composite.cgp`
- `hlsl/crt/crt-royale-ntsc-320px-svideo.cgp`
- `hlsl/crt/crt-royale-pal-r57shell.cgp`
- `hlsl/crt/crt-royale-tvout+image-adjustment.cgp`
- `hlsl/crt/crt-royale.cgp`
- `hlsl/crt/crtglow_gauss_ntsc_3phase.cgp`
- `hlsl/crt/crtglow_gauss.cgp`
- `hlsl/crt/crtglow_lanczos.cgp`

## Fidelity limitation: LUT wrap mode

42 presets. This group is not automatically rejected; it requires visual runtime qualification.

- `hlsl/borders/gameboy-player/gameboy-player+crt-royale.cgp`
- `hlsl/borders/sgb/sgb+crt-royale.cgp`
- `hlsl/cgp/crt-royale-kurozumi.cgp`
- `hlsl/crt/crt-royale-fake-bloom-intel.cgp`
- `hlsl/crt/crt-royale-fake-bloom.cgp`
- `hlsl/crt/crt-royale-gdapt.cgp`
- `hlsl/crt/crt-royale-intel.cgp`
- `hlsl/crt/crt-royale-ntsc-256px-composite.cgp`
- `hlsl/crt/crt-royale-ntsc-256px-svideo.cgp`
- `hlsl/crt/crt-royale-ntsc-320px-composite.cgp`
- `hlsl/crt/crt-royale-ntsc-320px-svideo.cgp`
- `hlsl/crt/crt-royale-pal-r57shell.cgp`
- `hlsl/crt/crt-royale-tvout+image-adjustment.cgp`
- `hlsl/crt/crt-royale.cgp`
- `hlsl/crt/GTU-famicom-radeon.cgp`
- `hlsl/handheld/console-border/gba-2x.cgp`
- `hlsl/handheld/console-border/gba-3x.cgp`
- `hlsl/handheld/console-border/gba-4x.cgp`
- `hlsl/handheld/console-border/gba-5x.cgp`
- `hlsl/handheld/console-border/gba-6x.cgp`
- `hlsl/handheld/console-border/gbc-2x.cgp`
- `hlsl/handheld/console-border/gbc-3x.cgp`
- `hlsl/handheld/console-border/gbc-4x.cgp`
- `hlsl/handheld/console-border/gbc-5x.cgp`
- `hlsl/handheld/console-border/gbc-6x.cgp`
- `hlsl/handheld/console-border/gg-2x.cgp`
- `hlsl/handheld/console-border/gg-3x.cgp`
- `hlsl/handheld/console-border/gg-4x.cgp`
- `hlsl/handheld/console-border/gg-5x.cgp`
- `hlsl/handheld/console-border/gg-6x.cgp`
- `hlsl/handheld/console-border/ngpc-2x.cgp`
- `hlsl/handheld/console-border/ngpc-3x.cgp`
- `hlsl/handheld/console-border/ngpc-4x.cgp`
- `hlsl/handheld/console-border/ngpc-5x.cgp`
- `hlsl/handheld/console-border/ngpc-6x.cgp`
- `hlsl/handheld/console-border/psp-2x.cgp`
- `hlsl/handheld/console-border/psp-3x.cgp`
- `hlsl/handheld/console-border/psp-4x.cgp`
- `hlsl/handheld/console-border/psp-5x.cgp`
- `hlsl/handheld/console-border/psp-6x.cgp`
- `hlsl/pal/pal-r57shell-nes-svideo.cgp`
- `hlsl/pal/pal-r57shell.cgp`

## No HLSL equivalent for a curated GL/GLES preset

These three curated GL/GL ES presets have no functional HLSL equivalent in the audited tree. Curated presets with a differently named HLSL equivalent, such as CRT Lottes and Zfast CRT, are not listed.

- `glsl/crt/crt-consumer.glslp`
- `glsl/pal/Amiga_a520.glslp`
- `glsl/vhs/vhs.glslp`

## Runtime qualification failures

The original qualification matrix was run on Windows with Kodi `22.0-BETA2 (21.90.802) Git:20260829-a74b193a2c`, the `game.shader.presets` `22.1.3` add-on, `game.libretro.xrick` `0.21212.0.50`, and a Parallels Display Adapter at D3D feature level 11_1. The executable under test was the x64 Debug build at `C:\Users\garrett\Documents\kodi\build\Debug\kodi.exe`; the deployed manifest and HLSL resource tree came from this repository. Post-fix qualification used a rebuild from the same Kodi tree with named-`TEQ` and pass-index-0 validation and diagnostics plus a valid 28-byte vertex layout carrying both `TEXCOORD0` and `TEXCOORD1`.

Each of the 55 Task 1 additions was selected in one fresh, visible, portable Kodi process. Qualification required an exact `RetroPlayer.VideoFilter` absolute-path match, a recognizable full-screen XRick composite screenshot, inspection of the complete fresh `kodi.log`, a scan for shader/parser/resource/D3D/LUT failures, and bounded shutdown. All 55 paths matched and all 55 screenshots were inspected. The first pass produced 29 clean normal-shutdown runs, 23 runs with documented shader failures, and three clean force-stop-only runs. The latter three (`DDT Extended`, `4x ScaleHQ`, and `Retro v2 + GBA Color`) each passed a fresh exact-path, clean-log, visually acceptable, normal-shutdown retest. Six isolated runs required the executable-guarded bounded force stop; three already had decisive shader failures, and the other three passed the normal-shutdown retests.

The final result is 31 retained additions and 24 removals, leaving 45 Windows HLSL presets total (the 14 pre-existing entries plus 31 additions). FXAA passed isolated and live-switch qualification after its wrapper correction. SGB remained removed after the Kodi UV1 fix exposed a separate final-pass FBO-scaling blocker. Evidence below is relative to `.superpowers/sdd/2026-08-29-expand-windows-shader-presets/`. The static-feature column maps a retained preset to renderer fidelity work that could still improve it; `none` means the static audit found no unsupported feature in the audited categories.

### Addition results

| # | Preset path | Static feature | Runtime result and blocker | Evidence |
|---:|---|---|---|---|
| 1 | `hlsl/xbr/xbr-lv2.cgp` | filter/wrap | **REMOVE** — HLSL X3014 numeric-constructor argument count; shader initialization failed. | `runtime-isolated/01-xbr_xbr-lv2.cgp/` |
| 2 | `hlsl/xbr/xbr-lv2-noblend.cgp` | filter/wrap | **REMOVE** — HLSL X3014 numeric-constructor argument count; shader initialization failed. | `runtime-isolated/02-xbr_xbr-lv2-noblend.cgp/` |
| 3 | `hlsl/xbrz/4xbrz-linear.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/03-xbrz_4xbrz-linear.cgp/` |
| 4 | `hlsl/xbrz/xbrz-freescale.cgp` | filter/wrap | **REMOVE** — HLSL X3503 `main_vertex` return value missing semantics; shader initialization failed. | `runtime-isolated/04-xbrz_xbrz-freescale.cgp/` |
| 5 | `hlsl/crt/crt-hyllian.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/05-crt_crt-hyllian.cgp/` |
| 6 | `hlsl/crt/crt-easymode.cgp` | filter/wrap | **REMOVE** — HLSL X3014 numeric-constructor argument count; shader initialization failed. | `runtime-isolated/06-crt_crt-easymode.cgp/` |
| 7 | `hlsl/handheld/dot.cgp` | filter/wrap | **REMOVE** — HLSL X3004 undeclared `IN`; shader initialization failed. | `runtime-isolated/07-handheld_dot.cgp/` |
| 8 | `hlsl/borders/color-grid.cgp` | none | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/08-borders_color-grid.cgp/` |
| 9 | `hlsl/borders/shiny-iterations.cgp` | none | **REMOVE** — HLSL X3004 undeclared `vec4`; shader initialization failed. | `runtime-isolated/09-borders_shiny-iterations.cgp/` |
| 10 | `hlsl/borders/snow.cgp` | none | **REMOVE** — HLSL X3004 undeclared `vec4`; shader initialization failed and the screenshot was visibly corrupt. | `runtime-isolated/10-borders_snow.cgp/` |
| 11 | `hlsl/anti-aliasing/reverse-aa.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/11-anti-aliasing_reverse-aa.cgp/` |
| 12 | `hlsl/anti-aliasing/fx-aa.cgp` | filter/wrap | **PASS** — The original run failed because the modern wrapper omitted `COMPAT_END`. With the terminator restored, the source emits `TEQ`/`P0`, passes the static compile gate, and completed a fresh exact-path isolated run with zero documented failure signatures, an acceptable full-screen composite, and normal shutdown. FXAA was also the final selection in a normal five-item live-switch batch with 5/5 exact focused paths and an acceptable selector preview. | Original failure: `runtime-isolated/12-anti-aliasing_fx-aa.cgp/`; static gate: `fxaa-acceptance.ps1`; isolated pass: `runtime-fxaa-fixed-retest/05-anti-aliasing_fx-aa.cgp/`; live switch: `runtime-live-switch-fxaa-final/batch-01/` |
| 13 | `hlsl/anti-aliasing/aa-shader-4.o-level2.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/13-anti-aliasing_aa-shader-4.o-level2.cgp/` |
| 14 | `hlsl/bicubic/bicubic-fast.cgp` | filter/wrap | **REMOVE** — HLSL X3503 `main_vertex` return value missing semantics; shader initialization failed. | `runtime-isolated/14-bicubic_bicubic-fast.cgp/` |
| 15 | `hlsl/bicubic/bicubic-sharp.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/15-bicubic_bicubic-sharp.cgp/` |
| 16 | `hlsl/interpolation/pixellate.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/16-interpolation_pixellate.cgp/` |
| 17 | `hlsl/interpolation/sharp-bilinear.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/17-interpolation_sharp-bilinear.cgp/` |
| 18 | `hlsl/windowed/jinc2-sharp.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/18-windowed_jinc2-sharp.cgp/` |
| 19 | `hlsl/ddt/ddt-extended.cgp` | filter/wrap | **PASS** — initial clean force-stop-only trigger; fresh retest had exact path, clean log, acceptable composite, and normal shutdown. | `runtime-isolated/19-ddt_ddt-extended.cgp/`; `runtime-retest-19/` |
| 20 | `hlsl/sharpen/adaptive-sharpen.cgp` | filter/wrap | **REMOVE** — No FX11 `TEQ` technique/`P0` pass: adaptive-sharpen legacy Cg-to-FX11 port requiring further semantic, binding, and compiler work. | `runtime-isolated/20-sharpen_adaptive-sharpen.cgp/` |
| 21 | `hlsl/scalehq/4xScaleHQ.cgp` | filter/wrap | **PASS** — initial clean force-stop-only trigger; fresh retest had exact path, clean log, acceptable composite, and normal shutdown. | `runtime-isolated/21-scalehq_4xScaleHQ.cgp/`; `runtime-retest-21/` |
| 22 | `hlsl/hqx/single-pass/hq2x.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/22-hqx_single-pass_hq2x.cgp/` |
| 23 | `hlsl/sabr/sabr-v3.0.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/23-sabr_sabr-v3.0.cgp/` |
| 24 | `hlsl/dithering/gdapt.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/24-dithering_gdapt.cgp/` |
| 25 | `hlsl/crt/crt-aperture.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal isolated shutdown; also passed a normal single-target live-switch run. | `runtime-isolated/25-crt_crt-aperture.cgp/`; `runtime-live-switch-singles/batch-15/` |
| 26 | `hlsl/crt/crt-caligari.cgp` | filter/wrap | **REMOVE** — HLSL X3503 `main_vertex` return value missing semantics; shader initialization failed. | `runtime-isolated/26-crt_crt-caligari.cgp/` |
| 27 | `hlsl/crt/crt-cgwg-fast.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/27-crt_crt-cgwg-fast.cgp/` |
| 28 | `hlsl/crt/crt-hyllian-fast.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/28-crt_crt-hyllian-fast.cgp/` |
| 29 | `hlsl/crt/crt-lottes-fast.cgp` | filter/wrap | **REMOVE** — HLSL X3507 not all `CrtsMask` control paths return a value; shader initialization failed. | `runtime-isolated/29-crt_crt-lottes-fast.cgp/` |
| 30 | `hlsl/crt/crt-nes-mini.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/30-crt_crt-nes-mini.cgp/` |
| 31 | `hlsl/crt/zfast-crt.cgp` | filter/wrap | **REMOVE** — HLSL X3000 unrecognized `vec2` and `p`; shader initialization failed. | `runtime-isolated/31-crt_zfast-crt.cgp/` |
| 32 | `hlsl/crt/dotmask.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/32-crt_dotmask.cgp/` |
| 33 | `hlsl/cgp/tvout/tvout.cgp` | none | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/33-cgp_tvout_tvout.cgp/` |
| 34 | `hlsl/cgp/tvout/tvout+ntsc-256px-composite.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/34-cgp_tvout_tvout_ntsc-256px-composite.cgp/` |
| 35 | `hlsl/cgp/tvout/tvout+ntsc-320px-svideo.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/35-cgp_tvout_tvout_ntsc-320px-svideo.cgp/` |
| 36 | `hlsl/cgp/tvout/gtu-famicom-240p.cgp` | filter/wrap | **REMOVE** — HLSL X3000 unexpected `[` plus X3014 numeric-constructor argument count; shader initialization failed. | `runtime-isolated/36-cgp_tvout_gtu-famicom-240p.cgp/` |
| 37 | `hlsl/ntsc/ntsc-256px-svideo.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/37-ntsc_ntsc-256px-svideo.cgp/` |
| 38 | `hlsl/ntsc/ntsc-vcr.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/38-ntsc_ntsc-vcr.cgp/` |
| 39 | `hlsl/crt/snes-hires-blend.cgp` | filter/wrap | **REMOVE** — clean log but the full-resolution composite was black except for the mouse cursor. | `runtime-isolated/39-crt_snes-hires-blend.cgp/` |
| 40 | `hlsl/handheld/bevel.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal isolated shutdown; also passed a normal single-target live-switch run. | `runtime-isolated/40-handheld_bevel.cgp/`; `runtime-live-switch-singles/batch-25/` |
| 41 | `hlsl/handheld/gb-palette-pocket.cgp` | filter/wrap | **REMOVE** — HLSL X3000 unrecognized `fixed4` and `out_color`; shader initialization failed. | `runtime-isolated/41-handheld_gb-palette-pocket.cgp/` |
| 42 | `hlsl/handheld/lcd-grid-v2-gba-color.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/42-handheld_lcd-grid-v2-gba-color.cgp/` |
| 43 | `hlsl/handheld/lcd-grid-v2-nds-color.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/43-handheld_lcd-grid-v2-nds-color.cgp/` |
| 44 | `hlsl/handheld/lcd-grid-v2-psp-color.cgp` | filter/wrap | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/44-handheld_lcd-grid-v2-psp-color.cgp/` |
| 45 | `hlsl/handheld/zfast-lcd.cgp` | filter/wrap | **REMOVE** — HLSL X3503 `main_vertex` return value missing semantics; shader initialization failed. | `runtime-isolated/45-handheld_zfast-lcd.cgp/` |
| 46 | `hlsl/cgp/retro-v2+gba-color.cgp` | filter/wrap | **PASS** — initial clean force-stop-only trigger; fresh retest had exact path, clean log, acceptable composite, and normal shutdown. | `runtime-isolated/46-cgp_retro-v2_gba-color.cgp/`; `runtime-retest-46/` |
| 47 | `hlsl/cgp/gameboy-colors.cgp` | none | **PASS** — exact path, clean log, acceptable composite, normal shutdown. | `runtime-isolated/47-cgp_gameboy-colors.cgp/` |
| 48 | `hlsl/handheld/console-border/gg-4x.cgp` | filter/wrap; LUT wrap | **REMOVE** — HLSL X3014 numeric-constructor argument count; shader initialization failed and the screenshot was visibly corrupt. | `runtime-isolated/48-handheld_console-border_gg-4x.cgp/` |
| 49 | `hlsl/handheld/console-border/gbc-4x.cgp` | filter/wrap; LUT wrap | **REMOVE** — HLSL X3014 numeric-constructor argument count; shader initialization failed. | `runtime-isolated/49-handheld_console-border_gbc-4x.cgp/` |
| 50 | `hlsl/handheld/console-border/psp-2x.cgp` | filter/wrap; LUT wrap | **REMOVE** — No FX11 `TEQ` technique/`P0` pass in shader pass 1: PSP legacy Cg-to-FX11 port requiring further semantic, binding, and compiler work. Kodi now supplies a valid `TEXCOORD1` in its 28-byte vertex input, but that cannot synthesize the missing effect technique; the original screenshot was visibly corrupt. | `runtime-isolated/50-handheld_console-border_psp-2x.cgp/` |
| 51 | `hlsl/borders/imgborder.cgp` | none | **REMOVE** — HLSL X3004 undeclared `mix`; shader initialization failed. | `runtime-isolated/51-borders_imgborder.cgp/` |
| 52 | `hlsl/borders/sgb/sgb.cgp` | absolute final-pass FBO scaling | **REMOVE** — The Kodi UV1 fix corrected the invalid 20-byte vertex-input contract and made the border visible, but the image remained tiny and streaked because Kodi discarded the final pass's explicit 256x224 absolute scale. The retest had the exact active path, zero documented failure signatures, and normal shutdown. | Original visual failure: `runtime-isolated/52-borders_sgb_sgb.cgp/`; post-UV1 retest: `runtime-sgb-uv1-fixed/32-borders_sgb_sgb.cgp/` |
| 53 | `hlsl/borders/water.cgp` | none | **REMOVE** — HLSL X3004 undeclared `vec4`; shader initialization failed. | `runtime-isolated/53-borders_water.cgp/` |
| 54 | `hlsl/mudlord/oldtv.cgp` | filter/wrap | **REMOVE** — No FX11 `TEQ` technique/`P0` pass: oldtv legacy Cg-to-FX11 port requiring further semantic, binding, and compiler work; screenshot visibly corrupt. | `runtime-isolated/54-mudlord_oldtv.cgp/` |
| 55 | `hlsl/waterpaint/waterpaint.cgp` | filter/wrap | **REMOVE** — HLSL X3037 constructor used a non-numeric base type; shader initialization failed. | `runtime-isolated/55-waterpaint_waterpaint.cgp/` |

### Blocker-to-future-work map

- **HLSL source/compiler portability (19 removals):** 1, 2, 4, 6, 7, 9, 10, 14, 26, 29, 31, 36, 41, 45, 48, 49, 51, 53, and 55. These presets become candidates only after their shader sources compile successfully through Kodi's D3DCompiler path. The table records the exact compiler class for each preset; this is shader-source compatibility work, not evidence that a missing preset-parser feature alone would unlock them.
- **Resolved FXAA modern-wrapper omission (1 retained preset):** 12 now ends with `COMPAT_END`. Preprocessing under Kodi's defines emits named `TEQ`/`P0`, both direct stages compile, the complete FX11 effect compiles and reflects that technique/pass, the current-source add-on build completes, and isolated plus live-switch runtime qualification passed. The old failure evidence remains linked to preserve the diagnosis.
- **No FX11 `TEQ` technique/`P0` pass — legacy Cg-to-FX11 ports (3 removals):** 20 (`adaptive-sharpen`, pass 0), 50 (PSP `gb-pass-5`, pass 1), and 54 (`oldtv`, pass 0) are zero-technique legacy Cg sources that need real FX11 ports, including semantic, resource/uniform binding, and compiler work; merely appending a technique is not sufficient. Kodi now supplies PSP's genuine `TEXCOORD1` input through its corrected 28-byte vertex format, so the remaining blocker is the shader port rather than missing vertex data.
- **Resolved Kodi input-contract and diagnostic bugs:** Effects11 accepted zero-technique sources as effect containers. Kodi previously queried technique/pass index 0, received the invalid sentinel, and failed `GetDesc` before `ID3D11Device::CreateInputLayout`; the old “input-layout creation” label described the propagated log, not the cause. Kodi now resolves the named `TEQ` technique and validates its pass at index 0, then uses that same pass consistently for reflection and layout creation while reporting the preset/pass/technique accurately. It also supplies in-bounds `TEXCOORD1` data through a 28-byte vertex layout instead of declaring UV1 beyond the old 20-byte stride. These corrections improve correctness and diagnosis but do not port the three remaining legacy sources.
- **Explicit absolute final-pass FBO scaling (1 tested removal; 5 presets affected):** 52 (`hlsl/borders/sgb/sgb.cgp`) remained tiny and streaked after UV1 made its border visible because Kodi discarded the pass's explicit 256x224 final output size. Implementing this renderer feature would make `hlsl/borders/gameboy-player/gameboy-player.cgp`, `hlsl/borders/gameboy-player/gameboy-player-gba-color.cgp`, `hlsl/borders/sgb/sgb.cgp`, `hlsl/borders/sgba/sgba.cgp`, and `hlsl/borders/sgba/sgba-gba-color.cgp` eligible for qualification.
- **Other valid-looking load with unusable visual output (1 removal):** 39 emitted no documented shader/resource/D3D/LUT signature, but its full-resolution composite was black except for the cursor. Future work needs output/pass-chain investigation rather than compiler repair.
- **Static renderer feature gaps:** the exhaustive frame-history/feedback, original/non-immediate-pass, pass-alias, filter/wrap, intermediate-mipmap, and LUT-wrap lists above remain the authoritative maps from missing renderer features to the full set of presets they may unlock or improve. No missing-source, LUT-load, texture-load, sampler, or target-texture failure was observed in the retained set.

### Live switching and regressions

After deploying the then-pruned 44-entry manifest, all 30 initially retained additions were exercised through Kodi's live Estuary video-filter panel in six restart-bounded five-item batches. Every focused `ListItem.Property(game.videofilter)` value matched the intended absolute path, and each batch's seeded active `RetroPlayer.VideoFilter` value matched its first intended path. All final batch logs had zero documented shader/parser/resource/D3D/LUT signatures. Four batches shut down normally. Two aggregate batches needed bounded force stop during preview teardown; pair isolation narrowed those timing events to batches ending at `CRT Aperture` and `Bevel`, and fresh single-target live-switch runs for both were exact-path, clean-log, visually acceptable, and normal-shutdown. The initial TCP response-framing artifact is retained separately as harness evidence; the corrected framing test passed before the final batches. Final historical switching evidence is in `runtime-live-switch-final-closed/`, `runtime-live-switch-subsets/`, `runtime-live-switch-singles/`, and `test-runtime-json.ps1`.

After restoring FXAA, a final five-item live-switch batch selected all 5/5 intended absolute paths, ended on FXAA, produced zero documented failure signatures and an acceptable selector preview, and shut down normally. Evidence is in `runtime-live-switch-fxaa-final/batch-01/`.

The four original regression presets were rerun after Kodi's named-technique and vertex-input fixes. Every fresh process used the exact active path, produced zero documented failure signatures and an acceptable image, and shut down normally.

| Regression preset | Static feature | Result | Evidence |
|---|---|---|---|
| Scale 2x — `hlsl/scalenx/scale2x.cgp` | filter/wrap | **PASS** — post-fix fresh process, exact path, zero documented failure signatures, acceptable composite, normal shutdown. | `runtime-kodi-input-regressions/01-scalenx_scale2x.cgp/` |
| CRT Geom — `hlsl/crt/crt-geom.cgp` | filter/wrap | **PASS** — post-fix fresh process, exact path, zero documented failure signatures, acceptable composite, normal shutdown. | `runtime-kodi-input-regressions/02-crt_crt-geom.cgp/` |
| NTSC — `hlsl/ntsc/ntsc.cgp` | filter/wrap | **PASS** — post-fix fresh process, exact path, zero documented failure signatures, acceptable composite, normal shutdown. | `runtime-kodi-input-regressions/03-ntsc_ntsc.cgp/` |
| Game Boy — `hlsl/cgp/gameboy-screen-grid.cgp` | none | **PASS** — post-fix fresh process, exact path, zero documented failure signatures, acceptable composite, normal shutdown. | `runtime-kodi-input-regressions/04-cgp_gameboy-screen-grid.cgp/` |
