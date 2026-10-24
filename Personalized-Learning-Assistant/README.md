# Personalized Learning Assistant

An AI-powered learning assistant built with **n8n**, **Google Gemini**,
and **Google Sheets**. It lets students ask questions through a chat
interface and can personalize explanations using saved learning
preferences and previous questions.

## Project objectives

-   Provide a chat-based interface for students.
-   Generate clear, beginner-friendly explanations with Google Gemini.
-   Use conversation memory for context within a chat session.
-   Store student preferences and question history in Google Sheets for
    use across new chats.
-   Demonstrate an adaptive agent that personalizes responses using
    stored context.

## Technology stack

-   **n8n Cloud** --- workflow automation and chat orchestration
-   **n8n Chat Trigger** --- receives student messages and exposes the
    chat interface
-   **AI Agent** --- manages the assistant's response
-   **Google Gemini Chat Model** --- generates explanations
-   **Simple Memory** --- keeps short-term context for a chat session
-   **Google Sheets** --- stores persistent student profiles and
    question history

## Repository structure

``` text
Personalized-Learning-Assistant/
├── README.md
├── workflow/
│   └── learning-assistant.json
├── docs/
│   ├── setup-guide.md
│   └── experiment-report.md
└── .gitignore
```

The workflow JSON is an export from n8n. Export and add it to
`workflow/learning-assistant.json` after checking that it contains no
secrets or private student data.

## How it works

1.  A student submits a question in the chat interface.
2.  The n8n workflow identifies the student using a stable student ID.
3.  Google Sheets is queried for the student's saved preferences and
    relevant question history.
4.  The AI Agent uses the current question, retrieved profile, and
    available conversation memory to personalize its answer.
5.  The response is returned to the chat interface.
6.  The workflow appends the question to `QuestionHistory` and updates
    the student profile when appropriate.

> **Important:** Saving a question does not automatically make it
> available to the AI. The workflow must retrieve relevant history from
> Google Sheets and pass it into the AI Agent as context.

## Google Sheets data model

Create a spreadsheet named `Student Learning Memory` with these tabs.

### `Students`

  -----------------------------------------------------------------------
  student_id        preferred_style   skill_level       topics_learned
  ----------------- ----------------- ----------------- -----------------
  STU001            Simple            Beginner          Python basics
                    explanations with                   
                    examples                            

  -----------------------------------------------------------------------

Keep one profile row per student. Use a stable ID rather than a name
alone.

### `QuestionHistory`

  student_id   timestamp       question               topic
  ------------ --------------- ---------------------- --------
  STU001       ISO timestamp   Explain Python loops   Python

Add one row per question. Avoid storing unnecessary personal or
sensitive information.

## Setup

Follow [`docs/setup-guide.md`](docs/setup-guide.md) for detailed
configuration steps.

## Experiment report

See [`docs/experiment-report.md`](docs/experiment-report.md) for the
objective, design, implementation, testing procedure, and conclusion for
Experiment 6.

## Security notes

-   Never commit API keys, access tokens, OAuth secrets, or credentials.
-   Use n8n credentials to store provider authentication.
-   Inspect exported workflow JSON before committing it.
-   Use fictional or anonymized student data for demonstrations.
-   Share the chat URL only with the intended audience and configure
    access controls where available.

## Limitations

-   Gemini can temporarily return service errors such as HTTP 503.
-   Google Sheets is suitable for a small demonstration, but larger
    deployments may need a database.
-   Cross-chat personalization depends on correctly identifying the
    student and retrieving saved rows.
-   The hosted chat URL and n8n credential setup may vary with account
    and n8n version.

## License

Add a license here if you intend to distribute or reuse this project
publicly.
