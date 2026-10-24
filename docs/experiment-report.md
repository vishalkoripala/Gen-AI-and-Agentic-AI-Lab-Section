# Experiment 6 --- Implement an Adaptive Agent with Memory

**Case study:** Personalized Learning Assistant\
**Course outcome:** CO3

## 1. Aim

To implement an adaptive AI agent that stores and retrieves conversation
context and uses student learning preferences and previous questions to
provide personalized explanations.

## 2. Problem statement

A general-purpose assistant may give the same style of explanation to
every student and may not retain useful context across separate chat
sessions. This experiment builds a learning assistant that can use
stored student preferences and question history to adapt its responses.

## 3. Objectives

-   Build a chat-based learning assistant using n8n.
-   Integrate Google Gemini as the language model.
-   Use conversation memory for context within a chat session.
-   Store student preferences and previous questions in Google Sheets.
-   Retrieve stored information across new chat sessions using a stable
    student identifier.
-   Test whether the assistant's responses reflect saved learning
    preferences.

## 4. Tools and technologies

  -----------------------------------------------------------------------
  Tool                                Purpose
  ----------------------------------- -----------------------------------
  n8n Cloud                           Workflow automation and
                                      orchestration

  Chat Trigger                        Receives student messages through a
                                      chat interface

  AI Agent                            Coordinates context and response
                                      generation

  Google Gemini Chat Model            Generates explanations

  Simple Memory                       Maintains short-term conversation
                                      context

  Google Sheets                       Stores persistent student profiles
                                      and question history

  Git and GitHub                      Version control and project
                                      documentation
  -----------------------------------------------------------------------

## 5. System design

The system consists of the following stages:

1.  **Input:** A student sends a question through the n8n chat
    interface.
2.  **Identification:** The workflow obtains a stable student ID.
3.  **Profile retrieval:** Google Sheets is queried for the student's
    preferences and skill level.
4.  **History retrieval:** Previous questions are retrieved for the same
    student when relevant.
5.  **Context preparation:** The current question, saved profile,
    relevant history, and available session memory are supplied to the
    AI Agent.
6.  **Response generation:** Gemini generates a response suited to the
    student.
7.  **Persistence:** The question is appended to `QuestionHistory`, and
    explicit preference changes update the `Students` profile.
8.  **Output:** The response is returned to the chat interface.

### Architecture diagram

``` text
Student
  |
  v
n8n Chat Trigger
  |
  v
Identify Student
  |
  +--------------------+
  |                    |
  v                    v
Google Sheets       Google Sheets
Students profile    QuestionHistory
  |                    |
  +---------+----------+
            |
            v
      Prepare context
            |
            v
        AI Agent <----- Simple Memory
            ^
            |
     Gemini Chat Model
            |
            v
     Personalized answer
            |
            v
      Student chat UI

Current question -----------------> Append to QuestionHistory
Preference change ----------------> Update Students profile
```

## 6. Data model

### Students tab

  Field               Description
  ------------------- --------------------------------------
  `student_id`        Stable identifier for the student
  `preferred_style`   Preferred explanation style
  `skill_level`       Current or self-reported skill level
  `topics_learned`    Topics the student has studied

Example test record:

  -----------------------------------------------------------------------
  student_id        preferred_style   skill_level       topics_learned
  ----------------- ----------------- ----------------- -----------------
  STU001            Simple            Beginner          Python basics
                    explanations with                   
                    examples                            

  -----------------------------------------------------------------------

### QuestionHistory tab

  Field          Description
  -------------- ----------------------------------------------
  `student_id`   Identifier linking the question to a student
  `timestamp`    Time the question was recorded
  `question`     Original student question
  `topic`        Topic or category of the question

Each question is recorded as a separate row. The workflow must retrieve
relevant rows and pass them to the AI Agent to use them for cross-chat
personalization.

## 7. Implementation procedure

1.  Created an n8n workflow with a Chat Trigger and AI Agent.
2.  Connected a Google Gemini Chat Model to the AI Agent.
3.  Connected Simple Memory to preserve context within a chat session.
4.  Created a Google Sheets spreadsheet with `Students` and
    `QuestionHistory` tabs.
5.  Configured Google Sheets nodes to read the student profile and
    append question history.
6.  Configured the workflow to pass retrieved profile and history data
    to the AI Agent.
7.  Added instructions directing the Agent to use saved preferences
    without inventing missing information.
8.  Configured the Chat Trigger's hosted interface and activated or
    published the workflow.
9.  Tested the workflow in a browser and inspected executions and
    spreadsheet rows.
10. Exported the workflow JSON for version control after checking for
    secrets.

Only list steps as completed if they match the implementation actually
performed.

## 8. Test cases

Record the observed result for each test after running the workflow.

  -------------------------------------------------------------------------
  Test              Procedure         Expected result     Actual result
  ----------------- ----------------- ------------------- -----------------
  Chat response     Ask a basic       The assistant       To be recorded
                    question in the   returns a response  
                    chat UI                               

  Session memory    State a           The assistant uses  To be recorded
                    preference, then  the preference      
                    ask a follow-up                       
                    in the same                           
                    session                               

  Save profile      Store a           The `Students` row  To be recorded
                    preference for    contains the        
                    `STU001`          preference          

  Save question     Ask a new         A row is added to   To be recorded
                    question          `QuestionHistory`   

  Cross-chat        Start a new chat  The saved           To be recorded
  profile           with the same     preference is       
                    student ID        retrieved and used  

  Cross-chat        Ask about a       The assistant uses  To be recorded
  history           previous topic in retrieved history   
                    a new chat        if it was passed to 
                                      the Agent           

  Missing profile   Use a student ID  The assistant does  To be recorded
                    without a profile not invent          
                                      preferences         

  Error handling    Test during a     The failure is      To be recorded
                    temporary Gemini  visible and can be  
                    service error     retried             
  -------------------------------------------------------------------------

## 9. Results and observations

Complete this section with evidence from your own tests:

-   **Chat interface:** \[Record whether the hosted chat responded
    successfully.\]
-   **Session memory:** \[Record the result of the same-session
    preference test.\]
-   **Persistent profile:** \[Record whether Google Sheets saved and
    returned the profile.\]
-   **Question history:** \[Record whether questions were saved and
    retrieved.\]
-   **Cross-chat personalization:** \[Record the result from a fresh
    chat using the same student ID.\]
-   **Execution logs:** \[Record whether successful and failed runs were
    visible in n8n.\]

Suggested evidence includes screenshots of the workflow, chat responses,
Google Sheets test rows with sample data, and n8n execution details.
Remove private information and credentials from screenshots.

## 10. Limitations

-   Gemini responses may vary and may be temporarily unavailable.
-   Simple Memory is not a replacement for persistent storage.
-   Cross-chat retrieval requires a consistent student identifier and
    explicit workflow steps to fetch and pass stored context.
-   Google Sheets is appropriate for a small demonstration but may not
    suit a high-volume production application.
-   Incorrectly configured update operations may overwrite profile data;
    test them with sample records.
-   The system should not claim to remember details that it did not
    retrieve.

## 11. Future enhancements

-   Add a student sign-in mechanism and reliable identity management.
-   Retrieve only recent or semantically relevant questions.
-   Track learning progress and mastered topics.
-   Add feedback buttons so students can rate explanations.
-   Add a database such as PostgreSQL or Supabase for larger
    deployments.
-   Add monitoring, retries, and safer handling of provider errors.

## 12. Conclusion

This experiment demonstrates the design of an adaptive learning
assistant using n8n, Google Gemini, conversation memory, and Google
Sheets. Short-term memory supports context within a session, while
Google Sheets can preserve student preferences and question history
across sessions. Persistent personalization works when the workflow
identifies the student, retrieves the relevant saved data, and supplies
it to the AI Agent. The final report should be completed with actual
test results and screenshots from the running implementation.

## 13. References

-   n8n documentation: https://docs.n8n.io/
-   n8n Google Sheets integration:
    https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.googlesheets/
-   Google AI Studio: https://aistudio.google.com/
-   Google Sheets: https://sheets.google.com/
