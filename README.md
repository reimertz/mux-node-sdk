![Mux Node Banner](github-nodejs-sdk.png)

# Mux Node

![build status](https://api.travis-ci.org/muxinc/mux-node-sdk.svg?branch=master) ![npm version](https://badge.fury.io/js/%40mux%2Fmux-node.svg)

Official Mux API wrapper for Node projects, supporting both Mux Data and Mux Video.

[Mux Video](https://mux.com/video) is an API-first platform, powered by data and designed by video experts to make beautiful video possible for every development team.

[Mux Data](https://mux.com/data) is a platform for monitoring your video streaming performance with just a few lines of code. Get in-depth quality of service analytics on web, mobile, and OTT devices.

This library is intended to provide Mux API convenience methods for applications written in server-side Javascript. **Please note** that this package uses Mux access tokens and secret keys and is intended to be used in server-side code only.

Not familiar with Mux? Check out https://mux.com/ for more information.

## Documentation

See the [Mux-Node docs](https://muxinc.github.io/mux-node-sdk)

## Installation

```
npm install @mux/mux-node --save
```

or

```
yarn add @mux/mux-node
```

## Usage

To start, you will need a Mux access token and secret for your Mux environment. For more information on where to get
an access token, visit the Mux Getting Started guide https://docs.mux.com/docs

Require the `@mux/mux-node` npm module and create a Mux instance. Your Mux instance will have `Data` and `Video` properties
that will allow you to access the Mux Data and Video APIs.

```javascript
const Mux = require('@mux/mux-node');
const { Video, Data } = new Mux(accessToken, secret);
```

If a token ID and secret aren't included as parameters, the SDK will attempt to use the `MUX_TOKEN_ID` and `MUX_TOKEN_SECRET` environment variables.

```javascript
// assume process.env.MUX_TOKEN_ID and process.env.MUX_TOKEN_SECRET contain your credentials
const muxClient = new Mux(); // Success!
```

As an example, you can create a Mux asset and playback ID by using the below functions on your Video instance.

```javascript
// Create an asset
const asset = await Video.Assets.create({
  input: 'https://storage.googleapis.com/muxdemofiles/mux-video-intro.mp4',
});
```

```javascript
// ...then later, a playback ID for that asset
const playbackId = await Video.Assets.createPlaybackId(asset.id, {
  policy: 'public',
});
```

Or, if you don't have the files online already, you can ingest one via the direct uploads API.

```javascript
const request = require('request');
let upload = await Video.Uploads.create({
  new_asset_settings: { playback_policy: 'public' },
});

// The URL you get back from the upload API is resumable, and the file can be uploaded using a `PUT` request (or a series of them).
await fs.createReadStream('/path/to/your/file').pipe(request.put(upload.url));

// The upload may not be updated immediately, but shortly after the upload is finished you'll get a `video.asset.created` event and the upload will now have a status of `asset_created` and a new `asset_id` key.
let updatedUpload = await Video.Uploads.get(upload.id);

// Or you could decide to go get additional information about that new asset you created.
let asset = await Video.Assets.get(updatedUpload['asset_id']);
```

You can access the Mux Data API in the same way by using your Data instance. For example, you can list all of the
values across every breakdown for the `aggregate_startup_time` metric by using the below function.

```javascript
const breakdown = await Data.Metrics.breakdown('aggregate_startup_time', {
  group_by: 'browser',
});
```

## Usage Details

Every function will return a chainable [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).

```javascript
Video.Assets.create({
  input: 'https://storage.googleapis.com/muxdemofiles/mux-video-intro.mp4',
}).then(asset => {
  /* Do things with the asset */
});
```

## Verifying Webhook Signatures

Learn more about verifying webhook headers in our [Webhooks Security Guide](https://docs.mux.com/docs/webhook-security)


```javascript
/*
  If the header is valid, this will return `true`
  If invalid, this will throw one of the following errors:
    * new Error('Unable to extract timestamp and signatures from header')
    * new Error('No signatures found with expected scheme');
    * new Error('No signatures found matching the expected signature for payload.')
    * new Error('Timestamp outside the tolerance zone')
*/

/*
  `payload` is the raw request body. It should be a string representation of a JSON object.
  `header` is the value in request.headers['Mux-Signature']
  `secret` is the signing secret for this configured webhook. You can find that in your webhooks dashboard
           (note that this secret is different than your API Secret Key)
*/

Mux.Webhooks.verifyHeader(payload, header, secret);
```

## JWT Helpers <small>([API Reference](https://muxinc.github.io/mux-node-sdk/class/src/utils/jwt.js~JWT.html))</small>

You can use any JWT-compatible library, but we've included some light helpers in the SDK to make it easier to get up and running.

```javascript
// Assuming you have your signing key specified in your environment variables:
// Signing token ID: process.env.MUX_SIGNING_KEY
// Signing token secret: process.env.MUX_PRIVATE_KEY

// Most simple request, defaults to type video and is valid for 7 days.
const token = Mux.JWT.sign('some-playback-id');
// https://stream.mux.com/some-playback-id.m3u8?token=${token}

// If you wanted to sign a thumbnail
const thumbParams = { time: 14, width: 100 }
const thumbToken = Mux.JWT.sign('some-playback-id', { type: 'thumbnail', params: thumbParams });
// https://image.mux.com/some-playback-id/thumbnail.jpg?token=${token}

// If you wanted to sign a gif
const gifToken = Mux.JWT.sign('some-playback-id', { type: 'gif' });
// https://image.mux.com/some-playback-id/animated.gif?token=${token}
```

## `request` and `response` events

The SDK returns the `data` key for every object, because in the Mux API that's always the thing you actually want to see. Sometimes, however, it's useful to see more details about the request being made or the full response object. You can listen for `request` and `response` events to get these raw objects.

```javascript
muxClient.on('request', req => {
  // Request will contain everything being sent such as `headers, method, base url, etc
});

muxClient.on('response', res => {
  // Response will include everything returned from the API, such as status codes/text, headers, etc
});
```

See the [Mux-Node docs](https://muxinc.github.io/mux-node-sdk/identifiers.html) for a list of all available functions.

## Development

Run unit tests: `npm test` or `npm run test:unit`

Run integration tests: `npm run test:int`

**Note**: running the integration tests will require you to configure the `MUX_TOKEN_ID` and `MUX_TOKEN_SECRET` environment variables with your Mux access token and secret.

To generate the ESDocs, run:

```
yarn esdoc
open ./docs/index.html
```

## Contributing

Find a bug or want to add a useful feature? That'd be amazing! If you'd like to submit a [pull request](https://help.github.com/articles/about-pull-requests/) to the project with changes, please do something along these lines:

1. Fork the project wherever you'd like
2. Create a meaningful branch name that relates to your contribution. Consider including an issue number if available. `git co -b add-node-lts-support`
3. Make any changes you'd like in your forked branch.
4. Add any relevant tests for your changes
5. Open the pull request! :tada:
