# PokéAgent — Pokémon Battle Simulator

At the start of every session, read the `.env` file to understand available environment variables.

## Project context
I'm building a Pokémon battle simulator for my kids.
This project follows an API-first design approach.
The battle business logic does NOT exist yet and will be coded later.

## Tools & MCP
- Always use the Postman MCP tools for any Postman operation
- Never create Postman resources manually or via direct API calls

## Postman workspace
- Workspace name: "PokéAgent"
- Collection name: "PokéAPI Data"
- Mock Server name: "Battle Engine Mock"
- Environment name: "prod"

## Phase 1 — PokéAPI data layer
OpenAPI spec: https://raw.githubusercontent.com/PokeAPI/pokeapi/master/openapi.yml
Base URL: https://pokeapi.co/api/v2
Always read the spec before creating any request.

Endpoints to create:
- GET /pokemon/{name}        → Pokémon stats, types, abilities (example: pikachu)
- GET /type/{name}           → Type damage relations (example: electric)
- GET /move/{name}           → Move power, accuracy, damage class (example: thunderbolt)

Tests on every request:
- Status code is 200
- Response body contains a "name" field

## Phase 2 — Battle API contract (mock only)
POST /battle
Request body:
  { "pokemon1": "pikachu", "pokemon2": "charmander" }

Mock response (fixed, no real logic):
  {
    "winner": "pikachu",
    "loser": "charmander",
    "damage_dealt": 45,
    "move_used": "thunderbolt",
    "type_advantage": true,
    "rounds": 3
  }

## Out of scope
- Battle calculation logic — this will be coded later
- Authentication — PokéAPI is public, no auth needed
- Pagination — not needed for this demo