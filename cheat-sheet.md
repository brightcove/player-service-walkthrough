# Player Service Workshop Reference

## Setup
To follow along, you'll need:

 * A computer
 * A terminal application with [CURL](http://curl.haxx.se)
 * A text editor
 * A Video Cloud account
 * Your Video Cloud Account ID (you can get this from right under the welcome message in the [Video Cloud Studio](https://videocloud.brightcove.com/))

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

If you want, you can paste the `preview_url` into your web browser to see what the player looks like before we start customizing it.  You will see your player settings applied to a test video.  You will also see a Brightcove watermark.  This is a friendly reminder that this is not the player that you want to publish on your site.

## Customizing Your Player

Now that you have a player you will likely want to make it your own.  To do that, we're going to have to create some configuration for our player. Open up your text editor and type (or copy) this [JSON](http://www.json.org/) into a file called config.json:

```json
{
  "stylesheets": [
    "https://players.brightcove.com/2335723943001/service-webinar/alternate.css"
  ]
}
```

This configuration specifies a different stylesheet that you would like applied to your player.  This will change the color of the volume control from fuschia to orange.

We'll use curl to update our player with this configuration:

```sh
curl --user $EMAIL -d @config.json -X PATCH -H "Content-Type: application/json" https://players.api.brightcove.com/v1/accounts/$ACCOUNT/players/$PLAYER/configuration | python -mjson.tool
```

## Creating Embeds

The question you're surely asking now is, how do I get my video in there? To do that we use embeds.  Embeds are finished instances of your player.  They combine your player settings with your media.  Once embeds are published production ready player files are pushed out to the CDN and are ready for you to use on your site.

A player can have many different embeds.  This allows you to make configuration changes at the player level and have those changes apply to all of the embeds.

Creating an embed is similar to configuring a player. In fact you pass in configuration settings that can add to or override settings that you have set up for your player.  Here we will just add.  Open up your text editor and type (or copy) this [JSON](http://www.json.org/) into a file called embed.json:

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
curl --user $EMAIL -d @embed.json -X POST -H "Content-Type: application/json" https://players.api.brightcove.com/v1/accounts/$ACCOUNT/players/$PLAYER/embeds | python -mjson.tool
```

This will return the ID for the new embed like so:

```json
{"id":"4ab5e8e3-5924-4496-95ae-1b3d0b4c9ba0"}
```

Make sure to save the embed `id` for later:

```sh
export EMBED=4ab5e8e3-5924-4496-95ae-1b3d0b4c9ba0
```

## Publishing Your Embed
(This is temporary, very soon embeds will be published automatically when you create them.)

Once your embed is created you need to publish it in order to start using it.  This will get rid of that watermark and create the optimized version of the player for production.  To do so run this:

```sh
curl --user $EMAIL -X POST https://players.api.brightcove.com/v1/accounts/$ACCOUNT/players/$PLAYER/embeds/$EMBED/publish | python -mjson.tool
```

The result will look like this:

```json
{
    "embed_code": "<iframe src='//players.brightcove.com/507787973001/07d9f51d-60ae-4b7f-99c2-f81edcc65b95_4ab5e8e3-5924-4496-95ae-1b3d0b4c9ba0/index.html' allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe>",
    "url": "http://players.brightcove.com/507787973001/07d9f51d-60ae-4b7f-99c2-f81edcc65b95_4ab5e8e3-5924-4496-95ae-1b3d0b4c9ba0/index.html"
}
```

You should see some new fields in the response this time.  Notice there are no longer preview* items.  Instead you will see url and embed_code.  Grab the `url` and try it out in your browser. 

At this point, you could use this player on your site by pasting in the `embed_code`.  There will also be an in page embed added to this response.  For now you can access it by using the URL and replacing 'index.html' with 'in_page.embed'.

If you would like you could create more embeds at this time.

## Customizing The Player
You can further customize your player by adding scripts, stylesheets, and plugins even after your embeds are created. Customizing your player will have the effect of updating all embeds that were created from that player.  

One common reason for customizing a player is to add advertisements. If you're using Doubleclick for advertisements, you would use the [IMA3 plugin](http://docs.brightcove.com/en/video-cloud/players/plugins/ima-plugin.html) to manage ads for your player. We can add the IMA3 plugin by patching the player we've been working on up to this point. Save this as config.json:

```json
{
  "scripts": [
    "https://players.brightcove.com/videojs/plugins/videojs-ima3/1.1.0/videojs.ima3.min.js"
  ],
  "stylesheets": [
    "https://players.brightcove.com/videojs/plugins/videojs-ima3/1.1.0/videojs.ima3.min.css",
    "https://players.brightcove.com/2335723943001/service-webinar/alternate.css"
  ],
  "plugins": [{
    "name": "ima3",
    "options": {
      "adSwf": "https://players.brightcove.com/videojs/plugins/videojs-ima3/1.1.0/VideoJSIMA3.swf"
    }
  }]
}
```

One thing to notice about this patch.  In most cases a patch will add your changes to the existing configuration object.  The one exception is with arrays which will be replaced entirely.  For this reason you should notice that the stylesheet that we added earlier must be included again when altering the "stylesheets" array.

You can make this update with the same command we used to add the stylesheet earlier:

```sh
curl --user $EMAIL -d @config.json -X PATCH -H "Content-Type: application/json" https://players.api.brightcove.com/v1/accounts/$ACCOUNT/players/$PLAYER/configuration | python -mjson.tool
```

Test out the preview once again.  This time you will see a preroll ad when you push play.  Once you're satisfied with the preview you can publish the player to have the new settings be applied to all of the player's embeds:

```sh
curl --user $EMAIL -X POST https://players.api.brightcove.com/v1/accounts/$ACCOUNT/players/$PLAYER/publish | python -mjson.tool
```

After a brief processing time when you refresh your published embed URL(s) you should see the preroll ad there as well.
