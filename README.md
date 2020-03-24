# Electron Auth0 Login

Enables Auth0 login for your Electron.js app.

Usage is simple: you call a `getToken` method, and one of three things happens (in order of preference):

1. If you have a valid token in memory, and won't expire in the next 60 seconds, we return it
2. If you have a refresh token, we exchange it for a new token
3. If you have no refresh token (or have refresh tokens disabled), we open a new window with the Auth0 login page and begin a PKCE flow.

Refresh tokens are stored securely on the user's machine using [node-keytar](https://github.com/atom/node-keytar).

Supports TypeScript out of the box.

## Simple setup (without refresh tokens)

### Auth0 setup

Make sure you have an Auth0 application set up for your Electron app (as a 'native' type, not 'machine-to-machine') and have whitelisted the following redirect URL:

`https://{your-auth0-domain}/mobile`

Get your "Audience URL" from the interface here:

### Dependencies

```
# Installing electron-auth0-login
npm install electron-auth0-login --save

# Installing peer dependencies
npm install request request-promise-native --save
```

### Initialization

**Note: you should add the initialization code to your main process.**

Create a module called `auth.ts`/`auth.js`: 

```typescript
import ElectronAuth0Login from 'electron-auth0-login';

const auth = new ElectronAuth0Login({
    // Get these from your Auth0 application console
    auth0Audience: 'https://api.mydomain.com',
    auth0ClientId: 'abc123ghiMyApp',
    auth0Domain: 'my-domain.eu.auth0.com',
    auth0Scopes: 'given_name profile'
});
```

## Advanced setup - with refresh tokens

To store refresh tokens securely, we use the [node-keytar](https://github.com/atom/node-keytar) package as an optional peerDependency. This uses native code to call Credential Store on Windows, Keychain on Mac, or libsecret on Linux. As such it must be compiled against your Electron v8 version using `electron-rebuild`.

### Dependencies & compile

```
npm install node-keytar --save
npm install electron-rebuild --save-dev
```

Then run Electron-Rebuild:

```
./node_modules/.bin/electron-rebuild
```

Call this again every time you upgrade Electron.

### Initialization

The application config then requires a few tweaks. Again, this code **must be in your main process, not app process**:

```typescript
import ElectronAuth0Login from 'electron-auth0-login';

export new ElectronAuth0Login({
    auth0Audience: 'https://api.mydomain.com',
    auth0ClientId: 'abc123ghiMyApp',
    auth0Domain: 'my-domain.eu.auth0.com',
    auth0Scopes: 'given_name profile offline_access', // add 'offline_access'
    applicationName: 'my-cool-app', // add an application name
    useRefreshTokens: true // add useRefreshTokens: true
});
```

## Usage

You can call `getToken` any time you need an auth0 token, in either the main or app process:

In main process code:

```typescript
import auth from './auth'; // module defined above

async function doSomethingWithAPI() {
    const token = await auth.getToken();
    api.get('/things', {
        headers: {
            'Authorization': 'Bearer ' + token
        }
    });
}
```

In renderer / app process code:

```typescript
import {remote} from 'electron';
import {api} from 'somewhere'

const auth = remote.require('./auth'); // depending where you put 'auth.js'

async function doSomethingWithAPI() {
    const token = await auth.getToken();
    api.get('/things', {
        headers: {
            'Authorization': 'Bearer ' + token
        }
    });
}
```

## Configuring the login window

You can also pass options to the electron `BrowserWindow` by adding a `windowOptions` object to your config, e.g.

```typescript
const auth = new ElectronAuth0Login({
    auth0Audience: 'https://api.mydomain.com',
    auth0ClientId: 'abc123ghiMyApp',
    auth0Domain: 'my-domain.eu.auth0.com',
    auth0Scopes: 'given_name profile',
    windowOptions: {
        width: 1024,
        height: 640,
    }
});
```

These options will be merged into the default options, which are

```
{
    width: 800,
    height: 600,
    alwaysOnTop: true,
    title: 'Log in',
    backgroundColor: '#202020'
};
```

## Credits

This package is based loosely on @adeperio's Electron PKCE example: https://gist.github.com/adeperio/73ce6680d4b80b45e624ab62bacfbdca
