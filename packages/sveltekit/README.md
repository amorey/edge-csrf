# SvelteKit

This is the documentation for Edge-CSRF's SvelteKit integration.

## Quickstart

First, add the integration library as a dependency to your app:

```console
npm install @edge-csrf/sveltekit
# or
pnpm add @edge-csrf/sveltekit
# or
yarn add @edge-csrf/sveltekit
```

Next, create a server-side hooks file (`hooks.server.ts`) for your project and add the Edge-CSRF handle:

```typescript
// src/hooks.server.ts

import { createCsrfHandle } from '@edge-csrf/sveltekit';

// initalize csrf protection handle
const csrfHandle = createCsrfHandle({
  cookie: {
    secure: process.env.NODE_ENV === 'production',
  },
});

export const handle = csrfHandle;
```

Now, all HTTP submission requests (e.g. POST, PUT, DELETE, PATCH) will be rejected if they do not include a valid CSRF token. To add the CSRF token to your forms, you can fetch it from the event's `locals` data object server-side. For example:

```typescript
// src/routes/+page.server.ts

export async function load({ locals }) {
  return {
    csrfToken: locals.csrfToken,
  };
}

export const actions = {
  default: async () => {
    return { success: true };
  },
};
```

```svelte
<!-- src/routes/+page.svelte -->

<script lang="ts">
  export let data;

  export let form;
</script>

{#if form?.success}
<span>success</span>
{:else}
<form method="post">
  <input type="hidden" value={data.csrfToken}>
  <input type="text" name="my-input">
  <input type="submit">
</form>
{/if}
```

Finally, to make typescript aware of the new `locals` attributes you can add Edge-CSRF types to your app's types:

```typescript
// src/app.d.ts

import type { CsrfLocals } from '@edge-csrf/sveltekit';

declare global {
  namespace App {
    // ...
    interface Locals extends CsrfLocals {}
    // ...
  }
}

export {};
```

## Examples

Here are some [examples](/examples) in this repository:

| Adapter    | Implementation                             |
| ---------- | ------------------------------------------ |
| vercel     | [HTML form](/examples/sveltekit-vercel)     |
| cloudflare | [HTML form](/examples/sveltekit-cloudflare) |

## Lower-level implementations

If you want lower-level control over the response or which routes CSRF protection will be applied to you can use the `createCsrfProtect()` method to create a function that you can use inside your own custom handle:

```typescript
// src/hooks.server.ts

import type { Handle } from '@sveltejs/kit';
import { CsrfError, createCsrfProtect } from '@edge-csrf/sveltekit';

// initalize csrf protection method
const csrfProtect = createCsrfProtect({
  cookie: {
    secure: process.env.NODE_ENV === 'production',
  },
});

// SvelteKit handle
export const handle: Handle = async ({ event, resolve }) => {
  try {
    await csrfProtect(event);
  } catch (err) {
    if (err instanceof CsrfError) return new Response('invalid csrf token', { status: 403 });
    throw err;
  }
    
  return resolve(event);
};
```

## Configuration

```javascript
// default config

{
  cookie: {
    name: '_csrfSecret',
    path: '/',
    maxAge: undefined,
    domain: '',
    secure: true,
    httpOnly: true,
    sameSite: 'strict',
    partitioned: undefined
  },
  excludePathPrefixes: [],
  ignoreMethods: ['GET', 'HEAD', 'OPTIONS'],
  saltByteLength: 8,
  secretByteLength: 18,
  token: {
    fieldName: 'csrf_token',
    value: undefined
  }
}
```

## API

The following are named exports in the the `@edge-csrf/sveltekit` module:

### Types

```
SveltekitCsrfProtect - A function that implements CSRF protection for SvelteKit request events

  * @param {RequestEvent} event - The SvelteKit request event instance
  * @returns {Promise<void>} - The function completed successfully
  * @throws {CsrfError} - The function encountered a CSRF error
```

### Classes

```
CsrfError - A class that inherits from Error and represents CSRF errors
```

### Methods

```
createCsrfHandle([, options]) - Create a new SvelteKit handle to be used in hooks.server.ts

  * @param {object} [options] - The configuration options
  * @returns {Handle} - The handle

createCsrfProtect([, options]) - Create a lower-level function that can be used inside a SvelteKit handle
                                 to implement CSRF protection for requests

  * @param {object} [options] - The configuration options
  * @returns {SveltekitCsrfProtect} - The CSRF protection function
```
