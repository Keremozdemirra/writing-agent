---
name: "uret"
description: "Görsel ve video üretir — işi en ucuz yetkin modele yönlendirir, sağlayıcı API'sine gider, çıktıyı tek bir düz klasöre kaydeder ve yanına promptunu içeren JSON kaydı bırakır. \"Görsel üret\", \"video oluştur\", \"thumbnail yap\", \"reklam görseli\", \"logo/afiş\", \"ürün fotoğrafı\", \"reels\", \"şunu canlandır\", \"varyasyon çıkar\" gibi her üretim isteğinde kullan. Also triggers on \"generate an image\", \"make me a video\", \"create a thumbnail\", \"produce an ad visual\", \"make variations of this\". Ödemeli video çağrısı öncesi maliyeti söyler ve onay bekler. API anahtarı gerektirir. Elle kurgulanan poster, afiş ve diyagramlar için kullanma; onun için canvas-design kullan."
---

# Generate — images and video

One command: routes the job to the cheapest model that can do it, generates,
saves into a single folder, and never loses the prompt.

> The architecture is adapted from RoboNuggets' free `/generate` build guide
> (skool.com/robonuggets). The model identifiers and prices belong to the date
> that guide was written — **verify them against the provider's own page before
> using them.**

## Four steps
**1. Route** → pick the model for the job and the cheapest provider that runs it,
then read that model's recipe file.
**2. Prepare the references** → load the real images from `refs/`.
**3. Generate** → call the API as the recipe says, wait if it is async, save the
file into the flat folder.
**4. Record** → write a JSON record beside it: which prompt, model, settings.

## Folder
```
uret/
├── models/          one recipe file per model
├── generations/     ALL output lives here, FLAT, NO subfolders
│   └── refs/        logo, face and style references
└── ayar.json        folder path + provider preferences
```
`.env` keys **never go into git.**

## PRECONDITION — on the first run
Claude has **no** built-in image or video generation; the work goes to an
outside provider.

Three aggregators, many models behind one key:

| Provider | Key | Why |
|---|---|---|
| Kie AI (kie.ai) | `KIE_API_KEY` | Usually the cheapest route for the popular models |
| fal.ai | `FAL_KEY` | Fast, wide catalogue, best documentation — first fallback |
| WaveSpeed (wavespeed.ai) | `WAVESPEED_API_KEY` | Second fallback |

**Authentication works three different ways — this is where people get stuck:**
- Google AI Studio → the key goes **in the URL**: `...:generateContent?key={KEY}`
- fal.ai → in the header, **`Authorization: Key {FAL_KEY}`**
- Kie AI → in the header, **`Authorization: Bearer {KIE_KEY}`**

Keep the keys in **environment variables**. **Never ask for a key, never look at
one, never write one to a file** — only check whether it exists with
`[ -n "$FAL_KEY" ]`. If it is missing: say what is needed and where to get it,
then **stop**. Do not fabricate output.

The sandbox may have no access to that API. Test the connection with one cheap
request before the first real call; if there is no access, say so plainly. Never
say "it probably works".

## The recipe file — `models/<model>.md`
Ten minutes per model, once:
```
| Model ID | Provider | Method (Sync/Async) | Type | API key | Docs | Cost |
## Endpoint      POST https://...
## Request       (the exact JSON body from the documentation)
## Response      (where the file is: a base64 field or a URL; for async, the
                  status endpoint and the field that says "finished")
## Notes         (rate limits, size limits, content rules)
```
**The async pattern** (most video models): POST → task id → poll status every
10-15 seconds → on completion a file URL arrives → **download it immediately**
(those URLs die within hours) → save → write the log.

Getting sync and async the wrong way round is the number one reason a first
attempt looks "stuck".

## Choosing models
Start with two: **a cheap image model** and **one video model**. One is sync and
one is async, so between them they teach both patterns.

| Job | What to look for |
|---|---|
| Everyday images, working from a reference | A cheap, fast image model with strong reference support |
| **Legible text inside the image** (signage, poster, menu, UI) | An image model that is good at text — this is a separate capability |
| General video | A reasonably priced video model with good motion |
| High quality video from a first frame | A high quality but slow video model |
| Animating reference images | A model that supports reference-to-video |

Never call a model that has no recipe in `models/`. If you get a "model not
found" error, refresh the identifier from the provider's page and update the
recipe — that is the system's only maintenance task.

## Cost (rough figures from the guide's date — verify them)
| Job | Range |
|---|---|
| Draft image, cheap model | $0.01 – $0.03 · this is the default; experiment freely here |
| Quality image, top model | $0.05 – $0.15 · finals only |
| Video, per second | $0.20 – $0.35 · 10s = $2–3.5 · this is why the cost gate exists |

## RULES — these are where the value actually is

**1. Quote the price before video, and wait for approval.** State the model,
duration, resolution and expected cost, then stop. *Quoting a price is not
approval. One approval means one run.*

**2. Draft cheap, finalise expensive.** Iterate on the cheap model. When the user
picks a favourite, repeat that same prompt on the quality model. Never pay premium
rates for drafts that are going in the bin.

**3. Never describe a logo or a face in words.** A described logo comes back
wrong every time — wrong shape, wrong colour, invented detail. Keep the real file
in `generations/refs/` and pass it to the API as a reference. **If the file does
not exist, stop and ask the user for it.**

**4. One flat folder.** Every output into the same folder, no subfolders.
Foldering feels tidy but breaks every tool that reads the library, and you will
not remember the scheme six months from now.
Naming: `{project}_{description}_{timestamp}.{ext}`

**5. Never hide a provider switch.** Say which route worked and why it fell
through to that one.

**6. One generation at a time.** Otherwise you hit rate limits.

**7. A sidecar log after every save.** Same name plus `.json`:
```json
{ "model": "...", "prompt": "the exact text that went to the API",
  "refs": ["refs/logo.png"], "params": {"aspect":"16:9","size":"2K"},
  "created": "2026-08-11T09:41:00Z" }
```
Three weeks later this is the only answer to "which prompt produced this".

## Writing the prompt — this is where quality is decided
Weak: *"a coffee advert image"*. A good prompt carries five things:
**subject** (what, in what state, from what angle) · **setting** · **light**
(hard or soft, direction, time of day) · **camera** (lens feel, depth, framing) ·
**style** (film, render or illustration; palette; texture).
Write the **negatives** too: watermark, malformed hands, unwanted text,
oversaturation. For video, add: camera movement (locked off / slow pan /
dolly-in), duration, and the movement in the scene.

Aspect ratios: 1:1 feed · 9:16 reels and stories · 16:9 YouTube and web ·
4:5 vertical feed.

For campaign consistency, keep a style card in `uret/stiller/<name>.md` (palette,
light, lens, texture, negatives) and append it to every prompt.

## Gallery
If the user asks, generate a single-file bento or masonry wall that reads the
`generations/` folder: newest first, videos playing muted on hover, click to
enlarge, no filters or tabs. Because the folder is flat, the gallery never needs
updating — new work appears on its own.

## Limits
- Never spend money without approval.
- If you cannot generate something, say so; never create a placeholder file.
- Do not generate the likeness of real people, and do not imitate brand logos.
- Do not try to reproduce the style of a copyrighted work one to one.
- For anything going public, remind the user to check whether an AI disclosure
  label is required.
