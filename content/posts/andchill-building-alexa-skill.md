---
title: "andChill: Building an Alexa Skill with Spotify and Hue"
date: 2018-03-11T13:05:38+01:00
draft: true
---

I've bought an Amazon Echo Dot as an entry into voice-controlling my home and I quite like it. It reacts fast enough, the voice recognition is good and it works beautifully with my Phillips Hue Lamps.

There is just one thing missing that annoys me: No macros. Often I'm coming home late in the evening and just want some music to play and the lights to be turned on. The voice recognicion is good, but it get's tiring to say "Alexa, play Spotify playlist evening" and "Alexa, turn on the bedroom lights on scene `Evening`".

Recently Amazon introduced Routines which sounded like the perfect fit. But after setting up my lights, I realized that there is no "Music" option to select.

So I guess I have to do it myself. In this blog post I'll walk you through creating your own Alexa Skill with NodeJS, Spotify and Phillips Hue. The source code is on GitHub.

<blockquote class="twitter-video" data-lang="en"><p lang="de" dir="ltr">Ist es noch prokrastinieren, wenn doch was halbwegs sinnvolles rauskommt? <a href="https://t.co/buwkuzJ1mt">pic.twitter.com/buwkuzJ1mt</a></p>&mdash; Simon (@der_simon) <a href="https://twitter.com/der_simon/status/955061756753301504?ref_src=twsrc%5Etfw">January 21, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>



# Step 1: Spotify

As debugging individual parts is easier than some big thing I started by creating a script to play my music. 
It is possible for Alexa Skills to play music, but you have to provide your own. As I already have quite some music on Spotify, I decided to look into the Spotify API.
[https://developer.spotify.com/](Their documentation) is quite good and you can try calls directly inside the browser.

The relevant call for this project is 
https://developer.spotify.com/web-api/start-a-users-playback/
```
PUT https://api.spotify.com/v1/me/player/play
```
It is possible to specify a device as a query parameter and an album/playlist/song to play in the body - perfect.
To start using the Spotify API, we'll need to create a Spotify App first https://beta.developer.spotify.com/dashboard/applications
Simply enter a name for the app, a short description and agree to the T&C. You'll get a Client ID and Client Secret for authentication. 

Going back to the nodeJS part:

Reinventing the wheel is possible, but using the `spotify-web-api-node` library seemed like a good idea. Installation as usual by running 
`npm install spotify-web-api-node --save`

In the `spotifyTest.js` we'll start by importing the library and setting up the client.

```js
const SpotifyWebApi = require('spotify-web-api-node');

let spotifyApi = new SpotifyWebApi({
  clientId : 'CEDENTIALS-FROM-PREVIOUS-STEP',
  clientSecret : 'CEDENTIALS-FROM-PREVIOUS-STEP',
  redirectUri : 'http://simon-schraeder.de/spotifyTestBlah'
});
```

Before authorizing our user to the app, we will need to define the required permissions. As we want to play music, the Spotify API docs list `user-modify-playback-state` as the required permission. To start the playback we'll also need to Device ID. This requires the `user-read-playback-state` permission. And as we want to play a playlist, we'll need the  `playlist-read-private` to get the playlist ID. It is also possible to do this manually, but why not automate everything? :)

```js
const scopes = ['user-modify-playback-state','user-read-playback-state', 'playlist-read-private'];
const state = 'some-state-of-my-choice';
```

The state is not too important at the moment. In order to authorize our user to the app, we'll need to generate an authorization URL:

```js
const authorizeURL = spotifyApi.createAuthorizeURL(scopes, state);
console.log(authorizeURL);
```
The console contains an URL starting with `accounts.spotify.com`. But opening this one won't work at the moment, we'll need to whitelist our URL in the Spotify API settings first.

Click on "Edit Settings" on the application screen and enter your redirect URL. Opening the URL from the console works now. Simply approve the app and you'll get redirected to your specified redirect URL. The URL parameter will contain a code. 

We can trade this access code into a access and refresh token with a quick API call.

```js
spotifyApi.authorizationCodeGrant(config.spotifyConfig.accessCode).then(function(data) {
    console.log("Access Token:" + data.body['access_token']+  "\n");
    console.log('Refresh token ' + data.body['refresh_token']);
    spotifyApi.setAccessToken(data.body['access_token']);
    spotifyApi.setRefreshToken(data.body['refresh_token']);
}).catch(console.log);
```
We'll need the access and refresh token later on, so it's best to take note of them.
Instead of creating an authorization URL, we'll just set the access and refresh tokens to access the API.

```js
spotifyApi.setAccessToken(config.spotifyConfig.accessToken);
spotifyApi.setRefreshToken(config.spotifyConfig.refreshToken);
```

For the next step, we'll need to grab the playlist ID. We do this by simply iterating over all playlists and logging the name and ID.
```js
spotifyApi.getUserPlaylists().then((response) => {
    for(playlist of response.body.items) {
        console.log(`${playlist.name} : ${playlist.id} \n`);
    }
}).catch(console.log);
```

We do the same for the devices.

```js
 spotifyApi.getMyDevices().then((response) => {
    for(device of response.body.devices) {
        console.log(`${device.name} : ${device.id} \n`);
    }
}).catch(console.log)
```

Take note of the playlist name, playlist id, device name and device ID.
The Spotify API uses "Context URIs" for playback, so we'll need to generate this. The structure is simply
`spotify:user:${username}:playlist:${targetPlaylist}`.

A quick call to 
```js
spotifyApi.play({
  context_uri: context_uri,
  device_id: device_id,
})
```
should play our playlist on the selected device.

I decided to wrap my Spotify Code into a wrapper class. You can check it out on GitHub.


# Part 2: Hue

The Hue API is quite simple. You specify a target group, a target scene and the state (on: true) and you are done. The API requires a token for authentication, but it seems to be account-specific and does not expire.

```js
return request
  .put(`/${targetGroup}/action`)
  .send({
     "on": true,
      "scene": targetSceneName
    })
    .set('x-token', token);
```
As the API is not yet public, discovering the API url is left as an excercise for the reader.


# Part 3: Alexa

To create an Alexa Skill, we simply use the `alexa-sdk` library. 

```js
const Alexa = require('alexa-sdk');
const APP_ID = 'amzn1.ask.skill.';
```

The index.js needs to export a handler function that returns specific handlers.

```js
exports.handler = function(event, context, callback) {
    const alexa = Alexa.handler(event, context, callback);
    alexa.registerHandlers(handlers);
    alexa.execute();
};
```

As we don't want complex dialogs and just call our code on launch, we simply create a handler for the `LaunchRequest`. We call our code and return a simple response. 

```js
const handlers = {
    'LaunchRequest': function () {
        ourCode();
        this.response
        .cardRenderer('Stated music', 'Enjoy your evening!')
        this.emit(':responseReady');
	},
};
```

The LaunchRequest handler calls ourCode(), so we'll need to create a function handling the calls. Instead of hard-coding the playlists and devices, I decided to move them into environment variables. This makes changing the devices and playlists easier and makes sure we store no sensitive data in the code.

```js
const targetDevice = process.env.spotifyTargetDeviceName;
const targetPlaylist = process.env.spotifyTargetPlaylist;
const targetUsername = process.env.spotifyTargetUsername;
```

Putting it together we simply refresh the Spotify API token, fetch the device ID, play the music and turn the light on.

```js
 
function playMusic() {
    return spotifyHelper.refreshAPIToken().then(() => {
        return spotifyHelper.getDeviceIDByName(targetDevice);
    }).then((targetDeviceID) =>{
        let targetUri = spotifyHelper.getContextUri(targetUsername, targetPlaylist);
        return spotifyHelper.play(targetDeviceID, targetUri);
    }).catch(console.log);
}

function setLight() {
    return hueHelper.turnLightOn();
}

function ourCode() {
    playMusic().then(setLight).catch(console.log);
}
```


To create our Alexa skill, we'll head to the [Amazon Developer Console](https://developer.amazon.com/edw/home.html#/).  
Click on `Alexa Skills Kit` and `Create new Skill`. There are pre-build skills types, but we'll create a custom one. 
![Skill](/static/img/skill.png)


We don't care about Invocation or intents about the moment, but click on Endpoint. Copy the `Skill ID` to your clipboard.
Head to [AWS](http://aws.amazon.com/) AWS, nagivate to Lambda and create a new function. We'll create a function from scratch. Enter a name and click on `Create Function`. Select `Alexa Skills Kit` from the left as a trigger. 

![Lambda](/static/img/lambda.png)

Save the changes and upload our .zip file. 

![Function](/static/img/function.png)

The `Handler` in the right box specifies the function that is called on invocation. "index.handler" in this case means the `function handler()` in the `index.js`. The section `Environment Variables` below allows us to store the tokens and `targetDevices` we defined in our code earlier. You can click on `Test` to test the code and make sure the music starts and the lights are dimmed.

If everything works, copy the `ARN` in the top-right section. Head back to the Alexa Developer Console and paste the `ARN` into the `Default Region`. Save the endpoint. This links our Alexa skill to the Lambda endpoint.

![Invocation](/static/img/invocation.png)

Now we'll need to define a invocation name. This is the name you'll say to start the action. I decided to go with `Chill` as it's easy to pronounce (and Netflix might be implemented as well :)) 

We only want to invoce this simple skill by saying `Alexa, open chill` but we still need to define one utterances to build to Invocation Model. Simply click on `HelpIntent` enter anything there. Building the Interaction Model is then possible.
![Utterance](/static/img/utter.png)


After waiting a few seconds to build the model, we can finally activate the test. The skill is now accessible from your Alexa. Simply say "Alexa, open chill" and the music will start playing :)

![Test](/static/img/tes.png)


Check out my [GitHub](https://github.com/c0dr/andChill)for a ready-made .zip to upload to AWS Lambda. The source code with quite a bit of wrapper code is on there as well.
