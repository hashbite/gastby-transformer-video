# gatsby-transformer-video

> Convert videos via FFMPEG. Easily convert & host **small** videos on your own.

[![NPM Version Badge](https://badgen.net/npm/v/gatsby-transformer-video)](https://www.npmjs.com/package/gatsby-transformer-video)
[![NPM Monthly Downloads Badge](https://badgen.net/npm/dm/gatsby-transformer-video)](https://www.npmjs.com/package/gatsby-transformer-video)
[![Github Check Badge](https://badgen.net/github/checks/hashbite/gatsby-transformer-video/main)](https://github.com/hashbite/gatsby-transformer-video/actions)
[![License Badge](https://badgen.net/npm/license/gatsby-transformer-video)](https://github.com/hashbite/gatsby-transformer-video/blob/main/LICENSE)

**This is a beta plugin and supports Gatsby v2 and v3. The videos will work fine on your website, but the implementation code still needs some love.**

:warning::warning::warning: Converting videos might take a lot of time. Make sure to have an effective caching mechanism in place. See [caching](#caching)

## Features

- Source: Works with nodes from `gatsby-source-filesystem` and `gatsby-source-contentful`
- Defaults optimized for small files with decent quality and quick and seamless streaming
- Supported codecs: h264, h265, VP9, WebP & gif
- Several parameters to tweak the output: maxWidth/maxHeight, overlay, saturation, duration, fps, ...
- Create video conversion profiles. Create a converter function using `fluent-ffmpeg` to unlock all FFMPEG features.
- Take screenshots at any position of the video

## Usage example

https://github.com/hashbite/gatsby-transformer-video/tree/packages/example

## Installation

```sh
npm i gatsby-transformer-video
```

## Prerequisites

To properly convert and analyze videos, this relies on `ffmpeg` and `ffprobe`. If these are not available for your node process, they will be **downloaded automatically**.

If you want to use your own version of `ffmpeg` and `ffprobe`, it should be compiled with at least the following flags enabled:

`--enable-gpl --enable-nonfree --enable-libx264 --enable-libx265 --enable-libvpx --enable-libwebp --enable-libopus`

Some operating systems may provide these FFMPEG via a package manager. Some may **not include** `ffprobe` when installing `ffmpeg`.

If you can't find a way to get ffmpeg with all requirements for your system, you can always compile it on your own: https://trac.ffmpeg.org/wiki/CompilationGuide

**How to link your custom binaries to this plugin**

- Provide the path to ffmpeg && ffprobe via GatsbyJS plugin configuration
- Or set an environment variable -> https://github.com/fluent-ffmpeg/node-fluent-ffmpeg#ffmpeg-and-ffprobe
- Or add the static binaries to any folder in your `$PATH`

### Linux

Linux/Debian users might be able to: `sudo apt-get install ffmpeg`

Make sure you have `ffprobe` installed as well.

### OSX

If you got [brew](https://brew.sh/) installed, you can get FFMPEG including FFPROBE via:

```sh
brew tap homebrew-ffmpeg/ffmpeg
brew install homebrew-ffmpeg/ffmpeg/ffmpeg --with-fdk-aac --with-webp
```

<sub>(Source: https://trac.ffmpeg.org/wiki/CompilationGuide/macOS#ffmpegthroughHomebrew)</sub>

## Usage

### config @ `gatsby-config.js`

The plugin works fine in most cases without any configuration options set.

Here is a full list of available, optional, options:

```js
const { resolve } = require(`path`)
const { platform } = require(`os`)
module.exports = {
  plugins: [
    {
      resolve: `gatsby-transformer-video`,
      options: {
        /**
         * Set if FFMPEG & FFPROBE should be downloaded if they are not found locally.
         *
         * Downloaded binaries are stored in `node_modules/.cache/gatsby-transformer-video-bins/`
         *
         * Default: true
         */
        downloadBinaries: false,
        /**
         * Optional: Pass your own binaries
         *
         * Assumes you store your binaries in the following pattern:
         * ./bin/darwin/ffmpeg
         * ./bin/darwin/ffprobe
         * ./bin/linux/ffmpeg
         * ./bin/linux/ffprobe
         * ...
         *
         * Default: null
         */
        ffmpegPath: resolve(__dirname, 'bin', platform(), 'ffmpeg'),
        ffprobePath: resolve(__dirname, 'bin', platform(), 'ffprobe'),

        // Optional profiles for full fluent-ffmpeg access
        profiles: {
          sepia: {
            extension: `mp4`,
            converter: function({ ffmpegSession, videoStreamMetadata }) {
              // Example:
              // https://github.com/hashbite/gatsby-transformer-video/blob/packages/example/gatsby-config.js
            },
          },
        },
      },
    },
  ],
}
```

### GraphQL Query

```graphql
query {
  allFile {
    edges {
      node {
        id
        videoGif(duration: 2, fps: 3, maxWidth: 300) {
          path
        }
        videoH264(
          overlay: "gatsby.png"
          overlayX: "end"
          overlayY: "start"
          overlayPadding: 25
          screenshots: "0,50%" (See: https://github.com/fluent-ffmpeg/node-fluent-ffmpeg#screenshotsoptions-dirname-generate-thumbnails)
        ) {
          path #String
          absolutePath #String
          name #String
          ext #String
          codec #String
          formatName #String
          formatLongName #String
          startTime #Float
          duration #Float
          size #Int
          bitRate #Int
          screenshots #String
        }
        videoProfile(profile: "yourProfileName") {
          path
        }
      }
    }
  }
  allContentfulAsset {
    edges {
      node {
        # Same fields available as above
      }
    }
  }
}
```

### Rendering

- Pain implementation: https://github.com/hashbite/gatsby-transformer-video/blob/main/packages/example/src/pages/index.js
- We plan to prove a component for this: https://github.com/hashbite/gatsby-transformer-video/issues/8

## Caching

Generating videos take time. A lot. You should cache your results. Otherwise, you might not even be able to publish on your hosting platform.

This plugin stores all converted files in `node_modules/.cache/gatsby-transformer-video/`

[We plan to improve the caching behavior](https://github.com/hashbite/gatsby-transformer-video/issues/6)

Here is a list of plugins that can help you:

- Netlify: https://www.gatsbyjs.org/packages/gatsby-plugin-netlify-cache/
- Via SFTP: https://www.gatsbyjs.org/packages/gatsby-plugin-sftp-cache/
- Gatsby Cloud should work well to store the generated files
- We could write a caching plugin using a node p2p network like https://dat.foundation/ to eventually eliminate the need of a server for caching generated files
