# VIEK Client Management

A React/Vite frontend and Express REST API for the Full-Stack Software Development Intern debugging assessment.

## Running locally

From the project root, start the backend:

```sh
cd server
npm install
npm start
```

In a separate terminal, start the frontend:

```sh
cd client
npm install
npm run dev
```

Backend: http://localhost:4000. Frontend: http://localhost:5173 (or the URL printed by Vite).

Demo login: `admin@viek.test` / `password123`.

Client data is stored in memory. Restarting the backend restores the seed data and resets the ID counter.

## Bugs, root causes, and solutions

| Issue | Root cause | Implemented fix |
| --- | --- | --- |
| `React is not defined` prevented rendering | The active JSX transform required React in the component module's scope | Added the default React import in `App.jsx` |
| `projects.map` crashed on initial rendering | Projects state initially contained `undefined` | Initialized projects with `[]` |
| `clients.length` crashed after loading clients | Frontend read `result.clients`, but the API returns `data` | Changed to `setClients(result.data)` |
| Invalid stored token caused a blank screen | Failed list responses were treated as successful data; their missing arrays overwrote state | Checked `response.ok` in both list loaders; on 401, cleared the stored token and token state, displayed the error, and returned to login |
| Adding a client returned 400 | JSON request body lacked the JSON content-type header, so Express did not parse the required fields | Added `Content-Type: application/json` |
| Deleting an existing client returned 404 | Route ID was a string, but stored IDs were numbers | Converted `req.params.id` using `Number()` |
| Filtering projects returned no matches | Query ID was a string, but project client IDs were numbers | Compared against `Number(clientId)` |
| Duplicate IDs produced React key warnings | `clients.length + 1` reused IDs after deletions | Initialized a next-ID counter once and incremented it for each addition |

## Testing

Manual browser testing covered login, client listing, adding and deleting clients, viewing projects, and filtering projects. The full core workflow was retested after the fixes and reported working.

Specific regression tests included:

- Changing the stored token after login: the application returned to the login screen after the rejected request instead of crashing.
- Adding a client after supplying the JSON header: creation succeeded, including a repeat test.
- Deleting a client after converting the route ID: deletion succeeded.
- Filtering for Acme Limited, Bright Solutions, and All Clients: filtering worked after query-ID conversion.
- Adding client 3, deleting clients 1 and 2, and adding more clients: originally reproduced duplicate key `3`; the counter fix passed the add/delete retest.

Additional local checks passed:

- Frontend production compilation using the installed Vite CLI.
- Backend syntax validation using `node --check server.js`.

These are manual regression tests and build/syntax checks, not an automated test suite. Dedicated network-outage, malformed-input, and repeated-delete edge-case testing was not recorded.

## Security and remaining limitations

The backend still rejects unauthorized requests, and the frontend now handles 401 responses during list loading. The following demo limitations remain and should be addressed before production use:

- Credentials are hardcoded and passwords stored in plaintext. Login also returns the entire user object, including its password. Use password hashing and return only necessary public user fields.
- Authentication uses a fixed, non-expiring `demo-token`. Replace it with securely managed sessions or validated, expiring tokens. The token is currently stored in localStorage, where injected scripts could access it.
- CORS allows all origins; production should restrict access to intended frontend origins. Login rate limiting is absent.
- Backend input validation only checks for missing name/email values. Type, format, whitespace, and ID validation remain limited.
- Fetch failures and invalid JSON responses lack comprehensive exception handling. Add/delete requests do not clear rejected sessions in the same way as the list loaders. The server exposes raw error messages.
- Data is not persistent. Deleting a client leaves its projects in memory; a deletion policy for related projects is still needed.

## Reflection

The debugging approach was to reproduce each failure, read the browser console, trace frontend state and API responses, apply a targeted fix, and retest before moving on.

The stored-token problem involved the most stages of investigation: a 401 response led to undefined list state and a rendering crash. Clearing localStorage restored login, but automatic recovery required checking failed responses and clearing invalid session state.

The core workflow passed the final manual retest. The remaining limitations are listed above rather than being presented as completed fixes.
