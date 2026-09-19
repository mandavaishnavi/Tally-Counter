# Sync setup

The app works with no backend at all. These steps add cross-device sync for you
alone; everyone else keeps using it locally.

## 1. Files

Your repo should end up like this:

    index.html
    schema.sql
    functions/
      api/
        state.js

Cloudflare Pages picks up `functions/` automatically. No build step, no
dependencies, no `wrangler.toml` needed.

## 2. Create the D1 database

Dashboard → Storage & Databases → D1 → Create database. Name it `tally`.

Open the database → Console, paste the contents of `schema.sql`, run it.

## 3. Bind it to the Pages project

Your Pages project → Settings → Bindings (older dashboards: Functions →
D1 database bindings).

- Add a **D1 database** binding, variable name `DB`, pointing at `tally`.
- Add an **environment variable**, name `SYNC_KEY`, marked as a **secret**.
  Use a long random value, e.g. from `openssl rand -hex 24` on Ubuntu.
- Add both to **Production**. Add them to Preview too if you want preview
  deployments to sync.

Redeploy after adding bindings — existing deployments don't pick them up.

## 4. Connect your devices

Open the site, tap the circular-arrows icon in the top right, paste the same
`SYNC_KEY` value, and hit Connect. Repeat on your other device.

## How it syncs

The whole app state is one JSON row, stamped with the time of your last local
change. On open, on tab focus, and 1.5 s after any change, the app compares
timestamps and the newer side wins.

That means last-write-wins, not merging. If you count on the phone while the
laptop is offline and also count on the laptop, whichever device syncs last
overwrites the other. For a personal counter that's almost never a problem, but
it's the tradeoff being made.

Without the key the app never calls `/api/state` at all, so visitors are
unaffected and cost you nothing.
