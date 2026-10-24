# Setup Guide --- Personalized Learning Assistant

This guide explains how to configure the n8n workflow, Google Gemini,
the chat interface, and Google Sheets persistence.

## Prerequisites

-   An n8n Cloud account
-   A Google account
-   Access to Google AI Studio and a Gemini API key
-   A Google Sheets spreadsheet
-   Basic familiarity with n8n nodes and workflow connections

## 1. Create the Google Sheets database

Create a spreadsheet named **Student Learning Memory**.

### Tab: `Students`

Add these headers in row 1:

``` text
student_id | preferred_style | skill_level | topics_learned
```

Example test row:

``` text
STU001 | Simple explanations with examples | Beginner | Python basics
```

Use one row per student. The `student_id` must remain stable across
chats.

### Tab: `QuestionHistory`

Add these headers in row 1:

``` text
student_id | timestamp | question | topic
```

Each question should be appended as a new row.

## 2. Configure Google Gemini in n8n

1.  Open your n8n workflow.
2.  Open the **Google Gemini Chat Model** node connected to your AI
    Agent.
3.  Select or create the appropriate Google Gemini credential.
4.  Store the API key in n8n credentials; do not hard-code it in prompts
    or files.
5.  Choose a model available to your account.
6.  Test the node.

If Gemini returns HTTP 503, the selected model may be temporarily
overloaded. Retry later or choose another currently available model. A
workflow retry setting can help with temporary errors, but does not
resolve a prolonged outage.

## 3. Configure the chat workflow

A typical workflow includes:

``` text
Chat Trigger
    ↓
Read student profile from Google Sheets
    ↓
Retrieve relevant question history (optional at first)
    ↓
Combine chat input and retrieved context
    ↓
AI Agent ← Google Gemini Chat Model
    ↑
Simple Memory (short-term conversation context)
    ↓
Save question to Google Sheets
    ↓
Return the AI response to the chat
```

The exact node order depends on the Chat Trigger response mode and your
workflow design. Ensure that the final response returned to the student
is the AI Agent's answer, not the output of a logging node.

### Chat Trigger

1.  Open the Chat Trigger node.
2.  Configure the chat mode and response behavior supported by your n8n
    version.
3.  For production use, activate or publish the workflow.
4.  Use the hosted chat URL or the exact production URL supplied by n8n.
5.  Do not manually construct a webhook URL from a webhook ID.
6.  Use the test URL only while testing in the mode that n8n specifies.

### Simple Memory

Keep the existing Simple Memory connection for context within the
current chat session. Configure the session key using the session
identifier supplied by the Chat Trigger and verify it in execution data.
Simple Memory alone should not be treated as durable storage across all
new sessions or restarts.

## 4. Connect Google Sheets

Add a Google Sheets credential using the Google account that can access
the spreadsheet.

### Read the student profile

1.  Add a Google Sheets node before the AI Agent.
2.  Choose the spreadsheet `Student Learning Memory`.
3.  Select the `Students` tab.
4.  Use the row-reading operation supported by your n8n version.
5.  Filter `student_id` by the current student's stable ID.
6.  Test the node and inspect its output.

For the initial test, you may use the fixed ID `STU001`. Replace this
with a reliable student identifier before allowing multiple students to
use the assistant. Do not rely on a fixed ID for all users.

### Pass profile data to the AI Agent

The AI Agent must receive both the student's current message and the
saved profile. If a Google Sheets node replaces the incoming item, use a
Merge node or another supported method to preserve both pieces of data.
Test the merged output before connecting it to the Agent.

Add instructions similar to the following to the Agent's system message:

> You are a personalized learning assistant for college students. Answer
> the current question accurately using clear, beginner-friendly
> language. If a saved student profile is provided, adapt the
> explanation to the student's preferred learning style and skill level.
> Use retrieved question history to connect related topics when useful.
> Do not invent preferences or claim to remember data that is not
> present in the supplied profile, history, or current conversation. If
> profile information is missing, ask a concise follow-up when
> appropriate.

### Save each question

Add a Google Sheets node to append a row to `QuestionHistory`. Map:

  -----------------------------------------------------------------------
  Column                              Value
  ----------------------------------- -----------------------------------
  `student_id`                        Current student's ID

  `timestamp`                         Current timestamp,
                                      e.g. `{{ $now.toISO() }}`

  `question`                          Original message from the Chat
                                      Trigger

  `topic`                             Topic identified from the question,
                                      or a simple initial category
  -----------------------------------------------------------------------

Select the original message field from the Chat Trigger's execution
data. The field name may differ by workflow. For example, if the trigger
node is named `When chat message received` and its message field is
`chatInput`, an expression may be:

``` javascript
{{ $('When chat message received').item.json.chatInput }}
```

Use the exact node name and field confirmed in your execution data.

### Retrieve previous questions

To personalize responses across new chats, add a read operation for
`QuestionHistory` filtered by the current student's ID. Pass a limited,
relevant set of rows to the Agent as context. Saving history without
retrieving and passing it to the Agent will not make it recallable.

For a small demonstration, retrieving a few recent questions is
sufficient. For larger histories, add limits or relevance filtering.

### Update student preferences

When a student explicitly shares or changes a preference, update their
profile in `Students` using an append-or-update operation keyed by
`student_id`, if supported by your node version. Preserve existing
values when the new update does not provide a replacement. Test profile
updates separately to avoid overwriting useful data with blanks.

## 5. Test the complete workflow

1.  Add a test profile for `STU001`.
2.  Open the hosted chat URL and send: "I prefer simple explanations
    with examples."
3.  Confirm the profile is stored or updated in `Students`.
4.  Ask: "Explain Python loops."
5.  Confirm the question is appended to `QuestionHistory`.
6.  Start a fresh chat session using the same student ID.
7.  Ask a related question.
8.  Verify that the workflow retrieves the profile and relevant history
    and passes them into the Agent.
9.  Inspect the execution details if the response is missing or not
    personalized.

## 6. View execution logs

1.  Open the workflow in n8n.
2.  Open **Executions**.
3.  Send a new message through the hosted chat.
4.  Refresh the execution list and open the latest run.
5.  Inspect node inputs, outputs, and any errors.

If successful executions do not appear, check the workflow's
execution-saving settings. Labels vary by n8n version. These settings
generally affect future executions, not runs that were never saved.

## 7. Export and add the workflow to Git

1.  Export or download the workflow JSON from n8n.
2.  Inspect it for credentials, API keys, private student data, and
    other secrets.
3.  Remove any secrets before committing.
4.  Save it as `workflow/learning-assistant.json` in this repository.
5.  Commit the JSON and documentation to your existing Git repository.

The exported workflow may need credentials and environment-specific
settings to be selected again when imported. Do not commit credential
secrets.

## Troubleshooting

### `404 — webhook is not registered`

-   Confirm that the workflow is active or published.
-   Use the exact production or hosted chat URL supplied by n8n.
-   Do not append `/chat` or manually construct a webhook URL unless n8n
    explicitly provides it.

### Gemini returns `503 Service unavailable`

-   Wait briefly and retry.
-   Verify the model is available to your account.
-   Enable retry-on-fail for temporary errors if appropriate.
-   Check provider status and account limits if the issue persists.

### The assistant forgets preferences in a new chat

-   Confirm that the new chat uses the same student ID.
-   Check that the `Students` lookup returns a row.
-   Confirm the profile data reaches the AI Agent.
-   Confirm that history is retrieved and passed to the Agent if
    previous questions should be recalled.
-   Do not rely on Simple Memory alone for persistent student profiles.

### Google Sheets returns no rows

-   Verify the spreadsheet and tab names.
-   Check the Google credential has access.
-   Confirm header spelling and the student ID match exactly.
-   Inspect the Google Sheets node output.

### Logs are not visible

-   Open the workflow's Executions page.
-   Check execution-saving settings.
-   Trigger a new run after changing settings.
