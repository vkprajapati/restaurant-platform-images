# restaurant-platform-images
Public image assets for the restaurant platform, including product photography and homepage sliders.

---

## Contents

| Path | Files | Description |
|---|---:|---|
| `products/` | 104 | One JPEG per menu product, across 15 categories |
| `sliders/` | 3 | The three homepage slider images |
| `products/mapping.csv` | — | Original filename ↔ product ↔ category ↔ source URL ↔ checksum |
| `sliders/mapping.csv` | — | Slider record ↔ slider image ↔ source product ↔ source URL |
| `IMAGE_EXTRACTION_REPORT.md` | — | Full extraction, verification and findings report |

**107 images total, 9.55 MB.** All are untouched JPEG masters served by the
restaurant — nothing was resized, re-encoded or recompressed.

## Source

Extracted from <https://arkuszowa.curry-house.pl/> (Curry House, ul. Arkuszowa 30,
Warsaw). Product URLs came from the menu markup; the product-to-URL mapping and the
slider definitions came from the platform's own
`database/seeders/data/menu.php` and `database/seeders/SliderSeeder.php`.

## Filenames

Files are named with Laravel's `Str::slug($productName)`, matching the
`Product.image_path` / `Slider.image_path` values used by the application, e.g.
`36-butter-chicken.jpg`. The original filename from the restaurant's server is
recorded in `mapping.csv` for every file, together with the exact source URL.

## Usage

```
https://raw.githubusercontent.com/vkprajapati/restaurant-platform-images/Master/products/36-butter-chicken.jpg
```

```html
<img src="https://raw.githubusercontent.com/vkprajapati/restaurant-platform-images/Master/products/36-butter-chicken.jpg"
     alt="36. Butter Chicken" width="500" height="500" loading="lazy">
```

## Re-downloading

The origin sends `Vary: Accept` and content-negotiates. Always request
`Accept: image/jpeg,image/png,image/*;q=0.8` — advertising `image/webp` or
`image/avif` makes the server return a lossy WebP re-encode under a `.jpg`
filename. Send a browser `User-Agent` and
`Referer: https://arkuszowa.curry-house.pl/`; the site blocks hotlinked requests.

## Known issues

See `IMAGE_EXTRACTION_REPORT.md` §10. In short: 12 groups of products share one
identical photograph **on the source website**, and the source site currently has no
image slider of its own (the 3 slider images are product photos matched to the 3
slider records).
