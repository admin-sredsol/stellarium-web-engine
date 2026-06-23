# Stellarium Web Engine

## Build (Engine — C → WASM)

Requires Emscripten SDK:
```
source $PATH_TO_EMSDK/emsdk_env.sh
make js               # release build (-j8, mode=release)
make js-debug          # debug mode (includes COMPILE_TESTS, SAFE_HEAP, ASSERTIONS)
make js-prof           # profile mode
make js-es6            # ES6 module output
```

- Output: `build/stellarium-web-engine.js` + `.wasm`
- SConstruct runs `tools/make-assets.py` automatically before each compile
- `-include src/private/Makefile` is optional — do not create the file

## Build (Web Frontend — Vue.js)

```
cd apps/web-frontend
make setup    # builds Docker images + engine + yarn install (first time only)
make dev      # runs dev server at http://localhost:8080
make build    # production build
make lint     # eslint via Docker
```

- Frontend expects engine at `apps/web-frontend/src/assets/js/stellarium-web-engine.{js,wasm}`
- To update engine only: `make update-engine` (from web-frontend dir)
- Uses Node 12 + Vue CLI 4 + Vue 2 + Vuetify 2 — do not upgrade
- **Gotcha**: `package.json` has `"vue": "^3.0.0"` which pulls in Vue 3 and breaks the build (vue-template-compiler is v2). Pin to `"vue": "2.6.12"` before `yarn install`.

## Architecture

- **Engine**: C compiled to WASM via Emscripten. Object system: `obj_t` base class + `obj_klass_t` vtable + `MODULE_REGISTER` macro (`src/modules/`).
- **Frontend**: Vue 2 + Vuex + Vue Router + Vuetify. Plugins auto-load from `src/plugins/`.
- **Build**: SCons for C engine. No package.json at root.
- **Deps**: `ext_src/` — vendored C libs (erfa, nanovg, webp, zlib, json, uthash, stb).
- **Data**: `data/` directory compiled into binary by `tools/make-assets.py` as `src/assets/*.inl`.

## Testing

- C tests compiled only in debug mode (`COMPILE_TESTS`). Register with `TEST_REGISTER()` macro.
- Run via `tests_run(filter)` — exposed to JS as `EMSCRIPTEN_KEEPALIVE`.
- `apps/test-skydata/` holds test fixtures (stars, DSOs, MPC orbits, TLE).

## Conventions

- C: GNU11, C++11. `-Werror` on by default (disable with `werror=0`).
- `src/config.h` force-included before all C files.
- Frontend i18n: `tools/update-i18n-en.py` to sync, locales in `src/locales/`.
- ESLint config is in `apps/web-frontend/package.json`.
- No CI/CD pipeline exists.
