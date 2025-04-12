# Frequently asked questions

Please read those before filing an issue, your question may be answered there.

## How do I do *X* with fluent-ffmpeg?

Please ensure first that you know how to achieve what you want on command line. If you don't, have a look at the [ffmpeg documentation][ffmpegdoc] and ask the [ffmpeg community][ffmpegcom] first, as you're far more likely to get an appropriate answer there.

When you know how to achieve your goal on command line and don't know how to translate it to fluent-ffmpeg calls, please ensure that you've [read the documentation][doc] first.  If you find some command line switch that has no matching fluent-ffmpeg method, you can use the `inputOptions()` and `outputOptions()` method instead.

If you can't find how to do what you want, feel free to create an issue.  Beware though this is not the most appropriate place to talk about ffmpeg usage (try their mailing list or irc channels instead).

## My command is not executed.

Please ensure that you added handlers for **both** the `end` and `error` handlers.  One of those should be called and give you more information about what happened.

## How can I add filters/options when taking screenshots or using `mergeToFile`?

Short answer: you can't. Do it manually with custom options/filters.

Longer answer: screenshot and merge methods are just helper "recipes" to help achieve common actions; as such they have a limited scope and limited possible options.  Screenshots methods in particular are already quite complex to cope with the whole scope of options they provide.

If you need something more specific, you're probably better off writing your own implementation.  Feel free to re-use the code in `lib/recipes.js` as a base for your implementation (and you may still create issues if you have questions with the code).

## Common error messages

### Error: ffmpeg exited with code 1

Fluent-ffmpeg tries to extract error messages from ffmpeg output, but sometimes it fails.  In that case, you can look at additional arguments passed to the `error` event handler: they hold the raw output from ffmpeg:

```js
ffmpeg(...)
  .on('error', function(err, stdout, stderr) {
    console.log('Error: ' + err.message);
    console.log('ffmpeg output:\n' + stdout);
    console.log('ffmpeg stderr:\n' + stderr);
  });
```

Most of the time you can get a clearer explanation this way.  If you still don't get what's wrong, please include stdout and stderr contents in your bug report.

### Unable to find a suitable output format for 'pipe:1' - pipe:1: Invalid argument

When you get those messages in stderr, you're most likely outputting to a pipe (or stream in node) and forgot to set an output format. Ffmpeg can only automatically determine an output format (from the file extension) when outputting to a file. You need to add a `.format()` call to your command.

### Output stream closed

When piping the output to an express stream, you may see that error happen from time to time.  It happens when the browser closes the connection by itself, not waiting for the whole media to download, so that may well explain this error when it happens infrequently.

However you may see it on *each* browser request.  This is because many browsers actually make *two* requests when trying to stream a media.  The first one will serve to determine the file format and codecs, and will stop after a few hundred bytes are downloaded, hence the error.  The browser will then proceed to the actual streaming with a second (most likely chunked) request.

### Muxer does not support non seekable output

This occurs when trying to stream a format that does not support it, with `mp4` being the most common example.  The errors happens because the muxer code needs to "go back" to the beginning of the output to write some kind of header information after having finished muxing.

In the case of the `mp4` format (or `mov` and `flv`, which are the same), you can add the `-movflags frag_keyframe+faststart` option using `command.outputOption('-movflags frag_keyframe+faststart')`.  For other formats, please refer to the [ffmpeg documentation][ffmpegdoc].


[doc]: README.md#fluent-ffmpeg-api-for-nodejs-
[ffmpegdoc]: http://ffmpeg.org/documentation.html
[ffmpegcom]: http://ffmpeg.org/contact.html
