# L10, Exercise A -- Webhook → Groq (LLM) → "Joke on a Given Topic"

## Goal

Create a workflow in **n8n** that receives a joke **topic** via
**Webhook**, calls **Groq (OpenAI-compatible Chat Completions)**, and
returns a short, clean joke in Polish as JSON.

------------------------------------------------------------------------
## Step 0.1: Obtain GROQ API KEY

1. Go to Groq.com
2. Click on "Start building"
3. Log in with your Google account
4. Click on "API Keys"
5. Click "Create new key"
6. Enter a name for the key (can be any)
7. Click Submit
8. Copy the generated key and store it in a safe place as it will not be dispalyed anymore.

## Step 0.2: Configure GROQ_API_KEY in n8n

Before you start, you must store your **Groq API key** as an environment
variable in Codespaces, so it can be used safely in n8n.

1.  In your Codespace, open the terminal

2.  Create a file named **`.env`** in the root of your repository (if it
    does not exist).

3.  Add the following line (replace `YOUR_KEY_HERE` with your actual
    Groq key):

        GROQ_API_KEY=YOUR_KEY_HERE

4.  Restart your Codespace so that n8n loads the new environment
    variable.

5.  In n8n, when configuring the HTTP Request node, use:

        Authorization: Bearer {{$env.GROQ_API_KEY}}

    This way, the key never appears in your workflow JSON export or
    screenshots.

------------------------------------------------------------------------

## Step 1: Create a New Workflow

1.  Start **n8n** in your fork Codespace.
2.  Go to **Workflows → + New**.
3.  Name it: `L10_ExerciseA_<your_github_username>`.

**NOTE:** In `L10/solutions/L10_ExerciseA.md` provide a screenshot showing
a newly created, properly named workflow. The browser address bar must
be visible.

------------------------------------------------------------------------

## Step 2: Add a Webhook Node

1.  Drag a **Webhook** node onto the canvas.
2.  Configure:
    -   **HTTP Method**: `GET`
    -   **Path**: `l10/joke`
    -   **Response Mode**: `Using 'Respond to Webhook' node`
3.  Save the workflow.

You now have two URLs: - Test:
`http://[YOUR_CODESPACE_URL]/webhook-test/l10/joke` - Production:
`http://[YOUR_CODESPACE_URL]/webhook/l10/joke`

**Expected query/body params** (any of these are optional except
`topic`): - `topic` -- the joke topic (e.g. "programmers", "bike", "exam
session")
- `style` -- e.g. `one-liner`, `dad-joke` *(default: "one-liner")*
- `temperature` -- `0–2` *(default: `0.7`)*

------------------------------------------------------------------------

## Step 3: Add an HTTP Request Node (Groq API)

1.  Click **+** to add a node after Webhook.
2.  Choose **HTTP Request**.
3.  Configure:
    -   **HTTP Method**: `POST`
    -   **URL**: `https://api.groq.com/openai/v1/chat/completions`
    -   **Authentication**: `None`
    -   **Headers**:
        -   `Content-Type: application/json`
        -   `Authorization: Bearer {{$env.GROQ_API_KEY}}`
    -   **Send Body**: `Yes`
    -   **Content Type**: `JSON`
    -   **Body Parameters (RAW JSON)**:
        In the **RAW JSON** body, use values from the Webhook:
        -   `model`: `openai/gpt-oss-20b`
        -   `stream`: `false`
        -   `temperature`: `{{$json.query.temperature || 0.7}}`
        -   `messages`:
            -   **System**: "You respond in English, concisely and
                without inappropriate language."
            -   **User**: "Tell a short, clean joke on the topic:
                `<topic taken from the previous step>`

**TIP:** Ensure that in **n8n** you have the environment variable
`GROQ_API_KEY` configured. Never paste the key directly into the node.

**Options (recommended):** - **Retry On Fail**: `On`
- **Max Retries**: `2`
- **Timeout**: `10000–15000 ms`

------------------------------------------------------------------------

## Step 4: Extract and Respond

1.  Add a **Set** node (after HTTP Request) to map the Groq response
    into simple JSON:
    -   **Keep Only Set**: `true`
    -   Fields to set:
        -   `topic` → `{{$json.query.topic}}`
        -   `style` → `{{$json.query.style || "one-liner"}}`
        -   `temperature` → `{{$json.query.temperature || 0.7}}`
        -   `joke` → `{{$json.choices[0].message.content}}`
2.  Add **Respond to Webhook**:
    -   **Response Code**: `200`
    -   **Respond With**: `All incoming items`

**NOTE:** Provide screenshots for the HTTP Request and Respond nodes.
The browser address bar must be visible.

------------------------------------------------------------------------

## Step 5: Test the Workflow

1.  Click **Execute Workflow** to enable the test endpoint.
2.  Test in a browser (GET):

-   Minimal (topic only):
    `http://[YOUR_CODESPACE_URL]/webhook-test/l10/joke?topic=programmers`

-   With parameters:
    `http://[YOUR_CODESPACE_URL]/webhook-test/l10/joke?topic=bike&style=dad-joke&temperature=0.6`

3.  Or test with `curl` (POST, JSON body):

``` bash
curl -X POST "http://[YOUR_CODESPACE_URL]/webhook-test/l10/joke"   -H "Content-Type: application/json"   -d '{"topic":"exam session","style":"one-liner","temperature":0.5}'
```

**NOTE:** Provide a screenshot with the successful execution result. The
address bar must be visible.

------------------------------------------------------------------------

## Note: Example Output

Example JSON should look like:

``` json
{
  "topic": "programmers",
  "style": "one-liner",
  "temperature": 0.7,
  "joke": "Why did the programmer bring a ladder to work? Because he heard the project had high levels of abstraction."
}
```

------------------------------------------------------------------------

## Step 6: Activate Workflow

1.  Toggle **Active** in the top-right corner.
2.  Use the **production** URL for permanent access:
    `http://[YOUR_CODESPACE_URL]/webhook/l10/joke?topic=AI&style=one-liner`

------------------------------------------------------------------------

## Submission checklist (what to include)

-   Screenshot of the named workflow.
-   Screenshot of **HTTP Request** configuration (headers + body).
-   Screenshot of **Respond to Webhook** configuration.
-   Screenshot of a successful test call (test URL visible).
-   Paste example JSON output to `L10/solutions/L10_ExerciseA.json`.
