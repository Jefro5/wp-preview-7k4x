# Hero carousel preview videos

The homepage phone carousel plays one looping video per slide. Drop the files here with these exact names:

```
assets/preview-1.mp4   preview-1.webm (optional)   preview-1.jpg (poster, optional)
assets/preview-2.mp4   preview-2.webm              preview-2.jpg
assets/preview-3.mp4   preview-3.webm              preview-3.jpg
```

Until the real files exist, each phone screen shows a gradient placeholder (day / sunset / night), and the
carousel still auto-advances — so the layout looks complete.

## What to capture
A short screen recording of the wallpaper **panning between home screens** (the parallax as you swipe),
looping cleanly. 3–6 seconds is plenty. Keep the app icons/clock if you want it to read as a real home
screen, or hide them for a pure-scene look — your call.

## Format / crop
- **Portrait**, aspect ratio ≈ **9 : 19.3** (matches the phone frame). The frame uses `object-fit: cover`,
  so a normal full-screen phone recording (e.g. 1080×2340 ≈ 9:19.5) fits fine — only a sliver is trimmed.
- **MP4 (H.264)** is required; **WebM (VP9)** optional and smaller. Both should be **muted** and **loop**able.
- Keep each file small — a few MB. It autoplays on load, so lighter = faster page.
- **Poster JPG** (optional): a single frame shown before the video loads / if it fails.

## Handy ffmpeg recipes
From a raw phone recording `raw.mp4` — trim to a clean loop, strip audio, and export web-ready files:

```sh
# MP4 (H.264), trimmed 2s–7s, muted, sized to a 9:19.3 crop of a 1080-wide capture
ffmpeg -ss 2 -to 7 -i raw.mp4 -an -vf "crop=1080:2314:0:ih/2-1157,scale=720:-2" \
       -c:v libx264 -crf 24 -movflags +faststart preview-1.mp4

# WebM (VP9), smaller
ffmpeg -i preview-1.mp4 -an -c:v libvpx-vp9 -b:v 0 -crf 34 preview-1.webm

# Poster frame
ffmpeg -i preview-1.mp4 -frames:v 1 -q:v 3 preview-1.jpg
```
(Adjust the `crop` offsets to keep the part of the screen you want; drop the `crop` filter entirely if the
recording is already the right shape and let `object-fit: cover` handle it.)
