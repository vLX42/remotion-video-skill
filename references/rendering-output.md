# Rendering & Output

## Preview in Remotion Studio

```bash
cd video
pnpm studio
# Opens browser at http://localhost:3000
# - Scrub timeline to preview any frame
# - Click play to preview animation
# - Switch between compositions
# - Inspect individual scenes
```

## Render a Single Frame (Testing)

```bash
# Quick test — render one frame to PNG
npx remotion still src/index.ts CompositionId out/test.png --frame=50

# Test multiple scenes by rendering key frames
npx remotion still src/index.ts MyVideo out/scene1.png --frame=50
npx remotion still src/index.ts MyVideo out/scene3.png --frame=350
npx remotion still src/index.ts MyVideo out/scene5.png --frame=700
```

This is the fastest way to verify scenes render without errors. Always test before doing a full render.

## Full Video Render

```bash
# Render to MP4 (H.264)
npx remotion render src/index.ts CompositionId out/video.mp4

# Or via package.json script
pnpm build
```

### Render Options

```bash
# Custom codec
npx remotion render src/index.ts MyVideo out/video.mp4 --codec=h264

# Quality (CRF: 0=lossless, 18=high, 23=default, 51=worst)
npx remotion render src/index.ts MyVideo out/video.mp4 --crf=18

# Specific frame range (for testing sections)
npx remotion render src/index.ts MyVideo out/clip.mp4 --frames=300-500

# Concurrency (parallel rendering, default: half CPU cores)
npx remotion render src/index.ts MyVideo out/video.mp4 --concurrency=8

# ProRes (for editing in Final Cut/Premiere)
npx remotion render src/index.ts MyVideo out/video.mov --codec=prores --prores-profile=4444

# WebM (VP8/VP9, for web)
npx remotion render src/index.ts MyVideo out/video.webm --codec=vp8

# GIF
npx remotion render src/index.ts MyVideo out/video.gif --codec=gif
```

## Codec Reference

| Codec | Format | Use Case | Quality |
|-------|--------|----------|---------|
| `h264` | .mp4 | Default, best compatibility | CRF 18 = high |
| `h265` | .mp4 | Smaller files, newer devices | CRF 18 = high |
| `prores` | .mov | Professional editing | Lossless-capable |
| `vp8` / `vp9` | .webm | Web embedding | Good |
| `gif` | .gif | Short clips, previews | Low |

## remotion.config.ts Options

```ts
import { Config } from "@remotion/cli/config";

Config.setVideoImageFormat("jpeg");   // "jpeg" or "png" (png = lossless but slower)
Config.setCodec("h264");
Config.setCrf(18);                     // 0-51, lower = better quality
Config.setPixelFormat("yuv420p");      // Best compatibility
Config.setOverwriteOutput(true);       // Don't prompt on overwrite
```

## Chrome Headless Shell

On first render, Remotion downloads Chrome Headless Shell (~80MB). This is cached for subsequent renders.

If you get Chrome-related errors:
```bash
npx remotion browser ensure
```

## Troubleshooting

### Version Mismatch Warning
```
Version mismatch: @remotion/google-fonts is 4.0.435 but others are 4.0.242
```
**Fix:** Pin ALL `@remotion/*` packages to the same version in package.json.

### Bundle Cache Issues
If you see stale content after code changes:
```bash
rm -rf node_modules/.cache
npx remotion render ...
```

### "Webpack config change detected. Clearing cache..."
This is normal — happens when dependencies change. Subsequent renders use the cache.

### Audio Not Playing in Preview
- Ensure audio file is in `public/` directory
- Use `staticFile("music.mp3")`, not a relative import
- Audio only plays during playback in Studio, not when scrubbing

### Blank/Black Frames
- Check that your `durationInFrames` in `<Composition>` matches your frame budget
- Ensure scenes don't reference frames beyond their duration
- Check `clamp` on all `interpolate()` calls to prevent out-of-range values

### Out of Memory on Long Videos
```bash
# Reduce concurrency
npx remotion render ... --concurrency=2

# Or increase Node memory
NODE_OPTIONS="--max-old-space-size=8192" npx remotion render ...
```

## Render Performance Tips

1. **Test with stills first** — `remotion still` is 10x faster than full render
2. **Use `--frames` for section testing** — only render the part you're working on
3. **JPEG image format** — faster than PNG, fine for video
4. **Avoid re-creating objects in render** — define data arrays, spring configs, etc. at module level
5. **Cached bundles** — second render of same code is much faster

## Output File Sizes (Approximate)

| Duration | Resolution | CRF | Size |
|----------|-----------|-----|------|
| 30s | 1920x1080 | 18 | 15-30 MB |
| 30s | 1920x1080 | 23 | 8-15 MB |
| 60s | 1920x1080 | 18 | 30-60 MB |
| 30s | 1280x720 | 18 | 8-15 MB |

## CI/CD Rendering

For rendering in CI (GitHub Actions, etc.):

```yaml
- name: Render video
  run: |
    cd video
    npm install
    npx remotion browser ensure
    npx remotion render src/index.ts MyVideo out/video.mp4
```

Requires ~2GB disk space (Chrome + node_modules) and at least 2 CPU cores.
