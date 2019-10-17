---
rank: 2
related_endpoints: []
related_guides:
  - embed/ui-elements
required_guides:
  - embed/ui-elements/installation
related_resources: []
alias_paths: []
cId: embed
scId: embed/ui-elements
id: embed/ui-elements/open-with
isIndex: false
---

# Content Open With

The Box Content Open With UI Element allows developers to embed a dropdown to
open content stored in box with a partner application, or locally on the
desktop.

The Element fetches information about enabled integrations using the
Box API, and calls out to partner services. Users can then take action in these
services, and the edited content will be automatically saved back to Box.

The integrations included in the Open With Element are Adobe Sign, Google Suite,
and Box Edit. Additional information on the Google Suite integration can be
found on the [Box Community site][community].

Currently, the element only supports [App Users](g://authentication/user-types)
for authentication.

## Box Edit

Box Edit requires additional setup in order to be integrated into a custom
application. Box Edit uses the desktop application [Box Tools][tools] in order
to open content locally.

- Requests must use a secure connection (from an `https` domain)
- The application's domain must be whitelisted by Box Tools. Instructions can be
  found [here][custom-domains]. The ideal workflow is to bundle these steps
  within an installer that also installs Box Tools.
- Safari requires a browser extension to connect to box tools. More details can
  be found [here][safari].

## G Suite

A valid G Suite account is required in order to use the Box for G Suite
integration. To connect a user's G Suite and Box account, the
`external_app_user_id` of the app user must be updated to be the email address
associated with the user's G Suite account.

The `external_app_user_id` of an app user can be updated via the
[`PUT /users/:id`](e://put-users-id) endpoint.

## Setup

The Open With UI Element is intended to be used after whitelisting your
application and enabling integrations for app users using Box API endpoints.

### Activate application

The 'Open With' UI Element is available to any developer building with the Box
Platform. To activate it for your instance, add the "Enable integrations" scope
to your application in the developer console.

<ImageFrame border>

![Enable Integrations](./images/enable-integrations.png)

</ImageFrame>

Once your application has been activated for API calls it will need to be
reauthorized in your enterprise. The steps for performing these actions are
available [here](g://applications/custom-apps/app-approval).

## List available integrations

The first step to assigning an app integration to a user is to get the list of
integrations that are available. This `GET` request can be made with the following
`cURL` request.

```curl
curl -X GET https://api.box.com/2.0/app_integrations \
  -H 'Authorization: Bearer [ACCESS_TOKEN]'
```

```json
{
  "next_marker": null,
  "entries": [
    { "type": "app_integration", "id": "10897" },
    { "type": "app_integration", "id": "1338" },
    { "type": "app_integration", "id": "13418" },
    { "type": "app_integration", "id": "3282" }
  ],
  "limit": 100
}
```

The app integration IDs are used to assign an integration to a given user.

## Get a specific integration

To obtain additional information about a specific integration, based on ID, the
following GET request may be made.

```curl
curl -X GET \
  https://api.box.com/2.0/app_integrations/[APP_INTEGRATION_ID] \
  -H 'Authorization: Bearer [ACCESS_TOKEN]'
```

```json
{
  "type": "app_integration",
  "id": "3282",
  "app": {
    "type": "app",
    "id": "81713"
  },
  "name": "Sign with Adobe Sign",
  "description": "Send your document for signature to Adobe Sign",
  "executable_item_types": ["FILE"],
  "restricted_extensions": ["pdf", "doc", "docx", "xls", "xlsx", "ppt", "pptx"],
  "scoped_to": "root"
}
```

## Add integration to user

To add an app integration to a valid app user, three pieces of information are
required:

- A valid [Service Account](g://authentication/user-types) Access Token.
- The ID of the app user to be assigned the integration
- The ID of the app integration to assign to the user

<Message warning>

While the two previous requests to get app integration information can be done
with any valid token including a valid developer token, adding and removing
app integrations requires a valid service account's access token. Using a
developer token will produce a `404 Not Found` error.

</Message>

The following `POST` request can be made to assign an app integration to an app
user:

```curl
curl -X POST https://api.box.com/2.0/app_integration_assignments \
  -H 'Authorization: Bearer [SERVICE_ACCOUNT_TOKEN]' \
  -d '{
    "assignee": {
      "type": "user",
      "id": "[APP_USER_ID]"
    },
    "app_integration": {
      "type": "app_integration",
      "id": "[APP_INTEGRATION_ID]"
    }
  }'
```

```json
{
  "type": "app_integration_assignment",
  "id": "72048301",
  "assignee": {
    "type": "user",
    "id": "6084519920"
  },
  "app_integration": {
    "type": "app_integration",
    "id": "3282"
  }
}
```

The ID in the JSON response can be used to manage app integrations after
assignment, and should be stored by the application.

## Remove integration from user

To remove an app integration from an app user, the following request may be made
with a valid service access token and the app integration assignment ID from the
previous step.

<!-- markdownlint-disable line-length -->

```curl
curl -X DELETE https://api.box.com/2.0/app_integration_assignments/[APP_INTEGRATION_ASSIGNMENT_ID] \
 -H 'Authorization: Bearer [SERVICE_ACCOUNT_TOKEN]'
```

```sh
204 No Content
```

<!-- markdownlint-enable line-length -->

## Browser Support

- Chrome, Firefox, Safari, and Edge (latest 2 versions)
- Limited support for Internet Explorer 11 (requires a `ES2015/Intl polyfill`)
- Mobile Chrome and Safari

<Message warning>

  # ES2015

Box UI Elements require an `ES2015`-capable browser supporting `Intl` (ECMAScript
Internationalization API). If your application supports Internet Explorer 11
or Safari 9, please include a polyfill library or a service like
[`polyfill.io`](https://polyfill.io) to smartly load only the polyfills your
users need. Box also hosts the `core-js` standard library at:

[`https://cdn01.boxcdn.net/polyfills/core-js/2.5.3/core.min.js`][polyfill]

</Message>

<Message warning>

  # Enabling Popups for Adobe Integrations

The Adobe integration currently creates popups that are blocked by the
browsers, enable popups for `echosign.integration.com` to prevent this.

</Message>

## Assets

### Current Version

- 10.1.0

### NPM

- [`www.npmjs.com/package/box-ui-elements`][npm]
- Use this when you are building a React based app and would like to import the
  components directly into your app at build time.

### Scripts and Stylesheets

- Use this when you are not building a React based app or you don't want to
  include the components as part of your app's build process.
- You only need to include `openwith.css` and one of `openwith.js` or
  `openwith.no.react.js`.
- [`openwith.css`][style]
  - Can also be used along with the NPM packaged component.
- [`openwith.js`][openwithjs]
  - Includes React and ReactDOM libraries
  - Use this when your project isn't already including React
  - The file size of this asset will be larger than the one below
- [`openwith.no.react.js`][openwithnoreactjs]
  - Use this when both React and ReactDOM libraries are already loaded on the
    page
  - The content explorer expects `17 > version >= 16.2` of React and ReactDOM
    available on the page

## Supported Locales

The above asset URLs use `en-US`. If you want to use another locale, then
replace `en-US` in the URLs above with any of the following:

`en-AU`, `en-CA`, `en-GB`, `da-DK`, `de-DE`, `es-ES`, `fi-FI`, `fr-CA`, `fr-FR`,
`it-IT`, `ja-JP,`, `ko-KR`, `nb-NO`, `nl-NL`, `pl-PL`, `pt-BR`, `ru-RU`,
`sv-SE`, `tr-TR`, `zh-CN`, `zh-TW`

## Source Code & Releases

Source code for the Open With Element is [hosted on GitHub][gh]. The repository
contains detailed documentation for usage and development. Please file any bugs
you encounter under the "Issues" tab with clear steps to reproduce. This
repository also holds a list of [releases][releases].

## Usage

There are two ways to use the Box Content Open With Element. If you’re looking
to build something quick and simple, use it as a library as shown below in this
documentation. Alternatively, if you are a building a React based app, you can
pull in the component from our NPM package. For details refer to the NPM link
above. As we continue to roll this out, we will provide some level of access to
the source.

<Message>

  Integrations open in a new tab or window, so you may want to notify users as
  some browsers block popups by default.

</Message>

## CORS

To use UI elements an application needs to whitelist the domain the widget is
used on for Cross Origin Resource sharing. See the [CORS guide][cors] for more
details.

## Sample HTML

<!-- markdownlint-disable line-length -->

```html
<!DOCTYPE html>
<html lang="en-US">
  <head>
    <meta charset="utf-8" />
    <title>Box Content Open With Demo</title>

    <!-- polyfill.io only loads the polyfills your browser needs -->
    <script src="https://cdn.polyfill.io/v2/polyfill.min.js?features=es6,Intl"></script>
    <!-- Alternatively, use polyfill hosted on the Box CDN
    <script src="https://cdn01.boxcdn.net/polyfills/core-js/2.5.3/core.min.js"></script>
    -->

    <!-- Latest version of the open with css for your locale -->
    <link
      rel="stylesheet"
      href="https://cdn01.boxcdn.net/platform/elements/{VERSION}/en-US/openwith.css"
    />
  </head>
  <body>
    <div class="container" style="height:600px"></div>
    <!-- Latest version of the core and open with js for your locale -->
    <script src="https://cdn01.boxcdn.net/polyfills/core-js/2.5.3/core.min.js"></script>
    <script src="https://cdn01.boxcdn.net/platform/elements/{VERSION}/en-US/openwith.js"></script>
    <script>
      var fileId = "123";
      var accessToken = "abc";
      var contentOpenWith = new Box.ContentOpenWith();
      contentOpenWith.show(fileId, accessToken, {
        container: ".container"
      });
    </script>
  </body>
</html>
```

## Demo

### Open With Example

<iframe
  height="560"
  scrolling="no"
  title="Box Open With Example"
  src="//codepen.io/box-platform/embed/984598a6fe6bf01785d02be770c5c96a/?height=560&theme-id=27216&default-tab=result&embed-version=2&editable=true"
  frameborder="no"
  allowtransparency="true"
  allowfullscreen="true"
  style="width: 100%;"
>

</iframe>

### Content Explorer + Open With Example

<iframe
  height="560"
  scrolling="no"
  title="Box Content Explorer Example + Open With"
  src="//codepen.io/box-platform/embed/519f67ba709fb581a93c3f73b64cf223/?height=560&theme-id=27216&default-tab=result&embed-version=2&editable=true"
  frameborder="no"
  allowtransparency="true"
  allowfullscreen="true"
  style="width: 100%;"
>

</iframe>

<Message>

  # Access Token

This demos may not fully function until you provide a valid access token. For
testing purposes, you can use your temporary developer token. This will need
to be updated under the JS tab in the demo.

</Message>

<!-- markdownlint-enable line-length -->

## Authentication

The UI Elements are designed in an authentication agnostic way so whether
you are using UI Elements for users who have Box accounts (Managed Users) or
non-Box accounts (App Users), UI Elements should just work out of the box. The
reason for this is that UI Elements only expect a "token" to be passed in for
authentication, and Box provides two different ways to generate tokens - OAuth
and JWT.

<CTA to="g://authentication/select">
  Learn about selecting an authentication method

</CTA>

## Scopes

To execute integrations with downscoped tokens, you must include the
`item_execute_integration` scope as well as the scope required by the specific
integration you would like to use.

- **Google**: `item_readwrite` on the parent folder
- **Adobe**: `root_readwrite`
- **Box Edit**: `item_readwrite` on the parent folder.
- **Box Edit SFC**: `item_readwrite` on the file.

More information on scopes can be found [here][scopes].

## API

```js
const { ContentOpenWith } = Box;
const contentOpenWith = new ContentOpenWith();

/**
 * Shows the content open with element.
 *
 * @param {string} fileId - The root file id
 * @param {string} accessToken - Box API access token
 * @param {Object} [options] - Options
 * @return {void}
 */
contentOpenWith..show(fileId, accessToken, options);

/**
 * Hides the content open with element, removes all event listeners,
 * and clears out the
 * HTML.
 *
 * @return {void}
 */
openWith.hide();

/**
 * Adds an event listener to the content open with element. Listeners should be added
 * before calling show() so no events are missed.
 *
 * @param {string} eventName - Name of the event
 * @param {Function} listener - Callback function
 * @return {void}
 */
contentOpenWith.addListener(eventName, listener);

/**
 * Removes an event listener from the content open with element.
 *
 * @param {string} eventName - Name of the event
 * @param {Function} listener - Callback function
 * @return {void}
 */
contentOpenWith.removeListener(eventName, listener);

/**
 * Removes all event listeners from the content open with element.
 *
 * @return {void}
 */
contentOpenWith.removeAllListeners();
```

<!-- markdownlint-disable line-length -->

### Parameters

| Parameter     | Type   | Description                                                                                                                                                                      |
| ------------- | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `fileId`      | String | Box File ID. This will be the ID of the file for which you would like to execute integrations.                                                                                   |
| `accessToken` | String | Box API access token to use. This should have read/write access to the folder above. The value passed in for the token is assumed to never expire while the explorer is visible. |
| `options`     | Object | Optional options. See below for details. For example: `contentExplorer.show(FOLDER_ID, TOKEN, {canDelete: false})` would be used to hide the delete option.                      |

### Options

| Parameter             | Type           | Description                                                                                                                                                                                                                                                |
| --------------------- | -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `dropdownAlignment`   | `left | right` | Determines the dropdown's alignment to the open with button. The default is `right`.                                                                                                                                                                       |
| `boxToolsName`        | `String`       | This string will replace the name of Box Tools in the "Install Box Tools to open this file on your desktop" message.                                                                                                                                       |
| `boxToolsInstallUrl`  | `String`       | This URL will be used instead of the default Box installation instructions which are linked in the "Install Box Tools to open this file on your desktop" message.                                                                                          |
| `onExecute`           | `Function`     | A callback that executes when an integration invocation is attempted.                                                                                                                                                                                      |
| `onError`             | `Function`     | A callback that executes when an error occurs.                                                                                                                                                                                                             |
| `requestInterceptor`  | `Function`     | Function to intercept requests. For an example see [this CodePen](https://codepen.io/box-platform/pen/jLdxEv). Our underlying XHR library is `axios.js` and we follow a [similar approach for interceptors](https://github.com/axios/axios#interceptors).  |
| `responseInterceptor` | `Function`     | Function to intercept responses. For an example see [this CodePen](https://codepen.io/box-platform/pen/jLdxEv). Our underlying XHR library is `axios.js` and we follow a [similar approach for interceptors](https://github.com/axios/axios#interceptors). |

### Events

| Event Name | Event Data     | Description                                               |
| ---------- | -------------- | --------------------------------------------------------- |
| `execute`  | Integration ID | Will be fired when an integration invocation is executed. |
| `error`    | Error          | Will be fired when an error occurs.                       |

<!-- markdownlint-enable line-length -->

[polyfill]: https://cdn01.boxcdn.net/polyfills/core-js/2.5.3/core.min.js
[style]: https://cdn01.boxcdn.net/platform/elements/10.1.0/en-US/openwith.css
[openwithjs]: https://cdn01.boxcdn.net/platform/elements/10.1.0/en-US/openwith.js
[openwithnoreactjs]: https://cdn01.boxcdn.net/platform/elements/10.1.0/en-US/openwith.no.react.js
[gh]: https://github.com/box/box-ui-elements
[releases]: https://github.com/box/box-ui-elements/releases
[cors]: guide//best-practices/cors
[npm]: https://www.npmjs.com/package/box-ui-elements
[downscope]: guide://authentication/access-tokens/downscope
[scopes]: guide://api-calls/permissions-and-errors/scopes
[community]: https://community.box.com/t5/Box-for-G-Suite-User-Guide/Introducing-Box-for-G-Suite/ta-p/60494
[tools]: https://community.box.com/t5/Box-Tools/ct-p/BoxEdit
[custom-domains]: TODO
[safari]: https://community.box.com/t5/Using-Box-Tools/Installing-Box-Tools/ta-p/50143
