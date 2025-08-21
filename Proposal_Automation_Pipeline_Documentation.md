# Proposal Automation Pipeline - Documentation

## Overview

The **Proposal Automation Pipeline (Ingestion Workflow) - POC** is an N8N workflow designed to automate the process of document ingestion, processing, and proposal generation using RAG (Retrieval-Augmented Generation) AI capabilities. The workflow combines document management, AI-powered content analysis, and automated proposal creation into a single integrated system.

## Architecture

This workflow consists of two main components:

### 1. Data Pipeline (Document Ingestion)
- **Purpose**: Automatically ingest and process documents from Google Drive
- **Trigger**: File creation/updates in monitored Google Drive folders
- **Processing**: Extract text, create embeddings, and store in vector database

### 2. RAG AI Agent (Chat Interface)
- **Purpose**: Interactive chat interface for proposal generation
- **Input**: Chat messages via Telegram or web interface
- **Output**: AI-generated proposals automatically formatted as Google Docs

---

## Key Features

### Document Processing
- **Multi-format Support**: PDF, Word documents, Excel, CSV, and text files
- **Recursive Folder Scanning**: Automatically processes nested folder structures
- **Duplicate Detection**: Prevents reprocessing of existing documents
- **Vector Storage**: Documents are embedded and stored in Supabase vector database

### AI-Powered Proposal Generation
- **RAG Integration**: Uses document knowledge base for contextual responses
- **Structured Output**: Generates proposals following PRD (Product Requirements Document) template
- **Auto-formatting**: Automatically creates formatted Google Docs
- **Chat Memory**: Maintains conversation context using PostgreSQL

### Integration Capabilities
- **Google Drive**: File monitoring and document creation
- **Supabase**: Vector storage and document metadata
- **PostgreSQL**: Structured data storage and chat memory
- **OpenAI**: Language model and embeddings
- **Telegram**: Chat interface for user interaction

---

## Workflow Components

### Triggers

#### 1. Manual Trigger
- **Node**: `When clicking 'Test workflow'`
- **Purpose**: Manual execution for testing and development

#### 2. Google Drive Triggers
- **File Created**: `Google Drive Trigger`
- **File Updated**: `Google Drive Trigger1`
- **Monitored Folder**: `Proposal-Test` (ID: 1uaHkFKBgArjlPpBr9xJWE5FhOLbpM6SG)
- **Polling**: Every minute

#### 3. Chat Triggers
- **Web Chat**: `When chat message received`
- **Telegram**: `Telegram Trigger`

### Document Processing Flow

#### 1. File Discovery
```
Google Drive → Loop Over Items1 → Check File/Folder
```
- Scans Google Drive folder for new/updated files
- Filters out folders, processes only supported file types
- Recursive processing for nested folder structures

#### 2. File Processing
```
Set File ID → Delete Old Records → Download File → Switch (File Type)
```
- Extracts file metadata (ID, type, title, URL)
- Removes existing records to prevent duplicates
- Downloads file content for processing
- Routes to appropriate extraction method based file type

#### 3. Content Extraction
- **PDF Files**: `Extract PDF Text`
- **Excel Files**: `Extract from Excel`
- **CSV Files**: `Extract from CSV1`
- **Documents**: `Extract Document Text`

#### 4. Data Storage
- **Vector Storage**: Text content → embeddings → Supabase vector store
- **Metadata Storage**: File information → PostgreSQL
- **Structured Data**: CSV/Excel rows → PostgreSQL tables

### AI Agent Flow

#### 1. Chat Processing
```
Chat Input → RAG AI Agent → Response Generation
```
- Receives user messages from multiple channels
- Processes queries using RAG capabilities
- Generates contextual responses

#### 2. Proposal Generation
When users request proposal generation:
1. **RAG Lookup**: Searches vector database for relevant context
2. **Template Application**: Uses comprehensive PRD template
3. **Content Generation**: Creates detailed proposal sections
4. **Document Creation**: Formats and saves as Google Docs
5. **Notification**: Sends confirmation via multiple channels

### Tools Available to AI Agent

#### Database Tools
- **List Documents**: `List Documents` - Browse available documents
- **Get File Contents**: `Get File Contents` - Extract full text from specific files
- **Query Document Rows**: `Query Document Rows` - SQL queries on structured data

#### RAG Tools
- **Vector Search**: `Supabase Vector Store` - Semantic search across documents
- **Document Retrieval**: Top 6 most relevant document chunks

#### Output Tools
- **Google Docs Creation**: `Markdown to Google Docs` - Create formatted documents
- **Communication**: Gmail, Slack, Telegram notifications

---

## Technical Specifications

### Supported File Types
- **Documents**: PDF, Word (.docx), Text files
- **Spreadsheets**: Excel (.xlsx), CSV
- **Folders**: Recursive processing

### Storage Systems
- **Vector Database**: Supabase with OpenAI embeddings
- **Relational Database**: PostgreSQL for metadata and chat history
- **File Storage**: Google Drive

### AI Models
- **Primary Model**: GPT-4.1 for main processing
- **Fallback Model**: GPT-4o-mini for auto-fixing
- **Embeddings**: OpenAI text-embedding-ada-002

### Output Formats
- **Proposals**: Structured markdown → Google Docs
- **Notifications**: Plain text via multiple channels
- **Data**: JSON structures for API responses

---

## Data Flow

### Ingestion Pipeline
1. **Trigger**: File created/updated in Google Drive
2. **Discovery**: Scan folder contents recursively
3. **Validation**: Check file type and existence
4. **Cleanup**: Remove old database records
5. **Download**: Retrieve file content
6. **Extraction**: Process based on file type
7. **Embedding**: Create vector representations
8. **Storage**: Save to databases
9. **Completion**: Return to folder scanning

### Query Pipeline
1. **Input**: User chat message
2. **RAG Search**: Find relevant documents
3. **Context Building**: Gather additional data if needed
4. **Generation**: Create response using AI model
5. **Formatting**: Structure output appropriately
6. **Delivery**: Send via original channel
7. **Memory**: Store conversation context

---

## Configuration

### Credentials Required
- **Google Drive OAuth2**: File access and creation
- **Google Docs OAuth2**: Document creation
- **Supabase API**: Vector database access
- **PostgreSQL**: Structured data storage
- **OpenAI API**: Language model and embeddings
- **Telegram Bot**: Chat interface
- **Gmail OAuth2**: Email notifications

### Database Schema

#### document_metadata
- `id` (string): Unique file identifier
- `title` (string): File name
- `url` (string): Google Drive link
- `created_at` (datetime): Timestamp
- `schema` (string): File structure for CSV/Excel

#### document_rows
- `id` (number): Auto-increment primary key
- `dataset_id` (string): References document_metadata.id
- `row_data` (jsonb): Structured data from CSV/Excel

#### documents (Supabase Vector Store)
- Vector embeddings with metadata
- Content chunks for semantic search

---

## Proposal Template Structure

The AI agent uses a comprehensive PRD template including:

### Core Sections
1. **Introduction**: Purpose, background, scope
2. **Product Overview**: Vision, key features, user stories
3. **Functional Requirements**: Detailed specifications
4. **Non-Functional Requirements**: Performance, security, usability
5. **Dependencies and Assumptions**: Technical and business constraints
6. **Timeline**: Milestones, MVP scope, critical path
7. **Risks and Mitigation**: Technical, project, and business risks
8. **Conclusion**: Summary, benefits, success measures

### Formatting Features
- Page breaks between major sections
- Hierarchical headings (H1, H2, H3)
- Bullet points and numbered lists
- Tables and diagram placeholders
- Professional styling for stakeholder presentation

---

## Error Handling

### Auto-fixing Parser
- **Purpose**: Corrects malformed AI outputs
- **Fallback Model**: GPT-4o-mini for corrections
- **Validation**: Ensures JSON schema compliance
- **Retry Logic**: Single retry attempt for failed outputs

### Data Validation
- **Duplicate Prevention**: Checks existing records before insertion
- **File Type Validation**: Ensures supported formats only
- **Content Verification**: Validates extracted text quality

---

## Monitoring and Notifications

### Success Notifications
- **Telegram**: Real-time chat responses
- **Gmail**: Email summaries (disabled by default)
- **Slack**: Team notifications (disabled by default)

### Error Handling
- **Graceful Degradation**: Continues processing on individual file failures
- **Logging**: Comprehensive error tracking
- **Recovery**: Automatic retry mechanisms

---

## Usage Scenarios

### Document Management
1. **Bulk Import**: Upload multiple documents to monitored folder
2. **Automatic Processing**: System processes files automatically
3. **Query Documents**: Ask questions about document contents
4. **Data Analysis**: SQL queries on structured data

### Proposal Generation
1. **Request**: "Generate a proposal for [project]"
2. **Context Gathering**: AI searches relevant documents
3. **Template Application**: Structures content using PRD format
4. **Document Creation**: Generates formatted Google Doc
5. **Delivery**: Provides document link

### Interactive Chat
1. **General Questions**: Ask about any documents in the system
2. **Data Queries**: Request specific information or statistics
3. **Document Analysis**: Compare multiple documents
4. **Follow-up**: Maintain conversation context

---

## Future Enhancements

### Planned Features (from sticky notes)
- **Enhanced Chat Interface**: Embedded web chat component
- **Advanced File Detection**: Improved duplicate detection algorithms
- **Custom PRD Formatting**: Specialized proposal templates
- **Performance Optimization**: Faster processing for large documents

### Scalability Considerations
- **Batch Processing**: Handle large document sets efficiently
- **Caching**: Reduce redundant API calls
- **Load Balancing**: Distribute processing across multiple instances
- **Archive Management**: Handle document lifecycle

---

## Security and Privacy

### Data Protection
- **OAuth2 Authentication**: Secure API access
- **Encrypted Storage**: Database encryption at rest
- **Access Control**: Role-based permissions
- **Audit Logging**: Track all document access

### Privacy Considerations
- **Data Retention**: Configurable document lifecycle
- **Access Restrictions**: Folder-based access control
- **Anonymization**: Option to remove sensitive data
- **Compliance**: Supports GDPR and other privacy regulations

---

## Troubleshooting

### Common Issues
1. **File Processing Failures**: Check file format and permissions
2. **Vector Storage Errors**: Verify Supabase connection and quotas
3. **AI Response Issues**: Monitor OpenAI API limits and model availability
4. **Google Drive Sync**: Confirm OAuth2 token validity

### Performance Optimization
- **Polling Frequency**: Adjust trigger intervals based on usage
- **Batch Size**: Configure loop processing limits
- **Memory Management**: Monitor PostgreSQL connections
- **API Limits**: Implement rate limiting and backoff strategies

---

## Conclusion

The Proposal Automation Pipeline represents a comprehensive solution for automated document processing and AI-powered proposal generation. By combining modern AI capabilities with robust document management, it enables organizations to streamline their proposal creation process while maintaining high quality and consistency.

The system's modular design allows for easy customization and extension, making it suitable for various business contexts and requirements. The integration of multiple communication channels ensures accessibility, while the structured data storage enables powerful querying and analysis capabilities.