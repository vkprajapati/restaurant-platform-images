# IMAGE_EXTRACTION_REPORT

Extraction of restaurant product and slider imagery from
**https://arkuszowa.curry-house.pl/** into the standalone repository
**https://github.com/vkprajapati/restaurant-platform-images**

| | |
|---|---|
| Report date | 2026-10-02 |
| Source site | https://arkuszowa.curry-house.pl/ (WordPress / WooCommerce, page `strona-glowna`, id 375) |
| Local output | `/Applications/XAMPP/xamppfiles/htdocs/restaurant-platform-images/` |
| Target repository | https://github.com/vkprajapati/restaurant-platform-images (public, default branch `Master`) |
| Target collection | 104 product images + 3 slider images |
| Result | **107 / 107 downloaded, 0 failed, 0 missing** |

---

## 0. Method and constraints

### How the source was inspected

1. `GET /` returned 584 KB of HTML. Every menu item is rendered as a
   `div.fditem-list.item-grid` containing an `<img>` with a `data-src` attribute
   (the site lazy-loads with Perfmatters).
2. Those `data-src` values are the product photography. There are **exactly 104**
   distinct URLs — matching the 104 `image_url` entries in the platform's own
   `database/seeders/data/menu.php` (the two sets are byte-identical, verified).
3. WordPress `srcset` attributes expose the full resolution ladder; for 103 of the
   104 products the unsuffixed URL (`…/name.jpg`) is the largest available asset and
   is the master file. The single exception is *Mini lunch*, referenced as a
   `-120x120` thumbnail (see §7).
4. Non-product imagery on the page was identified and deliberately **excluded**
   (see §3).

### Download headers — important quality finding

The origin responds with `Vary: Accept` and performs **content negotiation**:
when the request advertises WebP/AVIF it returns a lossy WebP re-encode of the
JPEG, even for `.jpg` URLs.

```
Accept: image/avif,image/webp,…            -> Content-Type: image/webp  (340 174 B)
Accept: image/jpeg,image/png,image/*;q=0.8 -> Content-Type: image/jpeg  (705 886 B)
```

A first pass using the browser-style `Accept` header (the same header
`MenuSeeder::downloadImage()` sends) produced **30 files that were actually WebP**
stored under a `.jpg` name. Those were discarded and re-downloaded with
`Accept: image/jpeg,image/png,image/*;q=0.8`, so every file in this repository is the
**untouched JPEG master served by the restaurant**. No image was resized,
re-encoded or recompressed at any point.

### Constraints honoured

* No application source file was created, modified or deleted.
* No database record was written; no migration or seeder was executed.
  `menu.php` and `SliderSeeder.php` were **read only**, to derive the mapping.
* `.env`, `.env.preview`, `vercel.json`, `.vercelignore` untouched.
* Nothing was committed or pushed to `vkprajapati/restaurant-platform`.
* The 104 pre-existing images in
  `restaurant-platform/storage/app/public/products/` were **not** deleted,
  overwritten or modified. They were only opened read-only, for comparison (§8).
* No website source code, script, CSS, logo, icon or unrelated asset was copied.
* No image was invented, generated or substituted for a product that had none —
  every one of the 104 products had a real photograph on the source site.

---

## 1. Total images found

| Category | Count | Notes |
|---|---:|---|
| Product images referenced by the homepage menu | **104** | one `<img data-src>` per menu item |
| Slider images mapped to slider records | **3** | see §5 |
| **Total in scope** | **107** | |
| Non-product imagery found and **excluded** | 4 | see below |

### Excluded non-product imagery (deliberate)

These were found while scanning the page but are **not** product photography or
slider imagery, so they were not downloaded:

| Source file | What it is | Why excluded |
|---|---|---|
| `wp-content/uploads/2020/11/cropped-logo.png` | site logo (910×110) | branding, not product/slider |
| `wp-content/uploads/2023/08/curry.jpg` | Open Graph share image, 1500 px wide | SEO/social asset |
| `wp-content/uploads/2023/08/top-table.jpg?id=6792` | CSS `background-image` of the welcome banner | decorative page background |
| `wp-content/uploads/2022/12/WhatsApp-Image-2026-06-25-at-7.31.39-PM.jpeg` | seasonal promo popup (Popmake id 5979) | time-limited promo overlay |

> **Uncertainty flagged:** `curry.jpg` (1500 px, brand hero photograph) and
> `top-table.jpg` are legitimate Curry House hero photography. They are **not**
> slider slides in the source site (see §2) but would be reasonable hero
> candidates for the platform. They were left out to keep the collection exactly
> 104 + 3. They can be added on request.

---

## 2. Does the source site have a homepage slider?

**No — the source website currently has no image slider on its homepage.** This is
a factual finding that shaped how the 3 slider images were chosen, so it is
recorded explicitly:

* The page is a Visual Composer page. Its content
  (`/wp-json/wp/v2/pages/375`) contains `vc_row` / `vc_column` / `vc_custom_heading`
  shortcodes only — there is no `[rev_slider]`, no `tp-caption`, no `rs-slide`,
  no `rev_slider_wrapper` and no carousel markup anywhere in the HTML.
* The only `#rev_slider_1_1_forcefullwidth` occurrences are two orphaned CSS rules
  left over from the theme — there is no matching markup.
* `/menu/` contains no product images at all; it is a category-navigation page.
* The hero area is a text heading ("Bielany – ul. Arkuszowa 30", phone number)
  over the static background `top-table.jpg`.

The platform's own `Database\Seeders\SliderSeeder` defines exactly **3** slider
records, and each reuses one of the product photographs. The 3 files in
`sliders/` therefore correspond **1:1 to those 3 slider records**, which is what
"mapped to its corresponding slider record" means here.

---

## 3. Total images downloaded

| | Count |
|---|---:|
| `products/` | **104** |
| `sliders/` | **3** |
| **Total files** | **107** |
| Total size | 9.55 MB |
| Files that are valid images | 107 (100%) |
| Empty (0 byte) files | 0 |
| Corrupt / unreadable files | 0 |

### Format and dimensions

| Dimensions | Files | Format |
|---|---:|---|
| 500 × 500 | 103 | JPEG (baseline/progressive, no re-encode) |
| 1500 × 1500 | 3 | JPEG (baseline/progressive, no re-encode) |
| 3240 × 3240 | 1 | JPEG (baseline/progressive, no re-encode) |

Every file was validated by parsing the JPEG SOF marker for real dimensions and
by confirming the JPEG magic bytes; none is a WebP or PNG wearing a `.jpg` name.

---

## 4. Missing images

**None.** All 104 products listed on the source homepage and all 3 slider records
resolved to a real, downloadable image.

| Check | Result |
|---|---|
| Products on source site | 104 |
| Products with an image on the source site | 104 |
| Products downloaded | 104 |
| Products missing an image | **0** |
| Products where a stand-in image was invented | **0** |
| Slider records | 3 |
| Slider images downloaded | 3 |
| Slider images missing | **0** |

Note: the source site's own menu numbering skips `95` (Desery jumps from 94 to 96)
and `05`/… numbering is continuous otherwise. This is a gap in the restaurant's
own menu, not a missing download.

---

## 5. Failed downloads

**None.** HTTP status and byte length were checked for every request.

| Metric | Value |
|---|---:|
| Requests issued | 108 (107 assets + 1 alternative-resolution probe) |
| HTTP 200 responses | 108 |
| Non-200 responses | 0 |
| Timeouts / connection errors | 0 |
| Empty response bodies | 0 |
| **Failed downloads** | **0** |

The site applies referrer-based hotlink protection, so every request sends a
browser `User-Agent` plus `Referer: https://arkuszowa.curry-house.pl/`.
The script also refuses to overwrite an existing destination file.

---

## 6. Duplicate images

Duplicates were detected by MD5 of the downloaded bytes. Two kinds exist and they
have very different origins.

### 6.1 Intentional — `sliders/` re-uses product photos (3 groups)

By design each of the 3 slider records points at a product photograph (that is how
`SliderSeeder` is written), so these files are intentionally identical to their
`products/` counterpart:

| Slider file | Identical product file | Source URL |
|---|---|---|
| `36-butter-chicken.jpg` | `products/36-butter-chicken.jpg` | `butter-chicken-1.jpg` |
| `61-chicken-sizzler.jpg` | `products/61-chicken-sizzler.jpg` | `chicken-sizzler.jpg` |
| `08-chicken-tikka-salad.jpg` | `products/08-chicken-tikka-salad.jpg` | `chicken-salad.jpg` |

### 6.2 Pre-existing on the source website — different products share one photo (12 groups)

These are **not** extraction defects. Each group comes from *different* source
filenames that the restaurant happens to have uploaded with identical content, so
the duplication already exists live on arkuszowa.curry-house.pl. No image was
altered, removed or replaced to hide this.

| # | Products sharing the identical file | Source filenames | Size |
|---:|---|---|---:|
| 1 | 09. Samosa (2 szt.), 15. Chicken/Mutton Samosa | `samosa-1.jpg`, `samosa-1-1.jpg` | 60,391 B |
| 2 | 18. Garlic Chicken Tikka, 19. Murgh Malai Kebeb | `garlic-chicken.jpg`, `garlic-chicken-1.jpg` | 95,628 B |
| 3 | 23. Veg shahi Korma 🍯, 39. Chicken Korma 🍯, 46. Mutton Korma 🍯 | `korma-1.jpg`, `korma-1-1.jpg`, `korma.jpg` | 69,629 B |
| 4 | 30. Paneer Tikka Masala 🌶️, 37. Chicken Tikka Masala 🌶️ | `chicken-tikka-masala.jpg`, `chicken-tikka-masala-1.jpg` | 69,156 B |
| 5 | 31. Palak Paneer, 40. Chicken Palak, 47. Mutton Palak | `palak-paneer-1-1.jpg`, `palak-paneer-1-1-1.jpg`, `palak-paneer-1-1-2.jpg` | 71,544 B |
| 6 | 34. Methi Paneer, 38. Methi Chicken | `methi-chicken.jpg`, `methi-chicken-1.jpg` | 81,963 B |
| 7 | 43. Chicken Madras 🌶️🌶️🌶️, 49. Mutton Madras 🌶️🌶️🌶️ | `mutton-madras.jpg`, `mutton-madras-1.jpg` | 73,671 B |
| 8 | 69. Plain Rice, 70. Jerra Rice | `jerra.jpg`, `jerra-1.jpg` | 67,506 B |
| 9 | 76. Vegetable Biryani 🌶️, 77. Chicken Biryani 🌶️, 78. Mutton Biryani 🌶️ | `biryani.jpg`, `biryani-1.jpg`, `biryani-2.jpg` | 111,485 B |
| 10 | 81. Plain Naan, 82. Butter Naan, 83. Garlic Naan | `naan-1.jpg`, `naan-1-1.jpg`, `naan-1-2.jpg` | 85,463 B |
| 11 | 87. Paneer Naan 🍯, 88. Keema Naan 🌶️, 89. Chicken Keema Naan, 90. Kashmiri Naan 🍯 | `paneer-naan.jpg`, `paneer-naan-1.jpg`, `paneer-naan-2.jpg`, `paneer-naan-3.jpg` | 87,530 B |
| 12 | 92. Plain Curd, 93. Raita | `raita-1.jpg`, `raita-1-1.jpg` | 50,291 B |

**18 of the 104 product files are therefore exact byte-copies of another product
file.** This is a content problem in the restaurant's source menu and needs a
human decision — see §9.

No *accidental* duplicates exist: there are no files left over from earlier runs,
no stray thumbnails such as `-300x300` or `-120x120`, no logos and no unrelated
assets in the directory.

---

## 7. Product-to-image mapping

Filenames follow Laravel's `Str::slug($productName)` — the exact same rule
`MenuSeeder` uses to build `Product.image_path`. That makes every file a drop-in
replacement for the corresponding `image_path` value.

Renaming rules applied (the original filename is preserved for every file in
`products/mapping.csv` and in the table below):

| Original site filename | Stored as | Reason |
|---|---|---|
| *all 104* | `<product-slug>.jpg` | mapped to the platform's `image_path` so the files are directly reusable; the original filename is preserved in `products/mapping.csv` and in the table below |
| `lunch-vege-120x120.jpg` | `products/mini-lunch.jpg` (from `lunch-vege.jpg`) | the menu.php URL points at a 120×120 thumbnail; the full 3240×3240 master of the same photograph was used instead to avoid shipping a downscaled image |

> Every other product URL already pointed at the largest WordPress size, so those
> files were fetched byte-for-byte with no resolution change.

The machine-readable version of this table is **`products/mapping.csv`**
(`file`, `original_filename`, `referenced_filename`, `product_name`,
`product_slug`, `category`, `width`, `height`, `bytes`, `md5`, `source_url`,
`github_raw_url`, `full_size_substituted`).

| # | Category | Product | Stored file | Original filename | Size | Source URL |
|---:|---|---|---|---|---:|---|
| 1 | Zestawy lunchowe | Mini lunch | `mini-lunch.jpg` | `lunch-vege.jpg` | 705,886 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2024/04//lunch-vege.jpg |
| 2 | Zestawy | Zestaw dla 2 osób | `zestaw-dla-2-osob.jpg` | `zestaw_1_new_2024.jpg` | 240,970 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2020/11/zestaw_1_new_2024.jpg |
| 3 | Zestawy | Zestaw dla 4 osób | `zestaw-dla-4-osob.jpg` | `zestaw_2_new_2024.jpg` | 326,910 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2020/11/zestaw_2_new_2024.jpg |
| 4 | Zestawy | Zestaw dla 5 osób | `zestaw-dla-5-osob.jpg` | `zestaw_3_new_2024.jpg` | 355,505 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2020/11/zestaw_3_new_2024.jpg |
| 5 | Zupy | 01. Dal soup | `01-dal-soup.jpg` | `dal-soup-1.jpg` | 84,360 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/dal-soup-1.jpg |
| 6 | Zupy | 02. Mushroom Palak Soup | `02-mushroom-palak-soup.jpg` | `palak-soup.jpg` | 65,167 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/palak-soup.jpg |
| 7 | Zupy | 03. Tomato soup | `03-tomato-soup.jpg` | `tomato-soup.jpg` | 89,239 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/tomato-soup.jpg |
| 8 | Zupy | 04. Chicken Soup | `04-chicken-soup.jpg` | `chicken-soup.jpg` | 84,571 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/chicken-soup.jpg |
| 9 | Zupy | 05. Mutton soup | `05-mutton-soup.jpg` | `mutton-soup.jpg` | 81,486 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/mutton-soup.jpg |
| 10 | Sałatki | 06. Green Salad | `06-green-salad.jpg` | `salad.jpg` | 91,447 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/salad.jpg |
| 11 | Sałatki | 07. Brinjol And Cheese Salad | `07-brinjol-and-cheese-salad.jpg` | `baklazan.jpg` | 73,314 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/baklazan.jpg |
| 12 | Sałatki | 08. Chicken Tikka Salad 🌶️ | `08-chicken-tikka-salad.jpg` | `chicken-salad.jpg` | 77,236 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/chicken-salad.jpg |
| 13 | Przystawki wegetariańskie | 09. Samosa (2 szt.) | `09-samosa-2-szt.jpg` | `samosa-1.jpg` | 60,391 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/samosa-1.jpg |
| 14 | Przystawki wegetariańskie | 10. Paneer Pakora | `10-paneer-pakora.jpg` | `paneer-pakora.jpg` | 71,276 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/paneer-pakora.jpg |
| 15 | Przystawki wegetariańskie | 11. Mix Veg. Pakora | `11-mix-veg-pakora.jpg` | `vege-pakora.jpg` | 87,866 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/vege-pakora.jpg |
| 16 | Przystawki wegetariańskie | 12. Onion Bhajia | `12-onion-bhajia.jpg` | `onion.jpg` | 73,028 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/onion.jpg |
| 17 | Przystawki wegetariańskie | 13. Chat Patta Paneer 🌶️ | `13-chat-patta-paneer.jpg` | `chat-patta.jpg` | 68,114 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/chat-patta.jpg |
| 18 | Przystawki wegetariańskie | 14. Paneer Tikka | `14-paneer-tikka.jpg` | `paneer-tikka.jpg` | 83,938 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/paneer-tikka.jpg |
| 19 | Przystawki mięsne | 15. Chicken/Mutton Samosa | `15-chickenmutton-samosa.jpg` | `samosa-1-1.jpg` | 60,391 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/samosa-1-1.jpg |
| 20 | Przystawki mięsne | 16. Chicken Tikka | `16-chicken-tikka.jpg` | `chicken-tikka.jpg` | 87,202 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/chicken-tikka.jpg |
| 21 | Przystawki mięsne | 17. Chicken Hariyali Tikka | `17-chicken-hariyali-tikka.jpg` | `haryali-1.jpg` | 90,547 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/haryali-1.jpg |
| 22 | Przystawki mięsne | 18. Garlic Chicken Tikka | `18-garlic-chicken-tikka.jpg` | `garlic-chicken.jpg` | 95,628 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/garlic-chicken.jpg |
| 23 | Przystawki mięsne | 19. Murgh Malai Kebeb | `19-murgh-malai-kebeb.jpg` | `garlic-chicken-1.jpg` | 95,628 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/garlic-chicken-1.jpg |
| 24 | Przystawki mięsne | 20. Tandoori Chicken | `20-tandoori-chicken.jpg` | `chicken-tandoori.jpg` | 84,111 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/chicken-tandoori.jpg |
| 25 | Dania wegetariańskie | 21. Dal Tadka | `21-dal-tadka.jpg` | `dal-tadka-1.jpg` | 70,992 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/dal-tadka-1.jpg |
| 26 | Dania wegetariańskie | 22. Dal Makhni | `22-dal-makhni.jpg` | `dal-makhni-1.jpg` | 74,959 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/dal-makhni-1.jpg |
| 27 | Dania wegetariańskie | 23. Veg shahi Korma 🍯 | `23-veg-shahi-korma.jpg` | `korma-1.jpg` | 69,629 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/korma-1.jpg |
| 28 | Dania wegetariańskie | 24. Mixed Vegetable Curry | `24-mixed-vegetable-curry.jpg` | `mix-veg-curry.jpg` | 97,901 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/mix-veg-curry.jpg |
| 29 | Dania wegetariańskie | 25. Chana Masala | `25-chana-masala.jpg` | `chana-masala-1.jpg` | 74,648 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/chana-masala-1.jpg |
| 30 | Dania wegetariańskie | 26. Aloo masala | `26-aloo-masala.jpg` | `aloo-masala-1.jpg` | 70,091 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/aloo-masala-1.jpg |
| 31 | Dania wegetariańskie | 27. Aloo Ghobi Bhaji | `27-aloo-ghobi-bhaji.jpg` | `ghobi.jpg` | 103,699 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/ghobi.jpg |
| 32 | Dania wegetariańskie | 28. Aloo Bhindi Bhaji | `28-aloo-bhindi-bhaji.jpg` | `aloo-bhindi.jpg` | 91,100 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/aloo-bhindi.jpg |
| 33 | Dania wegetariańskie | 29. Dahiwala Baingan | `29-dahiwala-baingan.jpg` | `dachiwala.jpg` | 86,996 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/dachiwala.jpg |
| 34 | Dania wegetariańskie | 30. Paneer Tikka Masala 🌶️ | `30-paneer-tikka-masala.jpg` | `chicken-tikka-masala.jpg` | 69,156 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/chicken-tikka-masala.jpg |
| 35 | Dania wegetariańskie | 31. Palak Paneer | `31-palak-paneer.jpg` | `palak-paneer-1-1.jpg` | 71,544 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/palak-paneer-1-1.jpg |
| 36 | Dania wegetariańskie | 32. Paneer Makhni | `32-paneer-makhni.jpg` | `paneer-makhni-1.jpg` | 71,257 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/paneer-makhni-1.jpg |
| 37 | Dania wegetariańskie | 33. Garlic Paneer | `33-garlic-paneer.jpg` | `garlic-paneer.jpg` | 59,858 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/garlic-paneer.jpg |
| 38 | Dania wegetariańskie | 34. Methi Paneer | `34-methi-paneer.jpg` | `methi-chicken.jpg` | 81,963 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/methi-chicken.jpg |
| 39 | Dania mięsne | 35. Chicken Curry | `35-chicken-curry.jpg` | `chicken-curry-1.jpg` | 100,225 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/chicken-curry-1.jpg |
| 40 | Dania mięsne | 36. Butter Chicken 🍯 | `36-butter-chicken.jpg` | `butter-chicken-1.jpg` | 86,971 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/butter-chicken-1.jpg |
| 41 | Dania mięsne | 37. Chicken Tikka Masala 🌶️ | `37-chicken-tikka-masala.jpg` | `chicken-tikka-masala-1.jpg` | 69,156 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/chicken-tikka-masala-1.jpg |
| 42 | Dania mięsne | 38. Methi Chicken | `38-methi-chicken.jpg` | `methi-chicken-1.jpg` | 81,963 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/methi-chicken-1.jpg |
| 43 | Dania mięsne | 39. Chicken Korma 🍯 | `39-chicken-korma.jpg` | `korma-1-1.jpg` | 69,629 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/korma-1-1.jpg |
| 44 | Dania mięsne | 40. Chicken Palak | `40-chicken-palak.jpg` | `palak-paneer-1-1-1.jpg` | 71,544 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/palak-paneer-1-1-1.jpg |
| 45 | Dania mięsne | 41. Chicken Jalfreiji 🌶️ | `41-chicken-jalfreiji.jpg` | `jalfreji.jpg` | 88,897 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/jalfreji.jpg |
| 46 | Dania mięsne | 42. Ginger Chicken 🌶️ | `42-ginger-chicken.jpg` | `ginger-chicken.jpg` | 73,471 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/ginger-chicken.jpg |
| 47 | Dania mięsne | 43. Chicken Madras 🌶️🌶️🌶️ | `43-chicken-madras.jpg` | `mutton-madras.jpg` | 73,671 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/mutton-madras.jpg |
| 48 | Dania mięsne | 44. Chicken Vindaloo 🌶️🌶️🌶️🌶️ | `44-chicken-vindaloo.jpg` | `vindaloo-1.jpg` | 82,535 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/vindaloo-1.jpg |
| 49 | Dania mięsne | 45. Mutton Curry | `45-mutton-curry.jpg` | `mutton-curry.jpg` | 87,879 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/mutton-curry.jpg |
| 50 | Dania mięsne | 46. Mutton Korma 🍯 | `46-mutton-korma.jpg` | `korma.jpg` | 69,629 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/korma.jpg |
| 51 | Dania mięsne | 47. Mutton Palak | `47-mutton-palak.jpg` | `palak-paneer-1-1-2.jpg` | 71,544 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/palak-paneer-1-1-2.jpg |
| 52 | Dania mięsne | 48. Mutton Rogan Josh 🌶️ | `48-mutton-rogan-josh.jpg` | `rogan-josh.jpg` | 79,055 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/rogan-josh.jpg |
| 53 | Dania mięsne | 49. Mutton Madras 🌶️🌶️🌶️ | `49-mutton-madras.jpg` | `mutton-madras-1.jpg` | 73,671 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/mutton-madras-1.jpg |
| 54 | Dania mięsne | 50. Kadai Mutton 🌶️🌶️ | `50-kadai-mutton.jpg` | `kadai-2.jpg` | 102,230 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/kadai-2.jpg |
| 55 | Dania mięsne | 51. Dal Gosht 🌶️🌶️ | `51-dal-gosht.jpg` | `dal-ghosh.jpg` | 88,231 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/dal-ghosh.jpg |
| 56 | Dania mięsne | 52. Bhuna Gosht 🌶️🌶️ | `52-bhuna-gosht.jpg` | `bhuna-gosh.jpg` | 69,759 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/bhuna-gosh.jpg |
| 57 | Dania mięsne | 53. Chicken Do Piyaza | `53-chicken-do-piyaza.jpg` | `pyaza.jpg` | 92,794 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/pyaza.jpg |
| 58 | Dania mięsne | 54. Chicken Hydaerabadi 🌶️🌶️ | `54-chicken-hydaerabadi.jpg` | `Hydaerabadi.jpg` | 108,390 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/Hydaerabadi.jpg |
| 59 | Dania mięsne | 55. Murgh Pasanda | `55-murgh-pasanda.jpg` | `passanda.jpg` | 101,132 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/passanda.jpg |
| 60 | Dania mięsne | 56. Balti Chicken / Mutton 🌶️🌶️ | `56-balti-chicken-mutton.jpg` | `balti-1.jpg` | 82,574 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/balti-1.jpg |
| 61 | Dania mięsne | 57. Mutton Keema Mattar 🌶️🌶️ | `57-mutton-keema-mattar.jpg` | `mutton-keema-matar.jpg` | 87,838 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/mutton-keema-matar.jpg |
| 62 | Dania mięsne | 58. Ginger Mutton 🌶️ | `58-ginger-mutton.jpg` | `ginger-mutton.jpg` | 81,905 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/ginger-mutton.jpg |
| 63 | Dania mięsne | 59. Mutton Lal Massala 🌶️🌶️ | `59-mutton-lal-massala.jpg` | `lal-masala.jpg` | 76,638 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/lal-masala.jpg |
| 64 | Indian Sizzler | 60. Vegetable Sizzler 🌶️ | `60-vegetable-sizzler.jpg` | `veg-sizzler.jpg` | 88,188 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/veg-sizzler.jpg |
| 65 | Indian Sizzler | 61. Chicken Sizzler 🌶️ | `61-chicken-sizzler.jpg` | `chicken-sizzler.jpg` | 87,535 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/chicken-sizzler.jpg |
| 66 | Indian Sizzler | 62. Mutton Sizzler 🌶️ | `62-mutton-sizzler.jpg` | `mutton-sizzler.jpg` | 87,101 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/mutton-sizzler.jpg |
| 67 | Ryby i owoce morza | 63. Fish Curry | `63-fish-curry.jpg` | `fish-curry.jpg` | 107,273 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/fish-curry.jpg |
| 68 | Ryby i owoce morza | 64. Fish Khada Masala 🌶️🌶️ | `64-fish-khada-masala.jpg` | `fish-khada.jpg` | 103,787 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/fish-khada.jpg |
| 69 | Ryby i owoce morza | 65. Lasooni Macchi 🌶️🌶️ | `65-lasooni-macchi.jpg` | `lasooni.jpg` | 95,016 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/lasooni.jpg |
| 70 | Ryby i owoce morza | 66. Prawn Curry | `66-prawn-curry.jpg` | `prawn-curry.jpg` | 107,640 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/prawn-curry.jpg |
| 71 | Ryby i owoce morza | 67. Prawn Chili Garlic Masala 🌶️🌶️🌶️ | `67-prawn-chili-garlic-masala.jpg` | `prawn-chilli.jpg` | 90,201 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/prawn-chilli.jpg |
| 72 | Ryby i owoce morza | 68. Prawn Jalfreji 🌶️ | `68-prawn-jalfreji.jpg` | `prawn-jalfreji.jpg` | 74,119 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/prawn-jalfreji.jpg |
| 73 | Ryż i dania z ryżu | 69. Plain Rice | `69-plain-rice.jpg` | `jerra.jpg` | 67,506 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/jerra.jpg |
| 74 | Ryż i dania z ryżu | 70. Jerra Rice | `70-jerra-rice.jpg` | `jerra-1.jpg` | 67,506 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/jerra-1.jpg |
| 75 | Ryż i dania z ryżu | 71. Safron Rice | `71-safron-rice.jpg` | `saffron.jpg` | 70,440 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/saffron.jpg |
| 76 | Ryż i dania z ryżu | 72. Lemon Rice | `72-lemon-rice.jpg` | `lemon-rice.jpg` | 86,951 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/lemon-rice.jpg |
| 77 | Ryż i dania z ryżu | 73. Methi Pulao | `73-methi-pulao.jpg` | `methi-pulao.jpg` | 115,403 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/methi-pulao.jpg |
| 78 | Ryż i dania z ryżu | 74. Matar Pulao | `74-matar-pulao.jpg` | `matar-pulao.jpg` | 78,246 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/matar-pulao.jpg |
| 79 | Ryż i dania z ryżu | 75. Kashmiri Pulao 🍯 | `75-kashmiri-pulao.jpg` | `pulao.jpg` | 87,167 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/pulao.jpg |
| 80 | Ryż i dania z ryżu | 76. Vegetable Biryani 🌶️ | `76-vegetable-biryani.jpg` | `biryani.jpg` | 111,485 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/biryani.jpg |
| 81 | Ryż i dania z ryżu | 77. Chicken Biryani 🌶️ | `77-chicken-biryani.jpg` | `biryani-1.jpg` | 111,485 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/biryani-1.jpg |
| 82 | Ryż i dania z ryżu | 78. Mutton Biryani 🌶️ | `78-mutton-biryani.jpg` | `biryani-2.jpg` | 111,485 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/biryani-2.jpg |
| 83 | Placki z pieca Tandoor | 79. Tandoori Roti | `79-tandoori-roti.jpg` | `rotti.jpg` | 90,520 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/rotti.jpg |
| 84 | Placki z pieca Tandoor | 80. Methi Roti | `80-methi-roti.jpg` | `methiroti.jpg` | 120,457 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/methiroti.jpg |
| 85 | Placki z pieca Tandoor | 81. Plain Naan | `81-plain-naan.jpg` | `naan-1.jpg` | 85,463 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/naan-1.jpg |
| 86 | Placki z pieca Tandoor | 82. Butter Naan | `82-butter-naan.jpg` | `naan-1-1.jpg` | 85,463 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/naan-1-1.jpg |
| 87 | Placki z pieca Tandoor | 83. Garlic Naan | `83-garlic-naan.jpg` | `naan-1-2.jpg` | 85,463 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/naan-1-2.jpg |
| 88 | Placki z pieca Tandoor | 84. Moti Naan | `84-moti-naan.jpg` | `moti-naan.jpg` | 91,274 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/moti-naan.jpg |
| 89 | Placki z pieca Tandoor | 85. Lachha Paratha | `85-lachha-paratha.jpg` | `laacha.jpg` | 81,921 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/laacha.jpg |
| 90 | Placki z pieca Tandoor | 86. Aloo Paratha | `86-aloo-paratha.jpg` | `aloo-paratha.jpg` | 94,606 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/aloo-paratha.jpg |
| 91 | Placki z pieca Tandoor | 87. Paneer Naan 🍯 | `87-paneer-naan.jpg` | `paneer-naan.jpg` | 87,530 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/paneer-naan.jpg |
| 92 | Placki z pieca Tandoor | 88. Keema Naan 🌶️ | `88-keema-naan.jpg` | `paneer-naan-1.jpg` | 87,530 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/paneer-naan-1.jpg |
| 93 | Placki z pieca Tandoor | 89. Chicken Keema Naan | `89-chicken-keema-naan.jpg` | `paneer-naan-2.jpg` | 87,530 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/paneer-naan-2.jpg |
| 94 | Placki z pieca Tandoor | 90. Kashmiri Naan 🍯 | `90-kashmiri-naan.jpg` | `paneer-naan-3.jpg` | 87,530 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/paneer-naan-3.jpg |
| 95 | Dodatki | 91. Roasted Papad | `91-roasted-papad.jpg` | `pappadam.jpg` | 81,766 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/pappadam.jpg |
| 96 | Dodatki | 92. Plain Curd | `92-plain-curd.jpg` | `raita-1.jpg` | 50,291 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/raita-1.jpg |
| 97 | Dodatki | 93. Raita | `93-raita.jpg` | `raita-1-1.jpg` | 50,291 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/raita-1-1.jpg |
| 98 | Desery | 94. Gulab Jamun | `94-gulab-jamun.jpg` | `gulab.jpg` | 54,802 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/gulab.jpg |
| 99 | Indyjskie Napoje | 100. Sweet Or Salty Lassi | `100-sweet-or-salty-lassi.jpg` | `lassi.jpg` | 42,268 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/lassi.jpg |
| 100 | Indyjskie Napoje | 101. Mango Lassi | `101-mango-lassi.jpg` | `mango-lassi.jpg` | 47,788 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/mango-lassi.jpg |
| 101 | Indyjskie Napoje | 96. Ginger Lemon Drink | `96-ginger-lemon-drink.jpg` | `ginger-lemon.jpg` | 49,161 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/ginger-lemon.jpg |
| 102 | Indyjskie Napoje | 97. Mango Juice | `97-mango-juice.jpg` | `mango-juice.jpg` | 45,983 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/mango-juice.jpg |
| 103 | Indyjskie Napoje | 98. Lychee Juice | `98-lychee-juice.jpg` | `lychee-1.jpg` | 55,822 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/lychee-1.jpg |
| 104 | Indyjskie Napoje | 99. Guava Juice | `99-guava-juice.jpg` | `guava-1.jpg` | 45,633 B | https://arkuszowa.curry-house.pl/wp-content/uploads/2022/06/guava-1.jpg |

**Coverage: 104 / 104 products mapped. 0 unmapped files.**

---

## 8. Slider-to-image mapping

Source of truth: `database/seeders/SliderSeeder.php`, which defines 3 slides with
`sort_order` 1-3 and an `image_path` pointing into `products/`.

| sort_order | Slider title | DB `image_path` | Stored slider file | Derived from product | Original filename | Size |
|---:|---|---|---|---|---|---:|
| 1 | Autentyczna kuchnia indyjska | `products/36-butter-chicken.jpg` | `sliders/36-butter-chicken.jpg` | 36. Butter Chicken 🍯 | `butter-chicken-1.jpg` | 86,971 B |
| 2 | Zestawy lunchowe | `products/61-chicken-sizzler.jpg` | `sliders/61-chicken-sizzler.jpg` | 61. Chicken Sizzler 🌶️ | `chicken-sizzler.jpg` | 87,535 B |
| 3 | Dostawa i odbiór osobisty | `products/08-chicken-tikka-salad.jpg` | `sliders/08-chicken-tikka-salad.jpg` | 08. Chicken Tikka Salad 🌶️ | `chicken-salad.jpg` | 77,236 B |

The machine-readable version is **`sliders/mapping.csv`**.

**Coverage: 3 / 3 slider records mapped. 0 unmapped files.**

---

## 9. Comparison with the images already inside the application

The platform already has 104 copies of these images in
`restaurant-platform/storage/app/public/products/`, written by `MenuSeeder`.
Those files were **only read**, never modified. Comparison result:

| Comparison | Count |
|---|---:|
| Byte-identical to the new collection | 74 |
| Differ (new file is the better master) | 30 |
| Missing locally | 0 |

All 30 differences are improvements, not regressions:

* **29 files** — the local copy is a **lossy WebP re-encode** stored under a `.jpg`
  name (a side effect of `MenuSeeder::downloadImage()` advertising `image/webp` in
  its `Accept` header). These are now the true JPEG masters, typically 3-5× larger.
* **1 file** (`mini-lunch.jpg`) — the local copy is a **120×120 thumbnail**; the
  repository now holds the 3240×3240 master.

| File | Local copy | In this repository |
|---|---|---|
| `01-dal-soup.jpg` | WebP, 17,954 B | JPEG, 500×500, 84,360 B |
| `06-green-salad.jpg` | WebP, 20,824 B | JPEG, 500×500, 91,447 B |
| `100-sweet-or-salty-lassi.jpg` | WebP, 3,416 B | JPEG, 500×500, 42,268 B |
| `11-mix-veg-pakora.jpg` | WebP, 20,044 B | JPEG, 500×500, 87,866 B |
| `16-chicken-tikka.jpg` | WebP, 19,014 B | JPEG, 500×500, 87,202 B |
| `17-chicken-hariyali-tikka.jpg` | WebP, 23,944 B | JPEG, 500×500, 90,547 B |
| `18-garlic-chicken-tikka.jpg` | WebP, 24,388 B | JPEG, 500×500, 95,628 B |
| `23-veg-shahi-korma.jpg` | WebP, 12,430 B | JPEG, 500×500, 69,629 B |
| `27-aloo-ghobi-bhaji.jpg` | WebP, 25,950 B | JPEG, 500×500, 103,699 B |
| `36-butter-chicken.jpg` | WebP, 16,930 B | JPEG, 500×500, 86,971 B |
| `38-methi-chicken.jpg` | WebP, 17,506 B | JPEG, 500×500, 81,963 B |
| `39-chicken-korma.jpg` | WebP, 12,430 B | JPEG, 500×500, 69,629 B |
| `41-chicken-jalfreiji.jpg` | WebP, 21,574 B | JPEG, 500×500, 88,897 B |
| `44-chicken-vindaloo.jpg` | WebP, 16,426 B | JPEG, 500×500, 82,535 B |
| `47-mutton-palak.jpg` | WebP, 13,554 B | JPEG, 500×500, 71,544 B |
| `49-mutton-madras.jpg` | WebP, 14,826 B | JPEG, 500×500, 73,671 B |
| `52-bhuna-gosht.jpg` | WebP, 13,504 B | JPEG, 500×500, 69,759 B |
| `60-vegetable-sizzler.jpg` | WebP, 19,776 B | JPEG, 500×500, 88,188 B |
| `65-lasooni-macchi.jpg` | WebP, 23,592 B | JPEG, 500×500, 95,016 B |
| `66-prawn-curry.jpg` | WebP, 30,658 B | JPEG, 500×500, 107,640 B |
| `68-prawn-jalfreji.jpg` | WebP, 15,012 B | JPEG, 500×500, 74,119 B |
| `71-safron-rice.jpg` | WebP, 13,336 B | JPEG, 500×500, 70,440 B |
| `78-mutton-biryani.jpg` | WebP, 31,110 B | JPEG, 500×500, 111,485 B |
| `94-gulab-jamun.jpg` | WebP, 8,858 B | JPEG, 500×500, 54,802 B |
| `96-ginger-lemon-drink.jpg` | WebP, 5,614 B | JPEG, 500×500, 49,161 B |
| `99-guava-juice.jpg` | WebP, 4,160 B | JPEG, 500×500, 45,633 B |
| `mini-lunch.jpg` | WebP, 2,768 B | JPEG, 3240×3240, 705,886 B |
| `zestaw-dla-2-osob.jpg` | WebP, 121,228 B | JPEG, 1500×1500, 240,970 B |
| `zestaw-dla-4-osob.jpg` | WebP, 180,366 B | JPEG, 1500×1500, 326,910 B |
| `zestaw-dla-5-osob.jpg` | WebP, 200,904 B | JPEG, 1500×1500, 355,505 B |

Nothing in the application was changed to make this happen — replacing the local
copies or pointing `image_path` at these URLs is a **separate, future task** that
was explicitly out of scope here.

---

## 10. Uncertainties requiring manual review

| # | Item | Risk | Suggested action |
|---:|---|---|---|
| 1 | **%d product photos are byte-identical duplicates of another product** (§6.2). Live on the source site today. | Customers see the same dish photo under two different menu items, e.g. *Veg shahi Korma* / *Chicken Korma* / *Mutton Korma*. | Ask the restaurant for the correct photo per dish, then re-download that single URL. Do **not** auto-substitute. |
| 2 | The source site has **no slider** (§2), so the 3 slider images are product photos chosen to match the 3 `Slider` records. | If dedicated banner art was expected, it does not exist upstream. | Confirm the 3 slides are acceptable, or supply/commission banner artwork. |
| 3 | `curry.jpg` (1500 px Open Graph hero) and `top-table.jpg` were found but excluded. | They may be the intended hero imagery. | Confirm whether to add them to `sliders/` or a `banners/` folder. |
| 4 | *Mini lunch* (`lunch-vege.jpg`, 3240×3240) is a **vegetarian** lunch photo; the live product is "w małej wersji mięsnej lub wegetariańskiej". | The photo may not represent the meat variant. | Confirm with the restaurant; the URL is recorded in `products/mapping.csv` so it is trivially swappable. |
| 5 | Filenames use the platform's slug convention, not the site's originals. | Anyone comparing directories directly will see different names. | `products/mapping.csv` holds the original filename and source URL for every file. |
| 6 | Product numbering on the site skips `95`. | Cosmetic data gap inherited from the source menu. | Confirm nothing should exist between 94 and 96. |
| 7 | The source site can re-encode on the fly (`Vary: Accept`). | A future re-download with a WebP `Accept` header would silently degrade quality again. | Always request `Accept: image/jpeg,image/png,image/*;q=0.8`; never advertise webp/avif. |

---

## 11. Repository layout

```
restaurant-platform-images/
├── README.md
├── IMAGE_EXTRACTION_REPORT.md      <- this file
├── products/
│   ├── mapping.csv                 <- original filename <-> product <-> source URL
│   └── 104 *.jpg
└── sliders/
    ├── mapping.csv
    └── 3 *.jpg
```

Consuming the files from the application:

```
https://raw.githubusercontent.com/vkprajapati/restaurant-platform-images/Master/products/36-butter-chicken.jpg
```

---

## 12. GitHub upload status

| | |
|---|---|
| Repository | https://github.com/vkprajapati/restaurant-platform-images |
| Visibility | public |
| Default branch | `Master` |
| Remote URL | `https://github.com/vkprajapati/restaurant-platform-images.git` |
| State before this task | existed, contained only `README.md`, **no local clone anywhere on this machine** |
| Local working copy | `/Applications/XAMPP/xamppfiles/htdocs/restaurant-platform-images` |
| Application repo | untouched — nothing committed or pushed to `restaurant-platform` |

