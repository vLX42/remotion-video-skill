# Project Setup

## Folder Structure

Create a standalone `video/` folder isolated from your app (no shared bundler config):

```
video/
  package.json
  tsconfig.json
  remotion.config.ts
  public/                  # Static assets (staticFile() reads from here)
    music.mp3
    logo.png
  src/
    index.ts               # registerRoot(Root)
    Root.tsx                # <Composition> registration
    MyVideo.tsx             # Top-level TransitionSeries + Audio
    constants.ts            # Colors, FPS, dimensions, scene frame ranges
    fonts.ts                # Google Fonts loading
    scenes/
      01-SceneName.tsx      # One file per scene
      02-SceneName.tsx
    transitions/
      TransitionName.tsx    # Custom TransitionPresentation (see remotion-transitions skill)
    components/
      Background.tsx        # Reusable background gradients
      MockComponent.tsx     # App UI mocks
    static/                 # Source copies (optional, public/ is what Remotion reads)
```

## package.json

```json
{
  "name": "my-video",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "studio": "remotion studio src/index.ts",
    "build": "remotion render src/index.ts CompositionId out/video.mp4",
    "preview": "remotion preview src/index.ts"
  },
  "dependencies": {
    "@remotion/cli": "4.0.242",
    "@remotion/google-fonts": "4.0.242",
    "@remotion/media-utils": "4.0.242",
    "@remotion/transitions": "4.0.242",
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "remotion": "4.0.242"
  },
  "devDependencies": {
    "@types/react": "^18.3.12",
    "@types/react-dom": "^18.3.1",
    "typescript": "^5.7.0"
  }
}
```

**CRITICAL:** All `@remotion/*` packages MUST be the same version. Mismatched versions cause context/hook breakage and unclear errors.

## tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ES2022",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "outDir": "./dist"
  },
  "include": ["src/**/*.ts", "src/**/*.tsx"]
}
```

## remotion.config.ts

```ts
import { Config } from "@remotion/cli/config";

Config.setVideoImageFormat("jpeg");
Config.setCodec("h264");    // Best compatibility (MP4)
Config.setCrf(18);           // Quality: 0=lossless, 18=high, 23=default, 51=worst
```

## Entry Point (src/index.ts)

```ts
import { registerRoot } from "remotion";
import { Root } from "./Root";
registerRoot(Root);
```

## Root Component (src/Root.tsx)

```tsx
import { Composition } from "remotion";
import { MyVideo } from "./MyVideo";

export const Root = () => (
  <Composition
    id="MyVideo"           // Used in CLI: remotion render src/index.ts MyVideo
    component={MyVideo}
    durationInFrames={918} // Total frames
    fps={30}
    width={1920}
    height={1080}
  />
);
```

## Constants (src/constants.ts)

```ts
export const FPS = 30;
export const WIDTH = 1920;
export const HEIGHT = 1080;
export const DURATION_FRAMES = 918;

export const COLORS = {
  primaryBlue: "#1a63f5",
  navy: "#002b45",
  // ... brand colors
} as const;
```

## Fonts (src/fonts.ts)

```ts
import { loadFont, fontFamily as poppinsFamily } from "@remotion/google-fonts/Poppins";
import { loadFont as loadInter, fontFamily as interFamily } from "@remotion/google-fonts/Inter";

loadFont("normal", { weights: ["400", "600", "700", "800"] });
loadInter("normal", { weights: ["400", "500", "600"] });

export const FONT_HEADING = poppinsFamily;
export const FONT_BODY = interFamily;
```

Apply at root level: `<AbsoluteFill style={{ fontFamily: FONT_BODY }}>` and use `FONT_HEADING` for titles.

## Static Assets

- Copy images/audio into `public/` directory
- Reference via `staticFile("filename.mp3")` — NOT relative imports
- For images: `<Img src={staticFile("logo.png")} />` from `remotion`
- For audio: `<Audio src={staticFile("music.mp3")} />`

## Install & Verify

```bash
cd video
pnpm install
npx remotion still src/index.ts MyVideo out/test.png --frame=0   # Render one frame
pnpm studio   # Open interactive preview
```
