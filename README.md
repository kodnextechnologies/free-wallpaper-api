# Free Wallpaper API

**NexWall** is a free wallpaper REST API built specifically for wallpaper apps. It serves curated **4K portrait wallpapers** (up to 2160×3840) in **50+ categories**, plus looping **live MP4 wallpapers**, as simple JSON.

- **Free plan:** 100 requests/day, 60 requests/minute, 1 API key, no credit card
- **Base URL:** `https://nexwall.kodnextech.com/api/developer/v1`
- **Get a free API key:** https://nexwall.kodnextech.com/developers/register
- **Docs:** https://nexwall.kodnextech.com/wallpaper-api/docs · **OpenAPI 3.0:** https://nexwall.kodnextech.com/openapi.json · **Try it in the browser:** https://nexwall.kodnextech.com/wallpaper-api/sandbox

> This repository is the public reference for the API: endpoints, copy-paste examples in 7 languages, and links to complete open-source starter apps.

---

## Why a wallpaper-specific API?

General photo APIs are great for blogs, but they are a poor fit for wallpaper apps:

- The [Unsplash API Guidelines](https://help.unsplash.com/en/articles/2511245-unsplash-api-guidelines) say you cannot replicate the core Unsplash experience and list *"unofficial clients, wallpaper applications, etc."* as examples.
- Stock libraries mix landscape and portrait photos and are organised by generic keywords, not wallpaper categories such as AMOLED, Anime, Neon or Minimalist.
- None of them provide looping live wallpapers.

NexWall is built for exactly this use case: portrait 9:16 wallpapers, wallpaper categories, and a [Developer API License](https://nexwall.kodnextech.com/wallpaper-api/license) that covers wallpaper apps (commercial / monetized apps need the Pro or Ultra plan).

## Quick start

Try it without a key (up to 10 free wallpapers, limited per IP):

```bash
curl "https://nexwall.kodnextech.com/api/developer/v1/demo/wallpapers?sort=random&per_page=5"
```

With your free API key:

```bash
curl "https://nexwall.kodnextech.com/api/developer/v1/wallpapers?sort=popular&per_page=20" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Accept: application/json"
```

## Official clients

```bash
npm install nexwall      # JavaScript / TypeScript (Node 18+, Bun, Deno, Workers)
pip install nexwall      # Python 3.9+ (client + CLI)
```

```js
import { NexWall } from "nexwall";
const nexwall = new NexWall({ apiKey: process.env.NEXWALL_API_KEY });
const { data } = await nexwall.wallpapers({ sort: "popular", perPage: 20 });
```

Package: [npmjs.com/package/nexwall](https://www.npmjs.com/package/nexwall) · Source: [nexwall-js](https://github.com/kodnextechnologies/nexwall-js) · Python: [pypi.org/project/nexwall](https://pypi.org/project/nexwall/) · [nexwall-python](https://github.com/kodnextechnologies/nexwall-python)
## Endpoints

| Method | Endpoint | What it returns |
|---|---|---|
| `GET` | `/demo/wallpapers` | Up to 10 free wallpapers, **no key needed** (limited per IP) |
| `GET` | `/categories` | Categories available on your plan |
| `GET` | `/wallpapers` | Paginated wallpaper feed (filters below) |
| `GET` | `/categories/{categoryId}/wallpapers` | Wallpapers in one category |
| `GET` | `/wallpapers/{id}` | A single wallpaper |

**`/wallpapers` query parameters**

| Parameter | Values | Notes |
|---|---|---|
| `page` | integer ≥ 1 | default `1` |
| `per_page` | 1–100 | default `50` |
| `category_id` | integer | filter by category |
| `type` | `image` \| `live` | `live` requires the Ultra plan |
| `search` | 2–100 characters | matched against tags |
| `sort` | `newest` \| `oldest` \| `popular` \| `random` | default `newest` |

**Example response (trimmed)**

```json
{
  "data": [
    {
      "id": 1042,
      "category_id": 1,
      "image_url": "https://nexwall.kodnextech.com/storage/wallpapers/example.webp",
      "thumbnail_url": "https://nexwall.kodnextech.com/storage/thumbnails/example.webp",
      "type": "image",
      "tags": "amoled, dark, minimal",
      "resolution": "2160x3840",
      "category": { "id": 1, "name": "AMOLED & Pure Black", "slug": "dark-moody" }
    }
  ],
  "current_page": 1,
  "last_page": 1,
  "per_page": 50,
  "total": 1,
  "plan": "free",
  "remaining_requests_today": 99
}
```

Every response also includes `X-RateLimit-Limit`, `X-RateLimit-Remaining` and `X-RateLimit-Reset` headers. Errors: `401` (bad key), `404` (not found / not on your plan), `422` (invalid parameters), `429` (quota exceeded, see `Retry-After`).

## Examples

<details open>
<summary><b>JavaScript (Node 18+ / server-side)</b></summary>

```js
const res = await fetch(
  "https://nexwall.kodnextech.com/api/developer/v1/wallpapers?sort=random&per_page=10",
  { headers: { Authorization: `Bearer ${process.env.NEXWALL_API_KEY}`, Accept: "application/json" } }
);
const { data } = await res.json();
console.log(data.map((w) => w.image_url));
```
</details>

<details>
<summary><b>Python</b></summary>

```python
import os, requests

r = requests.get(
    "https://nexwall.kodnextech.com/api/developer/v1/wallpapers",
    headers={"Authorization": f"Bearer {os.environ['NEXWALL_API_KEY']}", "Accept": "application/json"},
    params={"sort": "popular", "per_page": 20},
    timeout=15,
)
r.raise_for_status()
for w in r.json()["data"]:
    print(w["image_url"])
```
</details>

<details>
<summary><b>Flutter / Dart</b></summary>

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

Future<List<dynamic>> fetchWallpapers(String apiKey) async {
  final res = await http.get(
    Uri.parse('https://nexwall.kodnextech.com/api/developer/v1/wallpapers?per_page=30'),
    headers: {'Authorization': 'Bearer $apiKey', 'Accept': 'application/json'},
  );
  if (res.statusCode != 200) throw Exception('NexWall error ${res.statusCode}');
  return jsonDecode(res.body)['data'];
}
```
</details>

<details>
<summary><b>Android (Kotlin + Retrofit)</b></summary>

```kotlin
interface NexWallApi {
    @GET("api/developer/v1/wallpapers")
    suspend fun wallpapers(
        @Header("Authorization") auth: String,   // "Bearer YOUR_API_KEY"
        @Query("per_page") perPage: Int = 30,
        @Query("sort") sort: String = "newest",
    ): WallpaperPage
}
```
</details>

<details>
<summary><b>PHP / Laravel</b></summary>

```php
use Illuminate\Support\Facades\Http;

$wallpapers = Http::withToken(config('services.nexwall.key'))
    ->acceptJson()
    ->get('https://nexwall.kodnextech.com/api/developer/v1/wallpapers', ['per_page' => 20])
    ->throw()
    ->json('data');
```
</details>

<details>
<summary><b>Swift (iOS)</b></summary>

```swift
var req = URLRequest(url: URL(string: "https://nexwall.kodnextech.com/api/developer/v1/wallpapers?per_page=20")!)
req.setValue("Bearer \(apiKey)", forHTTPHeaderField: "Authorization")
req.setValue("application/json", forHTTPHeaderField: "Accept")
let (data, _) = try await URLSession.shared.data(for: req)
```
</details>

<details>
<summary><b>Postman / Insomnia</b></summary>

Import the OpenAPI spec directly: `https://nexwall.kodnextech.com/openapi.json`
</details>

> **Keep your key secret.** Don't ship it inside an APK, IPA or public JavaScript. Call the API from your own backend and cache responses; this also stretches the free quota across many users.

## Complete starter apps (MIT)

| Platform | Repository |
|---|---|
| Flutter | [nexwall-flutter-wallpaper-app](https://github.com/kodnextechnologies/nexwall-flutter-wallpaper-app) |
| Android (Kotlin, Jetpack Compose) | [nexwall-android-kotlin-wallpaper-app](https://github.com/kodnextechnologies/nexwall-android-kotlin-wallpaper-app) |
| React Native / Expo | [nexwall-react-native-expo-wallpaper-app](https://github.com/kodnextechnologies/nexwall-react-native-expo-wallpaper-app) |
| Next.js / React, Laravel, plain JavaScript | [nexwall-web-starter](https://github.com/kodnextechnologies/nexwall-web-starter) |
| Python client + daily desktop wallpaper changer | [nexwall-python](https://github.com/kodnextechnologies/nexwall-python) |

## Plans

| Plan | Price | Requests/day | Includes |
|---|---|---|---|
| **Free** | $0 / ₹0 | 100 | Non-premium categories, 4K static wallpapers |
| **Pro** | $4.99/mo · ₹399/mo | 10,000 | All 50+ categories, commercial use in monetized apps |
| **Ultra** | $10.99/mo · ₹899/mo | 50,000 | Everything in Pro + live MP4 wallpapers |

Details: https://nexwall.kodnextech.com/wallpaper-api/pricing

## FAQ

**Is there a free wallpaper API?**
Yes. NexWall's free plan gives 100 requests per day with an API key and no credit card.

**Can I use it in a Play Store app?**
Yes. Free is for building and testing; monetized apps (AdMob, subscriptions) need Pro or Ultra under the [Developer API License](https://nexwall.kodnextech.com/wallpaper-api/license).

**Does it have live wallpapers?**
Yes, looping MP4 live wallpapers via `type=live` on the Ultra plan.

**Is there an API without a key?**
Yes, for trying it out: `GET /demo/wallpapers` returns up to 10 free wallpapers with no key (10 requests/minute, 30/day per IP). For pagination and all other endpoints, use a free key. The [sandbox](https://nexwall.kodnextech.com/wallpaper-api/sandbox) also lets you try requests in the browser.

## License

The examples in this repository are MIT licensed. Wallpaper images and videos returned by the API are governed by the [NexWall Developer API License](https://nexwall.kodnextech.com/wallpaper-api/license), not by MIT.

Built by [KodNex Technologies](https://kodnextech.com). Disclosure: this is our API.
