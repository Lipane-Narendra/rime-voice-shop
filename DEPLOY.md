# How to Deploy (make this a real public website)

Right now, running `npm start` only works on YOUR computer (localhost).
For submission, judges need a public URL they can open on their own device.
The fastest free option is **Render.com**.

## Steps (Render — free, ~10 minutes)

1. Push this project to a **public GitHub repo** first (see README setup steps).
2. Go to https://render.com and sign up (free, no card needed for free tier).
3. Click **New +** → **Web Service**.
4. Connect your GitHub account and select this repo.
5. Fill in:
   - **Name:** anything, e.g. `voice-kirana-assistant`
   - **Runtime:** Node
   - **Build Command:** `npm install`
   - **Start Command:** `npm start`
   - **Instance type:** Free
6. Under **Environment Variables**, add:
   - Key: `RIME_API_KEY`
   - Value: your real Rime API key (NEVER put this in GitHub — only here, in Render's dashboard)
7. Click **Create Web Service**. Wait 2-5 minutes for it to build and deploy.
8. Render gives you a public URL like `https://voice-kirana-assistant.onrender.com`
   — THIS is the link you submit as your "public artifact URL".

## Notes

- Free Render services "sleep" after inactivity and take ~30-50 seconds to
  wake up on the first request. If your demo video shows a slow first load,
  mention this in your README so judges know it's a free-tier limitation,
  not your app being slow.
- Alternative free options if Render doesn't work for you: Railway.app,
  Cyclic.sh, or Vercel (Vercel needs slight code changes for serverless —
  Render is simpler for this Express app).
- Test the deployed URL yourself in Chrome before submitting, exactly like
  you tested localhost, to make sure the mic + Rime call both still work
  in production.
