# Cardinal's Nest / PioByte-Hub Codespaces demo

## 1. Prepare GitHub and secrets

1. Commit and push these changes to your GitHub repository (or a fork you own). Select that branch when creating the Codespace; merge to `main` first if you want to use `main`.
2. Provision a dedicated PostgreSQL demo database reachable from Codespaces. Use your provider's connection string, including its required TLS options. The app creates the schema on an empty database, seeds demo accounts, and runs migrations at startup, so use a disposable database or demo copy with the desired Cardinal's Nest settings.
3. On GitHub, click your profile picture → **Settings** → **Codespaces** → **New secret**. Name it `DATABASE_URL`, paste the connection string, grant access to the demo repository under **Repository access**, and click **Add secret**.
4. Repeat for `SESSION_SECRET`, using a long, randomly generated value from your password manager. Set this for the public demo to avoid the app's existing development fallback.

Repository administrators can instead use repository **Settings → Secrets and variables → Codespaces → New repository secret**. Actions secrets do not supply the Codespace environment. Optional integrations use the same mechanism for `TBA_API_KEY`, `TOA_API_KEY`, `NEXUS_API_KEY`, `VAPID_PUBLIC_KEY`, `VAPID_PRIVATE_KEY`, and `VAPID_SUBJECT`; they are not required for basic startup. Never put secret values in Git, terminal commands, or `VITE_*` variables (which can be exposed to browsers).

## 2. Create and start the Codespace

1. Open the repository on GitHub and select the branch containing `.devcontainer/devcontainer.json`.
2. Click **Code → Codespaces → + / Create codespace on [branch]**. If using **New with options**, select the same branch and the repository's dev container configuration, then click **Create codespace**.
3. Wait for container setup to finish. It installs Node.js 22 and runs `npm ci` automatically, including development dependencies. No database connection is needed during installation.
4. In **Terminal → New Terminal**, check secret presence without printing values:

   ```sh
   node -e 'for (const key of ["DATABASE_URL", "SESSION_SECRET"]) { if (!process.env[key]) { console.error(key + " is missing"); process.exitCode = 1; } }'
   ```

5. Start all three development processes (Express, Vite, and the CSS watcher):

   ```sh
   npm run dev
   ```

6. Wait for Vite on port **5000**, `Server running on port 3001`, and completion of startup migrations. Keep this terminal running. Leave `NODE_ENV` unset for this command: production mode makes Express use port 5000 and would conflict with Vite.
7. Open the **Ports** tab beside Terminal (use **View → Command Palette → Ports: Focus on Ports View** if hidden). Open port **5000** in the browser while it is still private. Verify login and a data-backed screen. For a fresh database, the existing seed routine prints initial login information; see `server/storage.ts`'s `seedDatabase()` for account names. Change all seeded account passwords through the app before public sharing.

## 3. Share with FRC Team 6696

1. In **Ports**, right-click **5000 → Port Visibility → Public**. If missing, click **Add port**, enter `5000`, and repeat. Organization policy may disable the Public option.
2. Keep the port protocol **HTTP**: Vite speaks HTTP inside the container; GitHub supplies HTTPS for visitors.
3. Use the copy-address icon on port **5000**. Share that `https://…-5000.app.github.dev` URL with members. Test it in a private/incognito browser window before distributing it.
4. Leave port **3001 Private**. It is forwarded for optional debugging only; members do not need its URL. Public visibility removes GitHub's access gate, so anyone with the URL can reach the app's login screen.
5. Keep the Codespace running during the demo; closing its browser tab does not guarantee it stays running. Check the Codespace idle timeout if needed. When finished, change port 5000 back to **Private**, press **Ctrl+C** in the running terminal, and use **Command Palette → Codespaces: Stop Current Codespace**.

## Routing and troubleshooting

- Existing Vite configuration binds to `0.0.0.0:5000` and accepts forwarded hostnames. Browser requests use relative `/api` URLs with session cookies. Vite forwards `/api` and `/manifest.json` internally to `http://localhost:3001`; the browser stays on the frontend HTTPS origin. No separate API URL or CORS change is needed.
- After adding/changing a Codespaces secret, stop and restart the Codespace. After changing dev container configuration, run **Codespaces: Rebuild Container**. Dependency installation runs again on rebuild; after pulling a changed lockfile, run `npm ci` yourself.
- If Vite chooses port 5001, stop the duplicate process occupying 5000 and restart `npm run dev`. Share port 5000 only once Vite reports it.
- If the frontend loads but API calls fail, check the Express terminal for database/TLS/network errors and wait for migrations. In a second terminal, `curl -i http://localhost:5000/api/me` should reach Express (an unauthenticated response is expected without a session). Check browser Network requests use the same `…-5000.app.github.dev` host, never your Mac's localhost or the private 3001 URL.
- `npm run build` checks the frontend production build. Use `npm run dev` for this two-port demo; `npm run preview` does not run Express or provide the development API proxy.
- Normal Mac development remains unchanged: supply environment variables and run `npm install` / `npm run dev` as before. No container is required locally.

GitHub references: [Codespaces secrets](https://docs.github.com/en/codespaces/managing-your-codespaces/managing-your-account-specific-secrets-for-github-codespaces), [forwarding and sharing ports](https://docs.github.com/en/codespaces/developing-in-a-codespace/forwarding-ports-in-your-codespace?tool=webui).
