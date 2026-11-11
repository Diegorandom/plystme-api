# Plyst Me — backend API for a music recommendation platform

The server that ran Plyst Me: Spotify login, user taste mining, playlist generation, and
collaborative listening events over sockets. Started 2019, where I was CTO. Product code, not
an exercise — it shipped, and the recommendation model was part of my responsibility.

This README is written from the code, not from memory. Every claim below was checked against a
file on 2026-11-10.

## What the server does

| Area | What it handles |
|---|---|
| **Auth** | Spotify OAuth: `/login` → `/callback`, with a signed session cookie |
| **Users** | Profile, preferences, per-user track history, user mining (`/mineria`) |
| **Recommendation** | `/pool` and `/callback` feed a queue, `/guardar/top50` persists a ranking |
| **Playlists** | `/create/playlist`, `/save/track`, `/track/profile` |
| **Events** | Collaborative listening rooms over Socket.IO, with join codes |
| **Ops** | `/heartbeat`, `/idle`, `/chequeoBD`, `/esHost`, `/esInvitado` |

## Stack, read from `package.json`

| | |
|---|---|
| Runtime | Node.js (`engines`: **v20.2.0**), CommonJS |
| HTTP | **Express** with `body-parser`, `cookie-parser`, `method-override`, `morgan`, `express-rate-limit` |
| Realtime | **Socket.IO** |
| Database | **Neo4j** over Bolt (`neo4j-driver`), hosted on GraphENEDB |
| Music API | **spotify-web-api-node** |
| Sessions | **client-sessions** — a signed cookie, not a server-side store |
| APM | **New Relic** (`newrelic.js`) |
| Audio | `supercolliderjs` is a dependency, but the SC controller is commented out |

## Running it

```bash
npm install
npm start        # nodemon index.js
```

Two branches at startup, and they behave differently:

- **`PORT == 5001`** — local mode. Callback goes to `http://localhost:5001/callback`.
- **anything else** — server mode. Callback goes to `https://www.plystme.com/callback`.

### Environment variables the code reads

| Variable | Used by |
|---|---|
| `PORT` | Express listen port; also selects local vs server mode |
| `client_id`, `client_secret` | Spotify OAuth |
| `GRAPHENEDB_BOLT_URL`, `GRAPHENEDB_BOLT_USER`, `GRAPHENEDB_BOLT_PASSWORD` | Neo4j connection |

There is no `requirements.txt` and no Python in this repository.

## HTTP surface

`index.js` mounts one controller per feature under `src/controllers/`:

| Route | Controller |
|---|---|
| `/`, `/heartbeat`, `/idle` | `inicio`, `heartbeat`, `idle` |
| `/login`, `/logOut` | `login`, `logOut` |
| `/callback`, `/pool` | `callbackAlgoritmo`, `poolAlgoritmo` |
| `/perfil`, `/preferencias`, `/usuarios`, `/userid` | `perfil`, `preferencias`, `usuarios` |
| `/track/profile`, `/save/track` | `perfilTrack`, `saveTrack` |
| `/create/playlist`, `/guardar/top50` | `guardaPlaylist`, `guardarTOP50` |
| `/mineria`, `/rango`, `/DatosBD`, `/chequeoBD` | `mineriaUsuario`, `rangoTiempo`, `DatosBD`, `chequeoBD` |
| `/esHost`, `/esInvitado`, `/environment` | `esHost`, `environmentSC` |
| `/error`, `*` | error handler and catch-all in `index.js` |

## Socket surface

`src/sockets/mainSocket.js` wires the events in `src/sockets/Eventos/`:

`connection` · `crearEvento` · `crearEventoCodigo` · `EventoConexion` · `usuarioNuevoCodigo` ·
`usuarioNuevoUbicacion` · `disconnect`

## Known state, honestly

This is a 2019 codebase kept for reference. Three things a reader will hit:

1. **The server branch is broken.** `index.js:107` passes `clientSecret: config.sessionSecret`
   in the `PORT != 5001` branch, and `config` was removed from the file when credentials moved to
   environment variables (`8f6c1f5`, June 2023). That branch throws `ReferenceError` on startup.
   Local mode works; production mode needs `process.env.client_secret` there instead.
2. **Every middleware is registered twice** (`index.js:25-30` and `67-72`): `logger`,
   `bodyParser`, static, `cookieParser`, `methodOverride`. Harmless, wasteful, and a sign of
   how the file grew.
3. **Dead references.** `express.static` points at a `public/` directory that is not in the repo,
   and `otrosProcesos` serves a list of `.ejs` templates that are not here either.

`node_modules` was committed for years and is gone from the current history; that is why the
repository is large.

## Security

- **`secret-config.json`** held a session secret and was committed in 2021. On 2026-11-10 every
  branch was rewritten to remove it and the credential was rotated in the Spotify dashboard.
  GitHub keeps `refs/pull/*` immutable, so the old blob can still be reached from some
  pull-request refs; it no longer grants access to anything.
- The **Spotify client id is hardcoded** in `index.js`, which is normal: client ids are public.
  The secret is not.
- The session cookie is named `sessions`, lasts **24 hours**, has an **active duration of 5
  minutes**, and is `ephemeral`.

## License

MIT — see `LICENSE`.
