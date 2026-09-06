# Horizontal chromatic-aberration bands

`MeshTransmissionMaterial` is doing two things in the original `FluidGlass` component:

1. it renders the scene through a transparent glass surface; and
2. `chromaticAberration` offsets the red, green, and blue wavelengths by a small amount.

The component below keeps the second part and changes the lens from a round, model-based
surface into two fixed horizontal bands. The bands sit at the top and bottom of the
viewport, so they can be placed over an ordinary HTML page. It uses layered backdrop
surfaces instead of a `.glb` file, which makes it a small, dependency-free extraction of
the visual effect.

> This is a CSS/DOM version of the effect. The original `MeshTransmissionMaterial` can
> refract a Three.js scene rendered into an FBO; CSS cannot sample arbitrary DOM pixels as
> a texture. The RGB layers below reproduce the visible colour-fringing effect while
> `backdrop-filter` supplies the glass blur and saturation.

## `HorizontalChromaticBands.tsx`

```tsx
import type { CSSProperties } from 'react';
import { useEffect, useRef } from 'react';
import './horizontal-chromatic-bands.css';

export interface HorizontalChromaticBandsProps {
  /** Height of both viewport bands. Any valid CSS length is accepted. */
  height?: string;

  /**
   * Strength of the RGB separation. This is the CSS equivalent of
   * MeshTransmissionMaterial's `chromaticAberration` prop.
   * Values are clamped to the range 0..1.
   */
  intensity?: number;

  /** Blur applied to the page visible through the glass. */
  blur?: number;

  /** Keep the overlay above the page without blocking clicks. */
  zIndex?: number;

  /** Let the colour split follow the horizontal pointer position. */
  followPointer?: boolean;
}

type CSSVariables = CSSProperties & {
  '--band-height': string;
  '--ca-distance': string;
  '--ca-blur': string;
  '--ca-z-index': number;
};

export default function HorizontalChromaticBands({
  height = 'clamp(3.5rem, 10vh, 7rem)',
  intensity = 0.65,
  blur = 12,
  zIndex = 20,
  followPointer = true
}: HorizontalChromaticBandsProps) {
  const overlayRef = useRef<HTMLDivElement>(null);
  const safeIntensity = Math.min(1, Math.max(0, intensity));

  useEffect(() => {
    if (!followPointer) return;

    const overlay = overlayRef.current;
    if (!overlay) return;

    // A small pointer-dependent offset gives the bands the same responsive feel
    // as the original `followPointer` lens, without moving the fixed bands.
    const updatePointerOffset = (event: PointerEvent) => {
      const progress = event.clientX / window.innerWidth - 0.5;
      const offset = progress * safeIntensity * 10;
      overlay.style.setProperty('--ca-pointer-offset', `${offset}px`);
    };

    window.addEventListener('pointermove', updatePointerOffset, { passive: true });
    return () => window.removeEventListener('pointermove', updatePointerOffset);
  }, [followPointer, safeIntensity]);

  const style: CSSVariables = {
    '--band-height': height,
    // Keep the separation subtle: large offsets stop looking like glass.
    '--ca-distance': `${1 + safeIntensity * 12}px`,
    '--ca-blur': `${Math.max(0, blur)}px`,
    '--ca-z-index': zIndex
  };

  return (
    <div
      ref={overlayRef}
      className="chromatic-bands"
      style={style}
      aria-hidden="true"
    >
      <ChromaticBand edge="top" />
      <ChromaticBand edge="bottom" />
    </div>
  );
}

type ChromaticBandProps = {
  edge: 'top' | 'bottom';
};

function ChromaticBand({ edge }: ChromaticBandProps) {
  return (
    <div className={`chromatic-band chromatic-band--${edge}`}>
      {/* The translucent surface lets the page show through as blurred glass. */}
      <span className="chromatic-band__glass" />

      {/*
        These three layers are the extracted aberration. Each is shifted to a
        slightly different horizontal position and tinted as one wavelength.
        The red and blue layers are deliberately farther apart than the green
        layer, just like a simple RGB split shader.
      */}
      <span className="chromatic-band__channel chromatic-band__channel--red" />
      <span className="chromatic-band__channel chromatic-band__channel--green" />
      <span className="chromatic-band__channel chromatic-band__channel--blue" />

      {/* A thin highlight makes the shape read as glass rather than a colour bar. */}
      <span className="chromatic-band__highlight" />
    </div>
  );
}
```

## `horizontal-chromatic-bands.css`

```css
.chromatic-bands {
  --ca-pointer-offset: 0px;

  position: fixed;
  inset: 0;
  z-index: var(--ca-z-index);
  pointer-events: none; /* the page remains fully interactive */
  overflow: hidden;
  isolation: isolate;
}

.chromatic-band {
  position: absolute;
  left: 0;
  width: 100%;
  height: var(--band-height);
  overflow: hidden;
  isolation: isolate;

  /* These are the rectangular replacements for the original round lens. */
  border-right: 1px solid rgba(255, 255, 255, 0.16);
  border-left: 1px solid rgba(255, 255, 255, 0.16);
  background: rgba(255, 255, 255, 0.025);
}

.chromatic-band--top {
  top: 0;
  border-bottom: 1px solid rgba(255, 255, 255, 0.24);
  border-radius: 0 0 1.25rem 1.25rem;
}

.chromatic-band--bottom {
  bottom: 0;
  border-top: 1px solid rgba(255, 255, 255, 0.24);
  border-radius: 1.25rem 1.25rem 0 0;
}

.chromatic-band__glass,
.chromatic-band__channel,
.chromatic-band__highlight {
  position: absolute;
  inset: 0;
  display: block;
  pointer-events: none;
}

.chromatic-band__glass {
  z-index: 0;
  background:
    linear-gradient(
      180deg,
      rgba(255, 255, 255, 0.19),
      rgba(255, 255, 255, 0.035) 42%,
      rgba(255, 255, 255, 0.09)
    );
  backdrop-filter: blur(var(--ca-blur)) saturate(1.35) contrast(1.08);
  -webkit-backdrop-filter: blur(var(--ca-blur)) saturate(1.35) contrast(1.08);
}

.chromatic-band__channel {
  z-index: 1;
  width: calc(100% + 2 * var(--ca-distance));
  left: calc(-1 * var(--ca-distance));
  right: auto;
  opacity: 0.6;
  filter: blur(7px);
  mix-blend-mode: screen;

  /* Limit the colour to the horizontal edges instead of tinting the whole page. */
  -webkit-mask-image: linear-gradient(
    90deg,
    #000 0%,
    rgba(0, 0, 0, 0.85) 9%,
    transparent 34%,
    transparent 66%,
    rgba(0, 0, 0, 0.85) 91%,
    #000 100%
  );
  mask-image: linear-gradient(
    90deg,
    #000 0%,
    rgba(0, 0, 0, 0.85) 9%,
    transparent 34%,
    transparent 66%,
    rgba(0, 0, 0, 0.85) 91%,
    #000 100%
  );
}

.chromatic-band__channel--red {
  /* Red is shifted left. */
  background:
    radial-gradient(ellipse at 0% 50%, rgba(255, 32, 94, 0.9), transparent 22%),
    radial-gradient(ellipse at 100% 50%, rgba(255, 50, 100, 0.42), transparent 19%);
  transform: translate3d(
    calc(var(--ca-pointer-offset) - var(--ca-distance)),
    0,
    0
  );
}

.chromatic-band__channel--green {
  /* Green stays close to the optical centre. */
  background:
    radial-gradient(ellipse at 0% 50%, rgba(74, 255, 176, 0.35), transparent 18%),
    radial-gradient(ellipse at 100% 50%, rgba(74, 255, 176, 0.18), transparent 16%);
  opacity: 0.35;
  transform: translate3d(var(--ca-pointer-offset), 0, 0);
}

.chromatic-band__channel--blue {
  /* Blue is shifted right, creating the opposite colour fringe. */
  background:
    radial-gradient(ellipse at 0% 50%, rgba(50, 116, 255, 0.42), transparent 19%),
    radial-gradient(ellipse at 100% 50%, rgba(28, 220, 255, 0.95), transparent 22%);
  transform: translate3d(
    calc(var(--ca-pointer-offset) + var(--ca-distance)),
    0,
    0
  );
}

.chromatic-band__highlight {
  z-index: 2;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(255, 255, 255, 0.21) 50%,
    transparent
  );
  opacity: 0.55;
}

/* Backdrop filters are optional; keep the RGB fringe visible on older browsers. */
@supports not (backdrop-filter: blur(1px)) {
  .chromatic-band__glass {
    background: rgba(18, 15, 23, 0.78);
  }
}

/* Do not continuously animate a visual effect for motion-sensitive users. */
@media (prefers-reduced-motion: reduce) {
  .chromatic-band__channel {
    filter: blur(4px);
  }
}
```

## Usage

Place the component once near the root of the page. Because the wrapper is fixed, it stays
at the viewport edges while the page scrolls.

```tsx
import HorizontalChromaticBands from './HorizontalChromaticBands';

export default function App() {
  return (
    <>
      <main>{/* the normal webpage */}</main>

      {/* The overlay is intentionally last so its glass sits above the page. */}
      <HorizontalChromaticBands
        intensity={0.55}
        height="clamp(4rem, 11vh, 8rem)"
        blur={14}
        followPointer
      />
    </>
  );
}
```

To make the effect calmer, lower `intensity` to about `0.25`. Values around `0.55` to
`0.7` are closer to the original lens example's `chromaticAberration` setting. The original
`ior`, `thickness`, and `transmission` props are not copied because they describe the
Three.js transmission simulation; the CSS equivalents here are `backdrop-filter`, the
translucent surface, and the RGB offsets.
