# 🕷️ BLACK WIDOW WhatsApp Bot v4

> A powerful multi-device WhatsApp bot built for automation, moderation,
> downloads, AI tools, and group management.

## Features

-   Multi-device support
-   QR & Pair Code login
-   Group moderation
-   Auto-replies
-   AI commands
-   Media downloader
-   Sticker tools
-   Admin utilities
-   Logging
-   Plugin-ready architecture

## Table of Contents

1.  Features
2.  Requirements
3.  Installation
4.  Configuration
5.  Running the Bot
6.  Deployment
7.  Commands
8.  Troubleshooting
9.  Contributing
10. License

## Requirements

-   Node.js 20+
-   Git
-   npm
-   Stable internet connection

## Installation

``` bash
git clone https://github.com/yourusername/BLACK-WIDOW.git
cd BLACK-WIDOW
npm install
```

## Configuration

Create a `.env` file:

``` env
SESSION_ID=
PREFIX=.
OWNER_NUMBER=2547XXXXXXXX
```

## Run

``` bash
npm start
```

Development:

``` bash
npm run dev
```

## Deployment

### Railway

Create a Railway project, connect your repository, configure environment
variables, and deploy.

### Render

Create a Web Service, set the start command to:

``` bash
npm start
```

### VPS

``` bash
npm install
npm start
```

## Commands

  Category   Examples
  ---------- ----------------------
  General    ping, menu, alive
  Group      kick, add, tagall
  Download   play, ytmp3, tiktok
  AI         ai, imagine
  Utility    sticker, qr, shorten

## Troubleshooting

-   Ensure Node.js version is supported.
-   Run `npm install` after pulling updates.
-   Verify environment variables are set correctly.
-   Check logs for runtime errors.

## Contributing

Fork the repository, create a feature branch, commit your changes, and
submit a pull request.

## Credits

Developed by **MASTERPEACE ELITE**.

## License

MIT License.
