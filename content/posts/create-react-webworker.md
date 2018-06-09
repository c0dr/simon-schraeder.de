---
date: "2018-06-09T13:45:19+02:00"
title: "Configuring the create-react-app ServiceWorker"
authors: []
categories:
  -
tags:
  -
draft: false
---

I recently created a new React app with the `create-react-app` boilerplate tool by Facebook. It's quite nice, but the `ServiceWorker` configuration is limited to the default output files by webpack. For my `CardOnly.de` website I wanted to cache the JSON data of the cards and filters as well. The process is actually not too complex:


## Step 1: Eject from the default configuration

To edit the webpack configuration files, we'll need to eject by running `npm run eject`.
This removes the depency on `create-react-app` and instead adds the dependencies for `webpack` and others directly.
This is a one-way-operation, so you might want to back it up if you are not using version control.

## Step 2: Configure the webpack script

The newly created `/config` folder now contains the webpack scripts used by `create-react-app`. We'll need to open the `config/webpack.config.prod.js` file.

The `module.exports.plugins` contains some of the webpack plugins including the `SWPrecacheWebpackPlugin` plugin.
As this plugin is just a webpack integration around [sw-precache](https://github.com/GoogleChromeLabs/sw-precache#options-parameter), we can pass any options parameter into it.
The `staticFileGlobs` parameter specifies string patterns for files the ServiceWorker should precache.

In my case I want to precache the files in `public/data/*`. so the configuration looks like this

`staticFileGlobs: ['public/data/*'],`


If the location of the files during runtime is not the same as the config, you'll need to set `stripPrefix` to remove certain folder prefixes.

By default the `SWPrecacheWebpackPlugin` uses webpacks `output.path` for the `stripPrefix` parameter and this is by default set to `public` in `create-react-app`, so we're good in this case.

The last thing we need to set is the `mergeStaticsConfig` property to `true`. This makes sure our `staticFileGlobs` parameter is merged with the paths of the assets emitted by webpack.

In total we simply added the following lines to `config/webpack.config.prod.js`:

```js

module.exports = {
  // [...]
plugins: [
  // [...]
  new SWPrecacheWebpackPlugin({
      // [...]
      staticFileGlobs: ['public/data/*'],
      mergeStaticsConfig: true,
    }),
// [...]

```

Voilà . Now even our files in the data folder end up being precached and serverd by the ServiceWorker.

![Skill](/static/img/serviceworker.png)