# Smart Email Assistant and Spring AI Projects

This repository is a collection of practical Spring Boot, Spring AI, React, and
Chrome Extension projects. The projects demonstrate how to connect web
applications and browser extensions to hosted and local generative AI models.

The main end-to-end workflow is the **Smart Email Assistant**:

1. A user enters an email in the React application, or opens a Gmail compose
   window with the browser extension installed.
2. The frontend or extension sends the email content and an optional tone to
   the Spring Boot API.
3. The backend builds a prompt and sends it to Google Gemini.
4. The generated reply is returned to the client.
5. The React application displays the reply and can copy it to the clipboard;
   the Gmail extension inserts it directly into the compose box.

The other folders contain related AI examples for chat, research, audio
transcription, image generation, recipes, and local DeepSeek inference.

## Repository layout

| Project | Technology | Workflow |
| --- | --- | --- |
| [`email-writer/`](./email-writer/) | Spring Boot, Gemini, React, Chrome Extension | Generate a reply from email content and an optional tone |
| [`gemini-chat/`](./gemini-chat/) | Spring Boot, Gemini, React | Submit a question and display the model response |
| [`research-assistant/`](./research-assistant/) | Spring Boot, Gemini, Chrome Extension | Summarize selected browser text or suggest related reading |
| [`audio-transcribe/`](./audio-transcribe/) | Spring Boot, Spring AI, OpenAI Whisper, React | Upload an audio file and receive a transcription |
| [`audio-transcriber/`](./audio-transcriber/) | Spring Boot, Spring AI, OpenAI Whisper, React | Duplicate transcription implementation kept as a separate example |
| [`spring-ai-demo/`](./spring-ai-demo/) | Spring Boot, Spring AI, OpenAI, React | Chat, configurable chat options, image generation, and recipes |
| [`deepseek/`](./deepseek/) | Spring Boot, Spring AI, Ollama | Run chat and streaming responses with a local DeepSeek model |

The `*.zip` files contain packaged versions of selected examples. They are
not required when running the source directories.

## Architecture

The projects follow the same general request flow:

```text
React UI / Chrome Extension
          |
          | HTTP request to localhost:8080
          v
Spring Boot controller
          |
          | Prompt construction and provider request
          v
Gemini / OpenAI / Ollama
          |
          v
Generated text, image URL, or transcription
```

Most examples use port `8080`. Run only one backend at a time unless you
configure different ports.

## Prerequisites

- Java 17 or newer for most projects
- Java 23 for `email-writer/email-writer-sb` and `gemini-chat/gemini-chat`
- Maven 3.9+ (or the Maven Wrapper included in each backend)
- Node.js 18+ and npm for the Vite frontends
- An API key for the provider used by the selected project
- Ollama and the DeepSeek model for the local `deepseek` example
- Google Chrome or Chromium for the browser extensions

Check the Java and Node installations:

```powershell
java -version
node --version
npm --version
```

## Configuration and secrets

Do not commit API keys. The checked-in properties files reference environment
variables, so configure credentials in the shell before starting a backend.

### Email Writer

`email-writer/email-writer-sb/src/main/resources/application.properties`
expects:

```powershell
$env:GEMINI_URL = "https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key="
$env:GEMINI_KEY = "your-gemini-api-key"
```

The URL must end with `key=` because the backend appends `GEMINI_KEY` when it
calls Gemini.

### Gemini Chat

`gemini-chat/gemini-chat/src/main/resources/application.properties` expects:

```powershell
$env:GEMINI_API_URL = "https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key="
$env:GEMINI_API_KEY = "your-gemini-api-key"
```

### OpenAI examples

The audio transcription projects use:

```powershell
$env:API_KEY = "your-openai-api-key"
```

The Spring AI demo uses:

```powershell
$env:MY_APP_KEY = "your-openai-api-key"
```

### Research Assistant

The research backend uses the Gemini REST API and expects:

```powershell
$env:GEMINI_KEY = "your-gemini-api-key"
```

### DeepSeek with Ollama

Install Ollama, start it, and download the configured model:

```powershell
ollama serve
ollama pull deepseek-r1:1.5b
```

The default model is configured in
`deepseek/deepseek/src/main/resources/application.properties`.

## Smart Email Assistant workflow

### Option 1: React email writer

Start the backend:

```powershell
cd email-writer\email-writer-sb
.\mvnw.cmd spring-boot:run
```

In a second terminal, start the frontend:

```powershell
cd email-writer\email-writer-react
npm install
npm run dev
```

Open the Vite URL shown in the terminal, normally
`http://localhost:5173`. Enter the original email, optionally select
`Professional`, `Casual`, or `Friendly`, and select **Generate Reply**. The
frontend calls:

```text
POST http://localhost:8080/api/email/generate
Content-Type: application/json

{
  "emailContent": "Could you send the updated report?",
  "tone": "professional"
}
```

The backend returns the generated reply as text. The UI renders it in a
read-only field and provides a **Copy to Clipboard** action.

### Option 2: Gmail Chrome extension

The extension files are in [`email-writer/email-writer-ext/`](./email-writer/email-writer-ext/).

1. Start the email writer backend on port `8080`.
2. Open `chrome://extensions`.
3. Enable **Developer mode**.
4. Select **Load unpacked**.
5. Choose the `email-writer/email-writer-ext` directory.
6. Open Gmail and create or open a compose window.
7. Select **AI Reply** in the compose toolbar.

The content script watches for Gmail compose windows, extracts the visible
email content, calls the same `/api/email/generate` endpoint, and inserts the
response into the editable compose area. Gmail's DOM can change over time; if
the button is not visible, reload the extension and Gmail tab.

## Other project workflows

### Gemini Chat

Start the backend:

```powershell
cd gemini-chat\gemini-chat
.\mvnw.cmd spring-boot:run
```

Start the React client in another terminal:

```powershell
cd gemini-chat\gemini-chat-frontend
npm install
npm run dev
```

The client posts a question to `POST /api/qna/ask` and renders the response.

### Research Assistant

Start the backend:

```powershell
cd research-assistant\research-assistant
.\mvnw.cmd spring-boot:run
```

Load [`research-assistant/research-assistant-ext/`](./research-assistant/research-assistant-ext/)
as an unpacked Chrome extension. Select text on a page, open the extension,
and select **Summarize**. The extension sends:

```text
POST http://localhost:8080/api/research/process
Content-Type: application/json

{
  "content": "Selected browser text",
  "operation": "summarize"
}
```

The supported operations are `summarize` and `suggest`. Notes are saved in
Chrome local storage.

### Audio transcription

The `audio-transcribe` and `audio-transcriber` directories expose the same
workflow. Start either backend:

```powershell
cd audio-transcribe\audio-transcribe
.\mvnw.cmd spring-boot:run
```

Then start its matching frontend:

```powershell
cd audio-transcribe\audio-transcribe-frontend
npm install
npm run dev
```

Choose an audio file in the UI. The frontend sends it as multipart form data
to `POST /api/transcribe`; Spring AI uses OpenAI Whisper (`whisper-1`) and
returns the transcription.

### Spring AI demo

Start the backend:

```powershell
cd spring-ai-demo\SpringAiDemo
.\mvnw.cmd spring-boot:run
```

The backend provides these examples:

| Endpoint | Purpose |
| --- | --- |
| `GET /ask-ai?prompt=...` | Basic chat response |
| `GET /ask-ai-options?prompt=...` | Chat response with configured options |
| `GET /generate-image?prompt=...` | Generate image URL(s) |
| `GET /recipe-creator?ingredients=...&cuisine=...&dietaryRestriction=...` | Generate a recipe |

The React demo is in
[`spring-ai-demo/spring-ai-demo-react/`](./spring-ai-demo/spring-ai-demo-react/).
It uses Create React App, so start it with:

```powershell
cd spring-ai-demo\spring-ai-demo-react
npm install
npm start
```

### DeepSeek with Ollama

Start Ollama and pull the model as described in the configuration section.
Then run:

```powershell
cd deepseek\deepseek
.\mvnw.cmd spring-boot:run
```

The controller exposes `GET /ai/generate` for a complete response and
`GET /ai/generateStream` for a streaming response. This example runs locally
and does not require a hosted API key.

## Building and testing

Run backend tests from the relevant project directory:

```powershell
cd email-writer\email-writer-sb
.\mvnw.cmd test
```

Build a backend:

```powershell
.\mvnw.cmd clean package
```

Run the Vite frontend checks:

```powershell
cd email-writer\email-writer-react
npm install
npm run lint
npm run build
```

For the Create React App example, use `npm test` and `npm run build` from
`spring-ai-demo/spring-ai-demo-react`.

## Troubleshooting

- **Connection refused:** Start the matching Spring Boot backend before the
  frontend or extension. Confirm it is listening on port `8080`.
- **Provider authentication errors:** Check that the required environment
  variable is set in the same terminal that launches Maven.
- **CORS errors:** Use the `WebConfig` in each backend as the source of truth
  for permitted frontend origins and run the frontend on its expected local
  port.
- **Maven cannot resolve Spring AI milestones:** Ensure Maven can access
  `https://repo.spring.io/milestone` and retry the build.
- **Ollama errors:** Confirm `ollama serve` is running and that the configured
  model name exists locally.
- **Extension button missing:** Reload the extension and the Gmail or research
  tab, then open a new compose window or select text again.

## Security and repository hygiene

- Keep API keys outside source control.
- Never replace the placeholder environment variables with real credentials
  in `application.properties`.
- Do not upload generated `target`, `node_modules`, `dist`, or build output.
- Review provider terms and data handling requirements before sending private
  email, audio, or browser content to an external AI provider.

## License and attribution

This repository is intended for educational and demonstration purposes. Review
the licenses and terms of the external frameworks, model providers, APIs, and
browser platforms used by each example before redistributing or deploying
these projects.
