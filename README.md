# SnapGen Video Automation

GitHub Actions and GitHub Pages dashboard for generating SnapGen videos from prompts.

Public dashboard: https://mmmmhub.github.io/snapgen-video-dashboard/

## Architecture

- `script.js` logs into SnapGen, submits a prompt, waits for the job, and writes `video-result.json`.
- `.github/workflows/generate.yml` runs the Playwright job on `repository_dispatch` with event type `create_video`.
- `site/index.html` is the public RTL dashboard. It dispatches the workflow, polls the run, downloads its artifact, and displays `video.mp4`.
- `app.py` is the optional local Gradio dashboard using the same workflow.

## GitHub Pages setup

1. Push the repository to GitHub with the `site/` folder and `.github/workflows/pages.yml`.
2. Open **Settings -> Pages** and select **GitHub Actions** as the source.
3. Run **Deploy GitHub Pages** once from the Actions tab, or push a change under `site/`.
4. The deployed page needs a GitHub access token from the person submitting a job. The page keeps that token in memory only and never saves it.

The token must have `Contents: write` and `Actions: read` access to this repository. For a public-facing service where visitors should not provide tokens, add a separate authenticated server; GitHub Pages alone cannot hide a shared token.

## GitHub setup

Add these repository secrets under **Settings -> Secrets and variables -> Actions**:

- `SNAPGEN_EMAIL`
- `SNAPGEN_PASSWORD`

The token used by the local dashboard must have access to this repository, `Contents: write` to dispatch events, and `Actions: read` to poll runs and download artifacts. Keep it in the `GITHUB_TOKEN` environment variable. Never commit it.

## Run the dashboard

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
export GITHUB_TOKEN="your-github-token"
export GITHUB_OWNER="mmmmhub"
export GITHUB_REPO="snapgen-video-automation"
python app.py
```

Open the local Gradio URL, enter a prompt, and click **Generate**. The UI waits for the Action to finish, downloads `video-result.json`, downloads the returned MP4, and displays it.

## Supported environment variables

- `SNAPGEN_PROVIDER`: `veo` by default; `grok` is also supported.
- `SNAPGEN_ASPECT`: `16:9` by default.
- `SNAPGEN_RESOLUTION`: `1080p` by default.
- `SNAPGEN_DURATION`: `8s` by default.
- `SNAPGEN_HEADLESS`: `true` by default.
- `SNAPGEN_CAPTCHA_TIMEOUT_SECONDS`: `180` by default; maximum wait for Turnstile to clear.
- `SNAPGEN_JOB_TIMEOUT_SECONDS`: `1200` by default.

## CAPTCHA behavior

The workflow does not attempt to bypass Cloudflare Turnstile or CAPTCHA. It waits for the challenge to clear. GitHub Actions has no human attached to its browser, so a challenge that does not clear automatically causes a clear failure instead of silently submitting a duplicate or bypassing site security.

On success, the workflow uploads `video-result.json`, `video-url.txt`, and the downloaded `video.mp4` as a run artifact. The Pages UI extracts the MP4 in the browser and provides a download button.

## Test dispatch

The dashboard performs this request automatically:

```json
{
  "event_type": "create_video",
  "client_payload": {
    "prompt": "A cinematic shot of a futuristic city, highly detailed, 8k resolution",
    "request_id": "unique-request-id"
  }
}
```
