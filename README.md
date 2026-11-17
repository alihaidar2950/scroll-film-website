# Scroll-film website with Claude + Kling: free guide

> Every number here is from a real client build: a website for a Lebanese sweets shop in Perth.
> A runnable version of the code is in [`example/index.html`](example/index.html).

---

## بالعربي

الموقع بيبلّش بفيلم، وكل ما تنزل بالـscroll الفيلم بيمشي معك frame بـframe. **Kling** عمل الفيلم،
و**Claude Code** كتب الموقع كلّو.

### ١. القصة أوّل شي
اكتب ٤–٥ لقطات بتحكي قصة البزنس، مش منتج عشوائي. نحنا عملنا: الأرزة اللي عاللوغو بالضباب، بعدين الفستق،
بعدين ورق العجين، بعدين القطر.
**قاعدة مهمة:** ما تخلّي الـAI يعمل المنتج نفسو (الحلو، العلبة، المحل، الناس). الـAI بيعمل الجو
والمكوّنات بس، والمنتج الحقيقي بيطلع بصور الزبون الحقيقية بعد الفيلم. واكتب عالموقع إنو الفيلم معمول بالـAI.

### ٢. صورة لكل لقطة
اعمل صورة وحدة لكل لقطة (keyframe) بنفس الإضاءة ونفس الألوان. نحنا عملنا ٥.
**آخر صورة لازم يكون لونها نفس لون خلفية الموقع** (عنّا أخضر غامق `#0a1a11`)، هيك الفيلم بيذوب بالصفحة.

### ٣. الفيديو بـKling
- **الطريقة:** image-to-video، ٥ ثواني لكل clip. أعطيه صورة البداية وصورة النهاية (اللي هي بداية الـclip اللي بعدو)، هيك الـclips بيتركّبوا ورا بعض بلا قطعة.
- **حركة الكاميرا:** بطيئة. الـscroll هو اللي بيعطي السرعة.
- **التكلفة عنّا:** ٤ clips و٢ إعادة، ٢٤٠ credit.

### ٤. الفيديو صار صور
- **الصور:** قسّم الـ٢٠ ثانية على ٢٢٠ صورة JPEG.
- **نسختين:** وحدة عريضة (1600x900) للكمبيوتر، ووحدة طويلة (720x1280) للتلفون.

### ٥. الـscroll بيشغّل الفيلم
- **الفكرة:** section طولها ٦ شاشات، وجوّاتها canvas ثابتة (sticky).
- **الحساب:** كل ما تنزل، منحسب وين وصلت (من 0 لـ1)، ومنرسم الصورة اللي بتقابلها.
- **النعومة:** الكود بالإنكليزي تحت.

### ٦. الموقع بعد الفيلم
هون الناس بيغلطوا: فيلم حلو وبعدين موقع عادي. خلّي باقي الموقع من نفس العالم:
- **الخلفية:** نفس اللون.
- **الصور:** صور الزبون الحقيقية، بتضوى وانت عم تنزل.
- **الخط:** خط serif واحد للعناوين.
- **الزر:** زر واحد بس بكل شاشة.

### ٧. شو تقول لـClaude Code
خود الـprompt من القسم الإنكليزي تحت، وحطّ مكان الأقواس تفاصيل البزنس تبعك.

---

## In English

The site opens on a film, and your scroll plays it frame by frame. **Kling** made the film;
**Claude Code** wrote the whole site.

### 1. Story first
Write 4–5 shots that tell the business's story. Ours: the cedar from the shop's label in mist,
then pistachios, then a sheet of pastry, then syrup.
**Rule:** don't let AI generate the product itself (the sweets, the packaging, the shop, people).
AI makes the mood and the ingredients. The real product appears in the client's own photos after
the film, and the page says the film is AI-generated.

### 2. One still per shot
One keyframe image per shot, all in the same light and palette. We made 5.
**End on the site's background colour** (ours is a deep green, `#0a1a11`) so the film dissolves
into the page.

### 3. Video in Kling
- **Mode:** image-to-video, 5 s per clip.
- **Chaining:** give each clip a start frame and an end frame, where the end frame is the next
  clip's start frame. The clips then chain with no cut.
- **Camera:** keep it slow. The scroll provides the speed.
- **Our cost:** 4 clips plus 2 re-shoots, 240 credits.

### 4. Video to frames
- Join the clips (20 s) and sample them evenly into 220 JPEGs.
- When joining, drop the first frame of every clip after the first, since it repeats the shared
  keyframe.
- Make two cuts: wide 1600x900 for desktop and tall 720x1280 for phones.

```bash
ffmpeg -i film.mp4 -vf "fps=11,scale=1600:900:force_original_aspect_ratio=increase,crop=1600:900" -q:v 4 wide/f_%04d.jpg
```

### 5. The scroll plays the film
A section 6 screens tall holds a sticky canvas. On every frame, work out how far through the
section you are (0 to 1) and draw that frame. Easing the playhead toward the target is what
makes it feel like film, not a flipbook.

```html
<section class="film" style="height:600vh">
  <canvas style="position:sticky;top:0;width:100%;height:100vh;display:block"></canvas>
</section>
<script>
  const section = document.querySelector(".film");
  const canvas = section.querySelector("canvas");
  const ctx = canvas.getContext("2d");
  const N = 220;
  const frames = [...Array(N)].map((_, i) => {
    const img = new Image();
    img.src = `/film/wide/f_${String(i + 1).padStart(4, "0")}.jpg`;
    return img;
  });

  function size() {
    const dpr = Math.min(devicePixelRatio || 1, 1.5);
    canvas.width = innerWidth * dpr;
    canvas.height = innerHeight * dpr;
  }

  function draw(i) {
    const img = frames[i];
    if (!img.complete || !img.naturalWidth) return;
    // "cover": fill the canvas, crop the overflow
    const s = Math.max(canvas.width / img.naturalWidth, canvas.height / img.naturalHeight);
    const w = img.naturalWidth * s, h = img.naturalHeight * s;
    ctx.drawImage(img, (canvas.width - w) / 2, (canvas.height - h) / 2, w, h);
  }

  let current = 0;
  function tick() {
    const r = section.getBoundingClientRect();
    const p = Math.min(1, Math.max(0, -r.top / (r.height - innerHeight)));
    current += (p * (N - 1) - current) * 0.14; // ease toward the scroll position
    draw(Math.round(current));
    requestAnimationFrame(tick);
  }

  addEventListener("resize", size);
  size();
  tick();
</script>
```

This minimal version keeps every frame in memory. That's fine on desktop. For phones, the real
site needed three more things:
- the tall cut;
- decoding only the frames near the playhead (`createImageBitmap`, about 10 at a time; mobile
  Chrome crashed holding more);
- a still poster for reduced motion.

Ask Claude for those once the basic version works.

### 6. After the film
This is where most of these sites fail: a beautiful film, then an ordinary website. Keep the rest
of the site in the film's world:
- **Background:** the same colour.
- **Photos:** the client's real photos, lit up as you scroll. CSS `animation-timeline: view()`
  does it with no JavaScript.
- **Type:** one serif for headlines.
- **Actions:** one solid button per screen.

### 7. The prompt for Claude Code

> Build a one-page site for [business] whose top section is a scroll-driven film. The frames are in
> `/film/wide/f_0001.jpg` to `f_0220.jpg` (16:9) and `/film/tall/` (9:16 for phones). Make a
> section 6 viewports tall with a sticky full-screen canvas. Map scroll progress to the frame
> index, ease the playhead toward it (0.14 per frame), and draw with object-fit "cover". Overlay 4
> short text beats that fade in at 0.1, 0.3, 0.5 and 0.75 of the scroll: [your 4 lines]. The film
> ends on [background colour]; the rest of the site uses that same background, with [client's
> photos] lit up as they scroll into view. Respect prefers-reduced-motion with a still poster.

---

Made by Ali Haidar with Claude Code + Kling. Want one built for your business? https://www.iqan.io
