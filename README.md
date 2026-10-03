# Lion AR

Web AR that runs in the browser. No app, no install. Point your phone camera at a marker image and a 3D lion statue appears on top of it.

**Live demo: https://armr01.vercel.app/**

## Quick start

1. Open https://armr01.vercel.app/ on your phone.
2. Allow camera access.
3. Point the camera at the marker (`docs/marker.jpg`), shown on another screen or printed.
4. The model locks on and rotates above the marker. A panel at the top shows live fps.

Tested on Chrome for Android. iOS Safari should work since the project only needs a camera and WebGL, but it is untested.

## How it works

Tracking uses [MindAR](https://hiukim.github.io/mind-ar-js-doc/) image targets, not WebXR, so there is no dependency on ARCore or ARKit support in the browser. The marker is compiled once into `targets.mind`. [A-Frame](https://aframe.io/) renders the model and loads it with the Draco decoder.

```
source model -> weld -> simplify -> texture resize -> Draco -> model.glb -> MindAR + A-Frame
```

## Asset pipeline

The source model was 13.56 MB and 300k triangles, too heavy for a phone browser. It is reduced with [glTF-Transform](https://gltf-transform.dev/).

| Metric | Before | After | Change |
|---|---|---|---|
| Triangles | 300,000 | 14,884 | -95.0% |
| Vertices | 176,331 | 14,408 | -91.8% |
| Base colour texture | 4096 x 4096 | 1024 x 1024 | -96.8% size |
| Texture GPU memory | 89.48 MB | 5.59 MB | -93.8% |
| File size | 13.56 MB | 0.24 MB | -98.2% |

```bash
npm install --global @gltf-transform/cli

gltf-transform weld original.glb step1.glb
gltf-transform simplify step1.glb step2.glb --ratio 0.05 --error 0.01
gltf-transform resize step2.glb step3.glb --width 1024 --height 1024
gltf-transform draco step3.glb model.glb
```

## Performance

About 46.8 fps while tracking on Android Chrome.

## Project structure

```
.
├── index.html          scene setup (A-Frame + MindAR)
├── assets/
│   ├── model.glb       optimized, Draco-compressed model
│   └── targets.mind    compiled image target
└── docs/
    └── marker.jpg      the image to point the camera at
```

## Run locally

Camera access requires HTTPS, except on localhost.

```bash
npx serve .
```

Then open the printed address. To test on a phone, deploy to any static host (Vercel, Netlify, GitHub Pages).

## Customize

**Marker.** Compile your own image with the [MindAR compiler](https://hiukim.github.io/mind-ar-js-doc/tools/compile) and replace `assets/targets.mind`. Flat, matte images with plenty of detail track best. Check that the detected feature points are spread across the whole image.

**Model.** Run your `.glb` through the pipeline above and replace `assets/model.glb`. If it appears too large, too small or off-centre, adjust `scale`, `rotation` and `position` in `index.html`.

## Built with

- [A-Frame](https://aframe.io/) 1.5.0
- [MindAR](https://hiukim.github.io/mind-ar-js-doc/) 1.2.5
- [Draco](https://github.com/google/draco) decoder 1.5.6
- [glTF-Transform](https://gltf-transform.dev/)

## Contributing

Issues and pull requests are welcome.

## License

Code is released under the MIT License. See `LICENSE`.

## Credits

3D model: "[model title]" by [author], licensed [licence]. Source: [Sketchfab link]
