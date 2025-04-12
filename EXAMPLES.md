# Examples

This document provides a series of examples demonstrating the use of the `fluent-ffmpeg` Node.js module. Examples are ordered by increasing complexity.
## Any To Mp4 Steam

This example demonstrates how to convert any input stream to MP4 with special `movflags` for streaming.

```js
// The solution based on adding -movflags for mp4 output
// For more movflags details check ffmpeg docs
// https://ffmpeg.org/ffmpeg-formats.html#toc-Options-9

var fs = require('fs');
var path = require('path');
var ffmpeg = require('../index');

var pathToSourceFile = path.resolve(__dirname, '../test/assets/testvideo-169.avi');
var readStream = fs.createReadStream(pathToSourceFile);
var writeStream = fs.createWriteStream('./output.mp4');

ffmpeg(readStream)
  .addOutputOptions('-movflags +frag_keyframe+separate_moof+omit_tfhd_offset+empty_moov')
  .format('mp4')
  .pipe(writeStream);

```
## Express Stream

This example sets up an Express.js server that streams converted video on-the-fly to a browser.

```js
var express = require('express'),
  ffmpeg = require('../index');

var app = express();

app.use(express.static(__dirname + '/flowplayer'));

app.get('/', function(req, res) {
  res.send('index.html');
});

app.get('/video/:filename', function(req, res) {
  res.contentType('flv');
  // make sure you set the correct path to your video file storage
  var pathToMovie = '/path/to/storage/' + req.params.filename;
  var proc = ffmpeg(pathToMovie)
    // use the 'flashvideo' preset (located in /lib/presets/flashvideo.js)
    .preset('flashvideo')
    // setup event handlers
    .on('end', function() {
      console.log('file has been converted succesfully');
    })
    .on('error', function(err) {
      console.log('an error happened: ' + err.message);
    })
    // save to stream
    .pipe(res, {end:true});
});

app.listen(4000);

```
## Full

This is a full-featured example demonstrating many of the available configuration options.

```js
var ffmpeg = require('../index');

// make sure you set the correct path to your video file
var proc = ffmpeg('/path/to/your_movie.avi')
  // set video bitrate
  .videoBitrate(1024)
  // set target codec
  .videoCodec('divx')
  // set aspect ratio
  .aspect('16:9')
  // set size in percent
  .size('50%')
  // set fps
  .fps(24)
  // set audio bitrate
  .audioBitrate('128k')
  // set audio codec
  .audioCodec('libmp3lame')
  // set number of audio channels
  .audioChannels(2)
  // set custom option
  .addOption('-vtag', 'DIVX')
  // set output format to force
  .format('avi')
  // setup event handlers
  .on('end', function() {
    console.log('file has been converted succesfully');
  })
  .on('error', function(err) {
    console.log('an error happened: ' + err.message);
  })
  // save to file
  .save('/path/to/your_target.avi');

```
## Image2Video

This example converts a still image into a short video by looping the image.

```js
var ffmpeg = require('fluent-ffmpeg');

// make sure you set the correct path to your video file
var proc = ffmpeg('/path/to/your_image.jpg')
  // loop for 5 seconds
  .loop(5)
  // using 25 fps
  .fps(25)
  // setup event handlers
  .on('end', function() {
    console.log('file has been converted succesfully');
  })
  .on('error', function(err) {
    console.log('an error happened: ' + err.message);
  })
  // save to file
  .save('/path/to/your_target.m4v');

```
## Input Stream

This example shows how to use a readable stream as input for processing.

```js
var fs = require('fs'),
  ffmpeg = require('../index');

// open input stream
var infs = fs.createReadStream(__dirname + '/test/assets/testvideo-43.avi');

infs.on('error', function(err) {
  console.log(err);
});

// create new ffmpeg processor instance using input stream
// instead of file path (can be any ReadableStream)
var proc = ffmpeg(infs)
  .preset('flashvideo')
  // setup event handlers
  .on('end', function() {
    console.log('done processing input stream');
  })
  .on('error', function(err) {
    console.log('an error happened: ' + err.message);
  })
  // save to file
  .save('/path/to/your_target.flv');

```
## Livertmp2Hls

This example shows how to convert a live RTMP stream to HLS format in real-time.

```js
var ffmpeg = require('../index');

// make sure you set the correct path to your video file
var proc = ffmpeg('rtmp://path/to/live/stream', { timeout: 432000 })
  // set video bitrate
  .videoBitrate(1024)
  // set h264 preset
  .addOption('preset','superfast')
  // set target codec
  .videoCodec('libx264')
  // set audio bitrate
  .audioBitrate('128k')
  // set audio codec
  .audioCodec('libfaac')
  // set number of audio channels
  .audioChannels(2)
  // set hls segments time
  .addOption('-hls_time', 10)
  // include all the segments in the list
  .addOption('-hls_list_size',0)
  // setup event handlers
  .on('end', function() {
    console.log('file has been converted succesfully');
  })
  .on('error', function(err) {
    console.log('an error happened: ' + err.message);
  })
  // save to file
  .save('/path/to/your_target.m3u8');

```
## Mergevideos

This example merges multiple video files into one using intermediate MPEG files.

```js
var ffmpeg = require('../index');

/*
 replicates this sequence of commands:

 ffmpeg -i title.mp4 -qscale:v 1 intermediate1.mpg
 ffmpeg -i source.mp4 -qscale:v 1 intermediate2.mpg
 ffmpeg -i concat:"intermediate1.mpg|intermediate2.mpg" -c copy intermediate_all.mpg
 ffmpeg -i intermediate_all.mpg -qscale:v 2 output.mp4

 Create temporary .mpg files for each video and deletes them after merge is completed.
 These files are created by filename pattern like [videoFilename.ext].temp.mpg [outputFilename.ext].temp.merged.mp4
 */

var firstFile = "title.mp4";
var secondFile = "source.mp4";
var thirdFile = "third.mov";
var outPath = "out.mp4";

var proc = ffmpeg(firstFile)
    .input(secondFile)
    .input(thirdFile)
    //.input(fourthFile)
    //.input(...)
    .on('end', function() {
      console.log('files have been merged succesfully');
    })
    .on('error', function(err) {
      console.log('an error happened: ' + err.message);
    })
    .mergeToFile(outPath);
```
## Metadata

This example demonstrates how to use `ffprobe` to extract metadata from a video file.

```js
var ffmpeg = require('../index');

// make sure you set the correct path to your video file
ffmpeg.ffprobe('/path/to/your_movie.avi',function(err, metadata) {
  console.log(require('util').inspect(metadata, false, null));
});

```
## Preset

This example demonstrates the use of a preset and how to override its settings.

```js
var ffmpeg = require('../index');

// make sure you set the correct path to your video file
var proc = ffmpeg('/path/to/your_movie.avi')
  // use the 'podcast' preset (located in /lib/presets/podcast.js)
  .preset('podcast')
  // in case you want to override the preset's setting, just keep chaining
  .videoBitrate('512k')
  // setup event handlers
  .on('end', function() {
    console.log('file has been converted succesfully');
  })
  .on('error', function(err) {
    console.log('an error happened: ' + err.message);
  })
  // save to file
  .save('/path/to/your_target.m4v');

```
## Progress

This example extends the input stream example by reporting progress.

```js
var fs = require('fs'),
  ffmpeg = require('../index');

// open input stream
var infs = fs.createReadStream(__dirname + '/test/assets/testvideo-43.avi');

infs.on('error', function(err) {
  console.log(err);
});

var proc = ffmpeg(infs)
  .preset('flashvideo')
  // setup event handlers
  .on('progress', function(info) {
    console.log('progress ' + info.percent + '%');
  })
  .on('end', function() {
    console.log('done processing input stream');
  })
  .on('error', function(err) {
    console.log('an error happened: ' + err.message);
  })
  .save('/path/to/your_target.flv');
```
## Stream

This example shows how to stream video conversion output directly to a writable stream.

```js
var ffmpeg = require('../index'),
  fs = require('fs');

// create the target stream (can be any WritableStream)
var stream = fs.createWriteStream('/path/to/yout_target.flv')

// make sure you set the correct path to your video file
var proc = ffmpeg('/path/to/your_movie.avi')
  // use the 'flashvideo' preset (located in /lib/presets/flashvideo.js)
  .preset('flashvideo')
  // setup event handlers
  .on('end', function() {
    console.log('file has been converted succesfully');
  })
  .on('error', function(err) {
    console.log('an error happened: ' + err.message);
  })
  // save to stream
  .pipe(stream, {end:true}); //end = true, close output stream after writing

```
## Thumbnails

This example demonstrates how to take screenshots (thumbnails) from a video at specific timemarks.

```js
var ffmpeg = require('../index');

var proc = ffmpeg('/path/to/your_movie.avi')
  // setup event handlers
  .on('filenames', function(filenames) {
    console.log('screenshots are ' + filenames.join(', '));
  })
  .on('end', function() {
    console.log('screenshots were saved');
  })
  .on('error', function(err) {
    console.log('an error happened: ' + err.message);
  })
  // take 2 screenshots at predefined timemarks and size
  .takeScreenshots({ count: 2, timemarks: [ '00:00:02.000', '6' ], size: '150x100' }, '/path/to/thumbnail/folder');

```
