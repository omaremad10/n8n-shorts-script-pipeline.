# n8n Shorts Script Pipeline

An [n8n](https://n8n.io) workflow that turns a Telegram message into a ready-to-record short-video package: send a topic to your bot and get back a researched script, a caption, hashtags, source links, and an MP3 voiceover.

Built for TikTok, YouTube Shorts and Instagram Reels scripts of roughly 30 to 45 seconds.

> **Status: untested.** The code steps were checked with sample data, but the full workflow has not yet been run end to end against the live Telegram, Tavily, Groq and Murf APIs. Expect to fix small things on first run, and please open an issue if you hit one.

## How it works

```
Telegram message ──> Extract topic ──> Tavily news search ──> Build prompt
                                                                   │
Send voiceover <── Murf text-to-speech <── Send script <── Parse script <── Groq (Llama 3.3)
```

1. **Telegram Trigger** receives your message. The text is the topic; a leading `/command` is ignored.
2. **Tavily** searches news from the last week for that topic.
3. **Groq** (Llama 3.3 70B) writes a 90 to 130 word script from those results and returns it as JSON.
4. **Telegram** sends you the title, hook, script, caption, hashtags and source links.
5. **Murf** turns the script into an MP3, which is sent to the same chat.

The script is written only from the search results, and the sources are included so you can check claims before you publish.

## What you need

| Service | Used for | Key |
|---|---|---|
| n8n (self-hosted or cloud) | Running the workflow | |
| Telegram bot | Input and output | Bot token from [@BotFather](https://t.me/BotFather) |
| [Tavily](https://tavily.com) | Trend and news search | API key (`tvly-...`) |
| [Groq](https://console.groq.com) | Script writing | API key (`gsk_...`) |
| [Murf](https://murf.ai/api) | Voiceover | API key |

Each service has its own pricing and free-tier limits; check them before heavy use.

## Setup

1. In n8n, choose **Import from File** and select `workflow/shorts-script-pipeline.json`.
2. Create a **Telegram** credential with your bot token and select it on the three Telegram nodes.
3. Create three **Header Auth** credentials and select each on its HTTP Request node:

   | Node | Header name | Header value |
   |---|---|---|
   | Search trends (Tavily) | `Authorization` | `Bearer tvly-YOUR_KEY` |
   | Write script (Groq) | `Authorization` | `Bearer gsk_YOUR_KEY` |
   | Generate voiceover (Murf) | `api-key` | `YOUR_MURF_KEY` |

4. Activate the workflow and send your bot a topic, for example `electric cars in Egypt`.

The Telegram Trigger needs n8n to be reachable over HTTPS. n8n Cloud handles this; for a local install, use a tunnel.

## Customising

- **Voice:** change `voiceId` in the *Generate voiceover (Murf)* node. Murf's `GET /v1/speech/voices` endpoint lists the options.
- **Model:** change `model` in the *Write script (Groq)* node. Groq retires models from time to time; see their [model list](https://console.groq.com/docs/models).
- **Script length and style:** edit the system prompt in the *Build prompt* node.
- **Search window:** change `time_range` (`day`, `week`, `month`) in the *Search trends (Tavily)* node.

## Limitations

- No video generation yet; the output is a script and a voiceover.
- No error message is sent to Telegram if an API call fails; the run fails in n8n.
- Anyone who finds your bot can trigger the workflow and spend your API credits. Restrict it to your own chat ID before sharing the bot.

## Security

The workflow file contains no API keys. Keys live in n8n credentials and are not included when a workflow is exported. Check for secrets before committing your own exports anyway.

## License

MIT. See [LICENSE](LICENSE).
