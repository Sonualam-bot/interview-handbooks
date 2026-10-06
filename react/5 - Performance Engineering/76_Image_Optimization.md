# Handbook 5 — React Performance Engineering

# Chapter 76 — Image Optimization

## 1. Why Images Matter for Performance

Images are often some of the largest resources on a web page.

A page might contain:

- hero images
- product images
- avatars
- thumbnails
- banners
- background images
- icons

Unnecessarily large images can increase:

- network transfer
- loading time
- memory usage
- decoding work
- rendering work

Core principle:

> **Don't download significantly more image data than the user needs.**

---

## 2. The Core Problem

Suppose an image is displayed at:

```text
400 × 300
```

but the application downloads:

```text
2400 × 1800
```

The browser receives far more data than the displayed image needs.

Ideally, the server/CDN should provide an appropriately sized version.

Common approaches:

- responsive image sizes
- image CDNs
- automatic image transformation
- thumbnails
- multiple image resolutions

---

## 3. Image Dimensions Matter

If an image is displayed at:

```text
400 × 300
```

serving a:

```text
2000 × 1500
```

version may waste bandwidth.

The goal is to serve an image appropriate for the rendered size and device.

---

## 4. Modern Image Formats

### JPEG

Good for:

- photographs
- complex images

### PNG

Useful when:

- lossless quality is important
- transparency is required

### WebP

Provides good compression and broad modern web support.

### AVIF

Can provide highly efficient compression and is useful for modern image delivery.

Interview point:

> **Choose an appropriate image format and compression level instead of serving unnecessarily large files.**

---

## 5. Compression

Compression reduces image file size.

### Lossless compression

Preserves image information exactly.

### Lossy compression

Removes some information to achieve a smaller file.

For many photographic images, lossy compression can dramatically reduce file size while maintaining visually acceptable quality.

The goal is:

```text
Good visual quality
+
Reasonable file size
```

Not maximum compression at any cost.

---

## 6. Responsive Images

Different devices have different screen sizes and pixel densities.

Serving the same huge image to every device is inefficient.

HTML provides `srcset`:

```html
<img
  src="image-800.jpg"
  srcset="
    image-400.jpg 400w,
    image-800.jpg 800w,
    image-1200.jpg 1200w
  "
  alt="Product"
/>
```

The browser can select an appropriate resource based on the available conditions.

This helps avoid downloading unnecessarily large images.

---

## 7. `sizes`

When using `srcset` with width descriptors, `sizes` can tell the browser how large the image is expected to appear.

```html
<img
  src="image-800.jpg"
  srcset="
    image-400.jpg 400w,
    image-800.jpg 800w,
    image-1200.jpg 1200w
  "
  sizes="(max-width: 600px) 100vw, 50vw"
  alt="Product"
/>
```

Conceptually:

```text
Viewport width
      +
Expected rendered image size
      ↓
Choose appropriate source
```

---

## 8. Lazy Loading

Images below the initial viewport often don't need to load immediately.

Native lazy loading:

```html
<img
  src="photo.jpg"
  loading="lazy"
  alt="Example"
/>
```

Useful for:

- long feeds
- large galleries
- lists
- images far below the fold

Lazy loading reduces unnecessary initial work.

---

## 9. Don't Lazy-Load Everything

### Above-the-fold / critical image

Usually should load quickly.

### Below-the-fold image

Can often be lazy-loaded.

If the main hero image is lazy-loaded unnecessarily, an important visual resource may be delayed.

Therefore:

> **Lazy-load non-critical images, not blindly every image.**

---

## 10. Image Dimensions Prevent Layout Shift

If the browser doesn't know an image's dimensions, its loading can cause surrounding content to move.

Providing dimensions reserves space:

```html
<img
  src="photo.jpg"
  width="800"
  height="600"
  alt="Example"
/>
```

This helps prevent **layout shift**.

---

## 11. Aspect Ratio

CSS can preserve the expected shape:

```css
.image {
  aspect-ratio: 4 / 3;
}
```

Example:

```css
.card-image {
  width: 100%;
  aspect-ratio: 16 / 9;
  object-fit: cover;
}
```

This helps the browser reserve appropriate space before the image is fully available.

---

## 12. `object-fit`

### `cover`

```css
img {
  object-fit: cover;
}
```

Fills the container while preserving aspect ratio, potentially cropping part of the image.

### `contain`

```css
img {
  object-fit: contain;
}
```

Keeps the entire image visible, potentially leaving empty space.

---

## 13. Image CDN

A CDN can transform and deliver images closer to the user.

```text
Original image
      ↓
Resize
      ↓
Compress
      ↓
Convert format
      ↓
Deliver through CDN
```

An application may request:

```text
width = 800
quality = appropriate
format = WebP/AVIF
```

instead of serving one enormous original file to everyone.

---

## 14. Thumbnails

Image-heavy applications should often use thumbnails.

```text
Original:
4000 × 3000

List thumbnail:
200 × 150

Detail page:
1200 × 900
```

A list doesn't need the full-resolution image.

Useful for:

- ecommerce
- galleries
- social feeds
- file managers
- search results

---

## 15. Preloading Critical Images

Sometimes an important image should be loaded early.

For example, a major hero/LCP image may be important to the initial page experience.

The browser can be given stronger loading hints for critical resources.

Preloading should be selective because too much preloading can compete for bandwidth with other critical resources.

---

## 16. React Does Not Automatically Optimize Images

React is primarily responsible for UI rendering.

Writing:

```jsx
<img src="/large-image.jpg" />
```

does not automatically mean:

- image is compressed
- image is resized
- best format is selected
- image is lazy-loaded
- image is served from a CDN

Those concerns are handled by the browser, framework, build pipeline, image service, or application architecture.

---

## 17. Image Optimization in React

A simple React image:

```jsx
function ProductImage({ product }) {
  return (
    <img
      src={product.image}
      alt={product.name}
      loading="lazy"
      width={400}
      height={300}
    />
  );
}
```

This addresses some concerns:

- meaningful `alt`
- lazy loading
- explicit dimensions

But the image delivery strategy still matters.

---

## 18. Next.js Image Optimization

In Next.js:

```jsx
import Image from "next/image";

<Image
  src="/product.jpg"
  alt="Product"
  width={800}
  height={600}
/>
```

The framework can help with image sizing, optimization, and responsive delivery depending on configuration and deployment.

The deeper interview concept is:

```text
Image optimization
=
right size
+
right format
+
right loading strategy
+
stable layout
+
efficient delivery
```

---

## 19. Images and Core Web Vitals

Images can affect important performance metrics.

### LCP

A large hero image can be the **Largest Contentful Paint** element.

```text
Large image
   ↓
Slow download/decode
   ↓
Delayed LCP
```

### CLS

Missing image dimensions can cause layout shifts.

```text
Image space unknown
      ↓
Image loads
      ↓
Content moves
      ↓
CLS increases
```

Image optimization is therefore not only about file size.

---

## 20. Common Mistakes

### Mistake 1 — Serving huge originals

```text
400px UI
↓
4000px image downloaded
```

### Mistake 2 — No lazy loading for long lists

Thousands of below-the-fold images may be requested unnecessarily.

### Mistake 3 — Lazy-loading the hero image

The most important image can become slower.

### Mistake 4 — No dimensions

Can cause layout shifts.

### Mistake 5 — Using one image size everywhere

Mobile and desktop may have different requirements.

### Mistake 6 — Ignoring image format

Older/larger formats may produce unnecessarily large files.

### Mistake 7 — Optimizing without measuring

Always consider actual network and rendering performance.

---

## 21. Interview Scenario

### Question

> You have an ecommerce page with 100 product cards. Each product has a 3000×2000 image. The page is slow. What would you do?

Strong answer:

```text
1. Measure network/image cost
2. Serve appropriately sized images
3. Use modern compressed formats
4. Generate thumbnails
5. Lazy-load below-the-fold images
6. Keep critical/hero images prioritized
7. Provide dimensions/aspect ratio
8. Use CDN/image transformation where appropriate
9. Measure again
```

This demonstrates that image optimization is a system rather than one trick.

---

## 22. Image Optimization Mental Model

Ask four questions:

```text
1. WHAT size should the user receive?
2. WHAT format should be used?
3. WHEN should the image load?
4. HOW should its layout space be reserved?
```

Therefore:

```text
Right size
    +
Right format
    +
Right loading priority
    +
Stable dimensions
    ↓
Better image performance
```

---

## 23. Golden Rules

1. Don't serve unnecessarily large images.
2. Resize images according to their rendered size.
3. Use appropriate compression.
4. Consider modern formats such as WebP and AVIF.
5. Use responsive images when image sizes vary across devices.
6. Lazy-load non-critical images.
7. Don't blindly lazy-load critical above-the-fold images.
8. Reserve image space with dimensions/aspect ratio.
9. Use thumbnails for lists and galleries.
10. Use CDNs/image transformation for large-scale applications.
11. Consider LCP for important images.
12. Consider CLS when image dimensions are unknown.
13. React itself does not automatically solve image delivery.
14. Measure before and after optimization.

---

## 24. Quick Interview Summary

```text
Image Performance
       ↓
Appropriate dimensions
       +
Compression / modern format
       +
Responsive delivery
       +
Lazy loading where appropriate
       +
Reserved layout space
       +
CDN / optimized delivery
       ↓
Faster + more stable UI
```

### One-line interview answer

> **Image optimization means delivering the right image size and format at the right time while reserving its layout space, so we reduce network/rendering cost without hurting visual quality or critical page performance.**
