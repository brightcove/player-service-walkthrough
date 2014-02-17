# Player Service Workshop Reference

## Setup
To follow along, you'll need:
 * A computer
 * A terminal application with [CURL](http://curl.haxx.se)
 * A text editor
 * A Video Cloud account

## Documentation
 * [Player Service Tour](http://docs.brightcove.com/en/video-cloud/players/guides/playertour.html)
 * [API Reference](http://docs.brightcove.com/en/video-cloud/player-management-api/reference/versions/v1/index.html)
 * [Using CURL](http://curl.haxx.se/docs/manpage.html)

## Creating a Player
To keep all the code examples consistent, we'll be using environment variables to store the account and player-specific information we need:

```sh
export ACCOUNT='Your Video Cloud account ID'
export EMAIL='The email address for you Video Cloud account'
```

Now, let's create a player:

```sh
curl --user $EMAIL -X POST https://players.api.brightcove.com/v1/accounts/$ACCOUNT/players | python -mjson.tool
```

The response will look something like this:

```sh
{
    "id": "ad40ec13-dcc3-47e8-8465-4d60b4917bac",
    "preview_embed_code": "<iframe src='//players.api.brightcove.com/2335723943001/ad40ec13-dcc3-47e8-8465-4d60b4917bac/preview/index.html' allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe>",
    "preview_url": "https://players.api.brightcove.com/2335723943001/ad40ec13-dcc3-47e8-8465-4d60b4917bac/preview/index.html"
}
```

Save the player ID so we can use it later:

```sh
export PLAYER=ad40ec13-dcc3-47e8-8465-4d60b4917bac
```

If you want, you can paste the `preview_url` into your web browser to see what the player looks like before we start customizing it.

The question you're surely asking now is, how do I get my video in there? To do that, we're going to have to create some configuration for our player. Open up your text editor and type (or copy) this [JSON](http://www.json.org/) into a file called config.json:

```json
{
  "media": {
    "sources": [{
      "src": "http://vjs.zencdn.net/v/oceans.mp4",
      "type": "video/mp4"
    }],
    "poster": {
      "highres": "http://www.videojs.com/img/poster.jpg"
    }
  }
}
```

This configuration specifies the video we want to use through the `media` property. In this case, we're only providing a single source URL that points to an MP4. If you have multiple encodings of the video, you can append them to the `sources` array in the order you'd prefer them to be used.

We'll use curl to update our player with this configuration:

```sh
curl --user $EMAIL -d @config.json -X PATCH -H "Content-Type: application/json" https://players.api.brightcove.com/v1/accounts/$ACCOUNT/players/$PLAYER/configuration | python -mjson.tool
```

Try opening up the `preview_url` in a browser to make sure everything worked.

It's time to get rid of that watermark and create the optimized version of the player for production. Run this curl command to kick that off:

```sh
curl --user $EMAIL -X POST https://players.api.brightcove.com/v1/accounts/$ACCOUNT/players/$PLAYER/publish | python -mjson.tool
```

You should see some new fields in the response this time:

```json
{
    "embed_code": "<iframe src=//players.brightcove.com/2335723943001/ad40ec13-dcc3-47e8-8465-4d60b4917bac_default/index.html allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe>",
    "embed_in_page": "http://players.brightcove.com/2335723943001/ad40ec13-dcc3-47e8-8465-4d60b4917bac_default/in_page.embed",
    "id": "ad40ec13-dcc3-47e8-8465-4d60b4917bac",
    "url": "http://players.brightcove.com/2335723943001/ad40ec13-dcc3-47e8-8465-4d60b4917bac_default/index.html"
}
```

Grab the `url` and try it out in your browser. At this point, you could use this player on your site by pasting in the `embed_code`.

## Customizing a Player
You can further customize your player by adding scripts, stylesheets, and plugins. One common reason for customizing a player is to add advertisements. If you're using Doubleclick for advertisments, you would use the [IMA3 plugin](http://docs.brightcove.com/en/video-cloud/players/plugins/ima-plugin.html) to manage ads for your player. We can add the IMA3 plugin by patching the player we've been working on up to this point. First, modify the configuration you've created for the player:

```json
{
  "media": {
    "sources": [{
      "src": "http://vjs.zencdn.net/v/oceans.mp4",
      "type": "video/mp4"
    }],
    "poster": {
      "highres": "http://www.videojs.com/img/poster.jpg"
    }
  },
  "scripts": [
    "https://players.brightcove.com/videojs/plugins/videojs-ima3/1.1.0/videojs.ima3.min.js"
  ],
  "stylesheets": [
    "https://players.brightcove.com/videojs/plugins/videojs-ima3/1.1.0/videojs.ima3.min.css"
  ],
  "plugins": [{
    "name": "ima3",
    "options": {
      "adSwf": "https://players.brightcove.com/videojs/plugins/videojs-ima3/1.1.0/VideoJSIMA3.swf"
    }
  }]
}
```

You don't actually need to inclue the `media` property since we're going to apply this change as a PATCH but it doesn't hurt either. You can make this update with the same command we used to add the video and poster earlier:

```sh
curl --user $EMAIL -d @config.json -X PATCH -H "Content-Type: application/json" https://players.api.brightcove.com/v1/accounts/$ACCOUNT/players/$PLAYER/configuration | python -mjson.tool
```

Once you're satisfied with the preview, re-publish it:

```sh
curl --user $EMAIL -X POST https://players.api.brightcove.com/v1/accounts/$ACCOUNT/players/$PLAYER/publish | python -mjson.tool
```

## Sharing Configuration Defaults
If you have many players on your site or would like to experiment with different player configurations, it often makes sense to share some configuration defaults. Any player created by the service can be used as the basis for as many variations, called *embeds*, as you'd like.

Let's start by creating an embed that just overrides the poster image. First, create a new configuration file called embed1.json:

```json
{
  "media": {
    "poster": {
      "highres": "//players.brightcove.com/2335723943001/service-webinar/oceans-alternate.jpg"
    }
  }
}
```

Then apply that configuration to your embed:

```sh
curl --user $EMAIL -d @embed1.json -X POST -H "Content-Type: application/json" https://players.api.brightcove.com/v1/accounts/$ACCOUNT/players/$PLAYER/embeds | python -mjson.tool
```

Make sure to save the embed `id` for later:

```sh
export EMBED1=c1730f23-3d9f-4b8f-a643-e581cd24ba27
```

You can create another embed from the same player with an alternate stylesheet, too. Create a file called embed2.json:

```json
{
  "stylesheets": [
    "https://players.brightcove.com/videojs/plugins/videojs-ima3/1.1.0/videojs.ima3.min.css",
    "https://players.brightcove.com/2335723943001/service-webinar/alternate.css"
  ]
}
```

Then create the embed:

```sh
curl --user $EMAIL -d @embed2.json -X POST -H "Content-Type: application/json" https://players.api.brightcove.com/v1/accounts/$ACCOUNT/players/$PLAYER/embeds | python -mjson.tool
```

and save the embed code again:

```sh
export EMBED2=c69a67a8-ae21-4192-bd6c-00ccb5c85baf
```

Now, you can publish both embeds and see how the configuration overrides are applied:

```sh
curl -u $EMAIL -X POST https://players.api.brightcove.com/v1/accounts/$ACCOUNT/players/$PLAYER/embeds/$EMBED1/publish | python -mjson.tool
curl -u $EMAIL -X POST https://players.api.brightcove.com/v1/accounts/$ACCOUNT/players/$PLAYER/embeds/$EMBED2/publish | python -mjson.tool
```

Copy the `url` into your browser to check them out or drop the `embed_code`s into your site to begin using them.
