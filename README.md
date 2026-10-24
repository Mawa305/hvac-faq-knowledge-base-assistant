# HVAC FAQ & Knowledge Base Assistant

A Telegram chatbot for an HVAC business that answers customer questions from the company's own FAQ document. Answers come only from a knowledge base stored in Pinecone, built from a Google Doc. If the answer is not in the document, the bot says so instead of guessing. Any failure is emailed to the business owner.

## Architecture

```
Part 1: Knowledge base ingestion (run manually)
Manual Trigger → Get a document (Google Docs) → Pinecone Vector Store (insert)
                                                    ↑ Default Data Loader
                                                    ↑ Embeddings OpenAI

Part 2: Customer chat (live)
Telegram Trigger → If (text exists?) → AI Agent (gpt-5-mini)
                                          ├── Pinecone Vector Store (retrieve as tool)
                                          ├── success → Send a text message (Telegram)
                                          └── error   → Edit Fields → Send a message (Gmail)
```

## Features

* Answers customer questions about services, pricing, service area, hours, warranty, and payment methods
* Uses only the knowledge base. The system prompt tells the agent to say it will connect the customer with the team if the answer is missing
* Retrieval runs as an agent tool, so the model searches the knowledge base only when it needs to
* Non-text Telegram messages are ignored by an If node
* Knowledge base is built from a Google Doc with one command. Update the doc and run the ingestion again to refresh it
* Errors are caught and emailed to the business owner with the error message and the original message data

## Tech Stack

* n8n 2.x (workflow automation)
* OpenAI `gpt-5-mini` (chat agent) and `text-embedding` model (embeddings)
* Pinecone (vector store)
* Google Docs (knowledge base source)
* Telegram Bot API (customer chat)
* Gmail API (error alerts)

## Setup

1. Create a Pinecone index named `hvac-faq-index`. Set the dimension to match the embedding model you use. The default OpenAI embedding model (`text-embedding-ada-002`) uses 1536.
2. Write your FAQ in a Google Doc. Include services, pricing ranges, service area, hours, warranty, and payment methods.
3. Import `hvac_faq_knowledge_base_assistant.json` into n8n.
4. Add credentials: OpenAI, Pinecone, Google Docs OAuth2, Telegram, and Gmail OAuth2. Assign them to the matching nodes.
5. Replace `YOUR_GOOGLE_DOC_URL` in the Get a document node with your FAQ doc link.
6. Replace `YOUR_EMAIL@example.com` in the Send a message node with the email that should receive error alerts.
7. Click **Execute workflow** once to load the FAQ into Pinecone. Check that the index has vectors.
8. Activate the workflow. Message your Telegram bot to test.

## Sample Questions

```
What areas do you cover?
How much does an AC tune-up cost?
Do you offer a warranty on repairs?
What payment methods do you accept?
Are you open on Sunday?
```

For a question that is not in the FAQ, the bot should say it will connect the customer with the team.

## Project Notes

* Built as part of my AI Automation portfolio.
* The agent only sees the knowledge base through the retrieval tool, so answers stay tied to the document.
* Only `gpt-5-mini` is used for the chat agent. Temperature and response format options are left at their defaults.
