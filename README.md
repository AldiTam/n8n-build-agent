# Building an AI Agent with n8n

This guide will walk you through creating a powerful AI agent using n8n, a workflow automation tool. This agent can automate tasks like sending payment reminders for overdue invoices.

## Part 1: Setting up n8n for Free

We will use Docker to run n8n locally for free.

1.  **Install Docker Desktop:** Download and install Docker Desktop for your operating system.
2.  **Create a Docker Volume:** Open your terminal or command prompt and run the following command to create a persistent volume for your n8n data:
    ```bash
    docker volume create n8n_data
    ```
3.  **Run n8n:** Execute the following command to start the n8n container:
    ```bash
    docker run -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
    ```
4.  **Access n8n:** Open your web browser and navigate to `http://localhost:5678`.
5.  **Create Your Account:** Follow the on-screen instructions to create your n8n account.

## Part 2: Building the AI Agent

Now, let's build the core of our AI agent.

1.  **Create a New Workflow:** From the n8n dashboard, create a new workflow.
2.  **Configure the Agent:** An AI agent in n8n consists of three main components: a Chat Model, Memory, and Tools.

    *   **Chat Model (LLM):** This is the reasoning power of your agent. We'll use Google's Gemini for this tutorial as it offers a generous free tier.
        1.  Go to `aistudio.google.com` to get a free API key.
        2.  In your n8n workflow, add the Gemini node and create a new credential using your API key.
        3.  Select a suitable Gemini model (e.g., `Gemini 2.5 flashlight preview 617`).

    *   **Memory:** Add a "Simple Memory" node to your workflow. This allows the agent to remember the context of the conversation.

    *   **Tools:** These are the actions your agent can perform.
        1.  **QuickBooks Online:** Integrate with QuickBooks to fetch data like overdue invoices. You'll need to set up your credentials.
        2.  **Send Email:** Add an email node (e.g., "Send Email" with SMTP) to enable the agent to send reminders. You'll need to configure it with your email provider's settings (e.g., for Gmail, use `smtp.gmail.com` and an "App Password").

3.  **Prompt Engineering:** A well-crafted prompt is crucial for the agent's performance.
    *   Structure your prompt with clear sections: `Role`, `Task`, `Tools`, and `Output`.
    *   Provide detailed context and instructions. For example, specify the tone of the email reminders based on how overdue the invoice is.
    *   To provide the current date to the agent, you can use the following JavaScript snippet in your prompt: `{{new Date().toISOString().split('T')[0]}}`

## Part 3: Adding Human-in-the-Loop Approval

To ensure you have control over the agent's actions, let's add an approval step before any emails are sent.

1.  **Structured Output:** In your agent's configuration, enable the "Require Specific Output Format" option and provide a JSON structure for the data you want the agent to return (e.g., `customer_name`, `email_subject`, `email_body`).
2.  **Modify the Agent's Task:** Update the agent's prompt to instruct it to *write* the email content but *not* send it. Remove the "Send Email" tool from the agent's initial set of tools.
3.  **Scheduled Trigger:** Change the workflow trigger from a manual chat to a schedule (e.g., weekly) for automated execution.
4.  **Process Emails Individually:**
    *   Use a "Split Out" node to separate the list of generated emails into individual items.
    *   Use a "Loop Over Items" node to process each email one at a time.
5.  **Discord for Approvals:**
    *   Integrate with Discord to send approval notifications. You'll need a Discord bot token.
    *   Create a message in a designated channel that includes the email details and "Approve" / "Disapprove" buttons.
6.  **Conditional Logic:**
    *   Use an "If" node to check the result of the Discord approval.
    *   If the email is approved, the workflow will proceed to the "Send Email" node.
7.  **Finalize the Workflow:** Connect all the nodes in the correct sequence to complete your automated, human-supervised AI agent.
