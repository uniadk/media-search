# Media search

A single page that searches games, manga, anime, movies, TV shows, and music. Open it from disk. There is no server, no install, and no build step.

## Use it locally

1. Download or clone this repository.
2. Open `index.html` in Chrome or Firefox (double-click the file, or drag it into the window).

The address will start with `file://`. That is expected. Typing a title sends the search from your browser straight to the public catalogs below.

## How to search

Type at least two characters. Results update shortly after you pause. Enter searches immediately. Escape clears the box.

**All** queries every catalog at once. A category tab queries only that catalog.

Each result shows the title, the type, a year or other detail when the catalog has one, a thumbnail when one is available, and a link to the original page.

## Catalogs

No API keys and no accounts. Each endpoint responds with `Access-Control-Allow-Origin: *`, so a page opened from disk can read it.

| Category | Catalog | Endpoint |
| --- | --- | --- |
| Games | [English Wikipedia](https://en.wikipedia.org/) | `https://en.wikipedia.org/w/api.php` with `origin=*` and `hastemplate:"Infobox video game"` |
| Manga | [AniList](https://anilist.co/) | `POST https://graphql.anilist.co` (`type: MANGA`) |
| Anime | [AniList](https://anilist.co/) | `POST https://graphql.anilist.co` (`type: ANIME`) |
| Movies | [English Wikipedia](https://en.wikipedia.org/) | same API, `hastemplate:"Infobox film"` |
| TV | [TVmaze](https://www.tvmaze.com/api) | `https://api.tvmaze.com/search/shows?q=` |
| Music | [Apple iTunes Search](https://performance-partners.apple.com/search-api) | `https://itunes.apple.com/search` (`media=music`, `entity=album`, US store, JSONP `callback`) |

On **All**, anime and manga share one AniList request. The other catalogs are requested in parallel. A failed catalog is named in the status line; the rest still render. AniList titles flagged as adult are omitted.

### Why these, and not the usual ones

Checked from a browser-like client on 3 October 2026:

- **Jikan** (`api.jikan.moe`) publishes only an IPv6 address and did not connect from this network. [AniList](https://docs.anilist.co/) answered a cross-origin GraphQL search for anime and manga, including covers and links, with no key. Its rate-limit header reported about 30 requests per minute, so the page waits until you pause typing.
- **CheapShark** is key-free and documents CORS, but it rejects normal browser User-Agent strings. A page cannot replace that header, so the games search would fail in every browser. **Speedrun.com** allows CORS, but a name search for Zelda ranked fan games above the main series. English Wikipedia’s video-game infobox search returned Portal, Portal 2, and Hades in a sensible order, with thumbnails and article links.
- **iTunes movie search** returned an empty catalog for Inception, Spirited Away, The Matrix, Toy Story, and even the word “love”, while album search still returned records. Movies therefore use Wikipedia. TV uses TVmaze because it returns shows (with a year, network, and poster) rather than store season listings; iTunes had no hit for Severance. Album results are loaded with JSONP because the search response omits CORS headers for a `file://` page.
- **RAWG, IGDB, and TMDB** need API keys, so they are not used.

### Attribution

- Game and movie rows link to English Wikipedia. Thumbnails are loaded from Wikimedia for identification and are not stored in this repository. Wikipedia text is available under [CC BY-SA](https://creativecommons.org/licenses/by-sa/4.0/). This page shows the title and a year, then sends you to the article.
- Anime and manga data and cover images come from [AniList](https://anilist.co/) and MyAnimeList’s community data behind it. Links go to AniList.
- TV data and posters come from [TVmaze](https://www.tvmaze.com/). Please keep the TVmaze link when you reuse this page.
- Music rows use album artwork and links from Apple’s iTunes Search API, only to identify the Apple Music listing next to that link. Apple is not involved with this project. Apple’s [search API terms](https://performance-partners.apple.com/search-api) apply to that artwork.

## Browser quirks

Checked by opening `index.html` from `file://` in headless Chrome (no special flags):

- The page origin is `null`. Wikipedia, AniList, and TVmaze answer `fetch` with `Access-Control-Allow-Origin: *`. Requests do not send credentials.
- Wikipedia only adds that header when the query includes `origin=*`. The page always sends it.
- AniList is a JSON `POST`, so the browser sends a preflight. AniList answers `OPTIONS` from `Origin: null` and allows `Content-Type`. That preflight succeeded from `file://`.
- iTunes varies CORS by the `Origin` header and does **not** send `Access-Control-Allow-Origin` for `null`. A normal `fetch` from this page fails. Music therefore uses Apple’s documented `callback` parameter (JSONP via a script tag), which is not subject to that check. The callback name is fixed in the page; the search text is only a query parameter.
- Safari often blocks network access from local files, including script tags. If every catalog fails and you are in Safari, open the file in Chrome or Firefox.
- A catalog that takes longer than about 15 seconds is skipped so the others can still appear.
- TV results below a loose relevance score are dropped so a search does not fill up with unrelated shows.
- Music is the US Apple Music catalog. Game and movie matches are English Wikipedia, so English titles work best. AniList also matches romaji.
- Nothing is cached by a service worker. Refresh the file after you replace it.

## Privacy

Searches leave your machine only as direct requests to Wikipedia, AniList, TVmaze, and Apple. This page has no analytics and no backend.

## License

[MIT](LICENSE). Catalog data stays under the terms of the services above.
