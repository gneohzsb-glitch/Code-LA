ARGUS landing page — assets
===========================

IMAGES  (WebP — smaller and faster than PNG at the same visual quality)

  argus-hero.webp       hero background (character seated on chair)
  argus-portrait.webp   gallery slot
  argus-face.webp       gallery slot
  favicon.ico           site icon  (add this one)

FONTS  (assets/fonts/, referenced from index.html as
        ./assets/fonts/<filename>.ttf)

  Vazirmatn (Persian)         Space Grotesk (English)
  ---------------------       -----------------------
  Vazirmatn-Thin.ttf          SpaceGrotesk-Light.ttf
  Vazirmatn-ExtraLight.ttf    SpaceGrotesk-Regular.ttf
  Vazirmatn-Light.ttf         SpaceGrotesk-Medium.ttf
  Vazirmatn-Regular.ttf       SpaceGrotesk-SemiBold.ttf
  Vazirmatn-Medium.ttf        SpaceGrotesk-Bold.ttf
  Vazirmatn-SemiBold.ttf
  Vazirmatn-Bold.ttf
  Vazirmatn-ExtraBold.ttf
  Vazirmatn-Black.ttf

Code / CLI output uses JetBrains Mono, loaded from Google Fonts
(the only external request the page makes).

All paths are relative, so the page works from GitHub Pages as-is.
No .png references remain anywhere in the HTML, CSS or JavaScript.
