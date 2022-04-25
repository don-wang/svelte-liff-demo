# A Demo LIFF App build with Svelte Kit

## Demo

You can try it by open this LIFF URL [https://liff.line.me/1656984897-84oW2nrm](https://liff.line.me/1656984897-84oW2nrm) via LINE.

LIFF QR

\*To use sendMessage function, it requires to open the liff in talk room:

![LIFF QR](./public/liff-qr.png)

The app itself is hosing in vercel: https://svelte-liff-demo.vercel.app/

## Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`),

### Set Environment Variables

Create .env file in root folder (use Environment variables for Vercel, AWS Amplify, etc), add variables as below:

```
VITE_LIFF_ID=16XXXXXXX-84xXxxXx
```

start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of your app:

```bash
npm run build
```
