# PokéDex — Personal Pokémon Collection App

## Project context
I'm building a personal Pokédex web app for my kids. They each have an account
and they collect Pokémon over time — the app remembers which Pokémon they've
caught, and lets them browse the details of each one.

This project follows an **API-first design** approach:
1. Consume an existing public API (PokéAPI) for raw Pokémon data
2. Define our own API contract for user-specific data (the Pokédex itself)
3. Mock that contract in Postman before writing any backend code
4. Generate the OpenAPI spec from the mock
5. Build a frontend that consumes BOTH APIs:
   - Our mock for user-specific data (profiles, caught Pokémon)
   - PokéAPI directly for rich reference data (full stats, descriptions)

The backend business logic does NOT exist yet — it will be coded later.
User-specific data in this demo runs against the Postman Mock Server.

## Tools & MCP rules
- Always use the Postman MCP tools for any Postman operation
- Never create Postman resources via direct API calls or manual UI clicks
- Always read OpenAPI specs before creating requests against an API
- The Pokédex business logic (catching, persistence, auth) is out of scope —
  use mock responses only

## Postman workspace structure
- Workspace name: `PokéDex`
- Collections:
  - `PokéAPI Data` — consumes the public PokéAPI (read-only reference data)
  - `PokéDex API` — our own API contract for user-specific data (mocked)
- Mock Server name: `PokéDex Mock`
- Environment name: `local`

---

## Phase 1 — PokéAPI data layer (public reference data)

OpenAPI spec: https://raw.githubusercontent.com/PokeAPI/pokeapi/master/openapi.yml
Base URL: https://pokeapi.co/api/v2

Always read the spec before creating any request.

Endpoints to create in the `PokéAPI Data` collection:
- `GET /pokemon/{name}`   → stats, types, abilities, sprites (example: pikachu)
- `GET /type/{name}`      → type damage relations (example: electric)
- `GET /move/{name}`      → move power, accuracy, damage class (example: thunderbolt)

Tests on every request:
- Status code is 200
- Response body contains a `name` field

---

## Phase 2 — PokéDex API contract (our own API, mocked)

This is the API my frontend will consume. The backend will be implemented later.

### Resource: User
A registered child using the app.
```json
{
  "id": "user_001",
  "name": "Lucas",
  "avatar": "pikachu",
  "created_at": "2025-01-15T10:00:00Z"
}
```

### Resource: CaughtPokemon
A Pokémon caught by a specific user.
```json
{
  "id": "catch_42",
  "user_id": "user_001",
  "pokemon_name": "pikachu",
  "pokemon_id": 25,
  "caught_at": "2025-03-10T14:30:00Z",
  "nickname": "Sparky",
  "level": 12
}
```

### Endpoints to mock

**`GET /users`** — List all users
Response 200:
```json
[
  { "id": "user_001", "name": "Lucas",  "avatar": "pikachu",    "created_at": "2025-01-15T10:00:00Z" },
  { "id": "user_002", "name": "Jules",  "avatar": "charmander", "created_at": "2025-01-18T11:00:00Z" },
  { "id": "user_003", "name": "Mathys", "avatar": "bulbasaur",  "created_at": "2025-01-20T09:00:00Z" },
  { "id": "user_004", "name": "Alba",   "avatar": "eevee",      "created_at": "2025-01-22T14:00:00Z" }
]
```

**`GET /users/{user_id}/pokedex`** — List Pokémon caught by a user
The mock should return a different Pokédex for each user. Some overlap between
users is fine, but each kid should have their own distinct collection.

Suggested mock data per user:

- **Lucas (user_001)** — electric & fast types
  - pikachu (id 25, lvl 12, "Sparky"), jolteon (id 135, lvl 18, "Volt"),
    raichu (id 26, lvl 22, "Thunder"), zapdos (id 145, lvl 30, "Storm")

- **Jules (user_002)** — fire types
  - charmander (id 4, lvl 8, "Blaze"), charizard (id 6, lvl 25, "Inferno"),
    arcanine (id 59, lvl 20, "Ash"), magmar (id 126, lvl 16, "Ember")

- **Mathys (user_003)** — grass & nature types
  - bulbasaur (id 1, lvl 10, "Leafy"), venusaur (id 3, lvl 28, "Bloom"),
    snorlax (id 143, lvl 24, "Big Guy"), pikachu (id 25, lvl 9, "Buddy")

- **Alba (user_004)** — cute & psychic types
  - eevee (id 133, lvl 14, "Fluff"), mew (id 151, lvl 20, "Sparkle"),
    jigglypuff (id 39, lvl 11, "Lullaby"), espeon (id 196, lvl 19, "Stardust")

Response 200 example for Lucas:
```json
{
  "user_id": "user_001",
  "total_caught": 4,
  "pokemons": [
    { "id": "catch_101", "user_id": "user_001", "pokemon_name": "pikachu",  "pokemon_id": 25,  "caught_at": "2025-03-10T14:30:00Z", "nickname": "Sparky",  "level": 12 },
    { "id": "catch_102", "user_id": "user_001", "pokemon_name": "jolteon",  "pokemon_id": 135, "caught_at": "2025-03-12T16:00:00Z", "nickname": "Volt",    "level": 18 },
    { "id": "catch_103", "user_id": "user_001", "pokemon_name": "raichu",   "pokemon_id": 26,  "caught_at": "2025-03-14T11:00:00Z", "nickname": "Thunder", "level": 22 },
    { "id": "catch_104", "user_id": "user_001", "pokemon_name": "zapdos",   "pokemon_id": 145, "caught_at": "2025-03-18T09:30:00Z", "nickname": "Storm",   "level": 30 }
  ]
}
```

**`POST /users/{user_id}/pokedex`** — Catch a new Pokémon
Request body:
```json
{ "pokemon_name": "squirtle", "nickname": "Bubbles", "level": 5 }
```
Response 201:
```json
{
  "id": "catch_45",
  "user_id": "user_001",
  "pokemon_name": "squirtle",
  "pokemon_id": 7,
  "caught_at": "2025-03-20T10:00:00Z",
  "nickname": "Bubbles",
  "level": 5
}
```

**`DELETE /users/{user_id}/pokedex/{catch_id}`** — Release a Pokémon
Response 204 (no body)

---

## Phase 3 — OpenAPI spec generation

Once the mock is set up, ask Postman to **generate an OpenAPI 3.0 specification**
from the `PokéDex API` collection. Save it as `pokedex-api.yaml` at the project root.

This spec will be the source of truth for the frontend client.

---

## Phase 4 — Frontend client

A single-page web app (HTML + Tailwind + vanilla JS or React) that consumes
**two APIs**:
- **PokéDex Mock** (Postman) — for user-specific data (profiles, caught list)
- **PokéAPI** (public) — for rich Pokémon reference data (description, sprites,
  evolution, full stats) when the user wants to dig deeper

### Screens & behavior

**Home screen — Users**
- Calls `GET {mock}/users` — lists all kid profiles
- Click on a user → navigates to their Pokédex screen

**Pokédex screen — User's collection**
- Calls `GET {mock}/users/{user_id}/pokedex` — gets the user's caught Pokémon
- Each Pokémon is shown as a card with: sprite, name, nickname, level, caught date
- Sprites are rendered from PokéAPI's CDN, built client-side from `pokemon_id`:
  `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/{id}.png`
- Each card has a **"Details"** button that opens the Pokémon detail modal
- A **"Catch a new Pokémon"** button opens the catch form
- A **"Release"** button on each card calls DELETE on the mock (with confirmation)

**Catch form — adding a new Pokémon**
- The user must NOT type the Pokémon name freely — they pick from a list
- Provide a **searchable autocomplete input** powered by PokéAPI:
  - On focus or first keystroke, fetch `GET https://pokeapi.co/api/v2/pokemon?limit=151`
    (the original 151 Pokémon — keep it manageable for kids)
  - Cache the result client-side so we don't refetch on every keystroke
  - Filter the list as the user types (substring match, case-insensitive)
  - Show a dropdown with up to 8 suggestions, each with the Pokémon sprite + name
  - On selection, the form auto-fills the name and shows a preview sprite
- Additional fields the user fills in: nickname (optional, free text), level (1-100, default 5)
- On submit: POST to `{mock}/users/{user_id}/pokedex` with the chosen Pokémon's name and metadata

**Pokémon detail modal — when "Details" is clicked**
- Calls `GET https://pokeapi.co/api/v2/pokemon/{name}` — full Pokémon data
- Calls `GET https://pokeapi.co/api/v2/pokemon-species/{name}` — flavor text /
  description in plain English (use the first English entry in `flavor_text_entries`)
- Displays: large sprite, full stats (HP, Attack, Defense, Sp. Atk, Sp. Def, Speed)
  shown as horizontal bars, types with colored badges, abilities, height, weight,
  and the description text
- Loading state while PokéAPI responds
- Error state if PokéAPI is unreachable (with a "Retry" button)

### Design requirements
- Clean, modern, kid-friendly UI — Pokémon-themed but not childish
  (think: minimalist gaming dashboard)
- Responsive (mobile + desktop)
- Loading and error states handled gracefully on both APIs
- Type badges use the canonical Pokémon type colors (electric = yellow,
  fire = orange-red, water = blue, etc.)

### Architectural note
This dual-API setup is intentional and a key teaching point:
- **Our mock** owns user-specific state (who caught what)
- **PokéAPI** owns reference data (what a Pokémon IS)
- The frontend orchestrates both — exactly how a real production app would
  combine an internal API with a third-party data source.

---

## Out of scope
- Authentication (single-user demo for now)
- Real database / backend implementation
- Real catch logic (random encounters, type-based capture rate, etc.)
- Pagination on the Pokédex (small collections only for the demo)