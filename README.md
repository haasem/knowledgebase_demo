# N8N AI Service Agent - Knowledge Base Workflow - Technical Documentation

## Production Test: [Test the AI Service Agent](https://haasem.github.io/knowledgebase_demo/index.html)

<img width="1079" height="464" alt="image" src="https://github.com/user-attachments/assets/0f115d26-0899-45ae-9bdd-ab2d4e927b9e" />


## Overview
This N8N workflow implements an intelligent knowledge base system that combines Google Drive document storage, Supabase vector database for semantic search, and an AI-powered chat agent. The system automatically processes documents, maintains a searchable knowledge base, and provides conversational access to information through a sophisticated AI agent.
Architecture Components
Core Technologies

## N8N: Workflow automation and orchestration
Google Drive: Document storage and file management
Supabase: Vector database for document indexing and semantic search
OpenAI: Embeddings generation and language model
LangChain: AI agent framework and document processing

## Workflow Breakdown
### 1. Chat Interface (Query Processing)
Trigger: When chat message received

Type: Chat Trigger (Webhook)
Function: Receives incoming chat messages from the web interface
Configuration: Public webhook accepting chat input

AI Agent: Core intelligence component

Model: GPT-4.1-mini via OpenAI Chat Model
System Prompt: Configured as "Service Agent at Hilarious Machines"
Constraints:

Only uses knowledge base information
No fact invention
Provides email contacts when available
Friendly, empathetic communication
No jokes policy


Memory: Buffer window memory with 10-message context
Session Management: Uses sessionId + chatId for conversation tracking

Vector Store Retrieval: Supabase Vector Store1

Mode: Retrieve-as-tool
Function: Semantic search tool for the AI agent
Description: "Call this tool to retrieve information from the knowledge base according to the request of the Chat user"
Table: documents
Integration: Connected to OpenAI embeddings (1536 dimensions)

### 2. Document Ingestion Pipeline
New Document Trigger: New Doc Trigger

Type: Google Drive Trigger
Watch Folder: KnowledgeBase (1Cq2li17E57yVQ8oF7b3Z0JXDcukGdV5P)
Event: fileCreated
Polling: Every minute
Function: Automatically detects new documents added to the knowledge base folder

Document Processing Chain:

Google Drive: Downloads the detected file

Operation: Download
File ID: Dynamic from trigger ({{ $json.id }})
Output: Binary file data


Default Data Loader: Processes document content

Input: Binary data from Google Drive
Metadata: Attaches file_id for tracking
Function: Converts various document formats to processable text


Embeddings OpenAI: Generates vector embeddings

Model: OpenAI Embeddings
Dimensions: 1536
Function: Creates semantic vectors for similarity search


Supabase Vector Store: Stores processed documents

Mode: Insert
Table: documents
Function: Stores document content, embeddings, and metadata



### 3. Document Update Management
Update Detection: Google Drive Trigger1

Watch Folder: Same KnowledgeBase folder
Event: fileUpdated
Polling: Every minute
Function: Detects when existing documents are modified

Update Processing Flow:

Edit Fields: Extracts file ID

Assignment: file_id = {{ $json.id }}
Function: Prepares file identifier for database operations


Supabase: Deletes old version

Operation: Delete
Table: documents
Filter: metadata->>file_id=like.*{{ $json.file_id }}
Function: Removes outdated document entries


Google Drive → Processing Chain: Re-processes updated document

Flow: Same as new document ingestion
Result: Fresh embeddings and content in database



### 4. Document Deletion Management
Deletion Detection: Google Drive Trigger2

Watch Folder: Recycle Bin (1ie3H_WHv5AHlKpgMnpga5SabDfiByL6j)
Event: fileCreated
Function: Detects when files are moved to trash

Deletion Processing:

Edit Fields1: Extracts deleted file ID
Supabase1: Removes from knowledge base

Operation: Delete
Filter: Same metadata-based filtering
Function: Maintains database consistency



## Data Flow Architecture
Document Ingestion Flow
Google Drive (New File) 
    ↓
Google Drive Node (Download)
    ↓
Default Data Loader (Process)
    ↓
OpenAI Embeddings (Vectorize)
    ↓
Supabase Vector Store (Store)
Query Processing Flow
Webhook (User Message)
    ↓
AI Agent (Process Query)
    ↓
Supabase Vector Store (Semantic Search)
    ↓
OpenAI Chat Model (Generate Response)
    ↓
Simple Memory (Store Context)
    ↓
Response to User
Update Management Flow
Google Drive (File Updated)
    ↓
Edit Fields (Extract ID)
    ↓
Supabase (Delete Old Version)
    ↓
Re-run Ingestion Pipeline
Technical Configuration
Authentication & Credentials

Google Drive OAuth2: Google Drive account 2
Supabase API: Supabase Test Account
OpenAI API: Two separate accounts for different operations

Chat model: OpenAi account
Embeddings: n8n free OpenAI API credits



## Database Schema
Supabase Table: documents

Content: Document text content
Embeddings: 1536-dimensional vectors
Metadata: JSON containing file_id for Google Drive tracking
Query Function: match_documents for similarity search

Memory Management

Type: Buffer Window Memory
Context Length: 10 messages
Session Key: Combination of sessionId and chatId
Function: Maintains conversation continuity

Key Features
Intelligent Document Processing

Automatic Detection: Real-time monitoring of Google Drive folders
Format Support: Handles various document formats through Default Data Loader
Semantic Indexing: Creates searchable vector embeddings
Metadata Tracking: Links database entries to source files

Advanced Query Processing

Semantic Search: Vector similarity for relevant content retrieval
Contextual Responses: AI agent with domain-specific instructions
Conversation Memory: Multi-turn dialogue support
Tool Integration: Vector store as a tool for the AI agent

Real-time Synchronization

Create: Automatic ingestion of new documents
Update: Re-processing of modified documents with cleanup
Delete: Removal from knowledge base when files are trashed
Consistency: Maintains database-file system alignment

Error Handling & Reliability

Polling Strategy: Minute-based checking for reliable detection
Unique Identification: File ID-based tracking prevents duplicates
Cleanup Operations: Automatic removal of stale entries
Credential Management: Separate API keys for different services

Workflow Optimization
Performance Considerations

Embedding Caching: 1536-dimensional vectors for efficient similarity search
Incremental Updates: Only processes changed documents
Memory Efficiency: Limited context window for conversation history
Polling Intervals: Balanced between responsiveness and API usage

Scalability Features

Modular Design: Separate triggers for different operations
Database Optimization: Vector similarity search with Supabase
API Rate Limiting: Distributed across multiple OpenAI accounts
Folder Organization: Structured Google Drive hierarchy

This workflow represents a production-ready knowledge base system that automatically maintains synchronization between document storage and a searchable AI-powered interface, providing intelligent responses based on current document content.
