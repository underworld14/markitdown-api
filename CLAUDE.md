# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MarkItDown API Server is a FastAPI-based REST API that converts various document formats (PDF, Word, Excel, PowerPoint, etc.) to Markdown using Microsoft's MarkItDown library. The application is containerized and designed to run in Docker.

## Development Commands

### Docker Development
```bash
# Build and run the Docker container
docker build -t markitdown-api:latest .
docker run -d --name markitdown-api -p 8490:8490 markitdown-api:latest

# Quick rebuild during development (stops, rebuilds, restarts)
./rebuild.sh
```

### Local Development
```bash
# Run the application locally (if not using Docker)
python app.py
# OR
uvicorn app:app --host 0.0.0.0 --port 8490

# Test the API
curl -X POST -F "file=@path/to/file.pdf" http://localhost:8490/process_file
```

## Architecture

### Core Components

1. **app.py** - Main FastAPI application containing:
   - `/process_file` endpoint: Accepts file uploads via multipart/form-data, validates file types, converts to markdown
   - File type validation: Blocks forbidden file extensions (executables, scripts, system files) defined in `FORBIDDEN_EXTENSIONS`
   - Temporary file handling: Uses `tempfile.NamedTemporaryFile` for secure file processing with automatic cleanup
   - MarkItDown integration: `convert_to_md()` function wraps the MarkItDown library

2. **utils/file_handler.py** - Utility functions for file operations (currently unused in main app):
   - `save_temp_file()`: Saves binary data to temp directory
   - `delete_file()`: Removes files from filesystem

### Request Flow

1. Client uploads file via POST to `/process_file`
2. File extension validated against `FORBIDDEN_EXTENSIONS` list
3. File written to temporary location using `tempfile.NamedTemporaryFile`
4. MarkItDown library processes file to extract text and convert to markdown
5. Temporary file deleted in finally block
6. JSON response returned with markdown content: `{"markdown": "..."}`

### Docker Architecture

Multi-stage build using Python 3.13:
- **Builder stage**: Installs dependencies using `uv` package manager into virtual environment
- **Final stage**: Minimal runtime image with virtual environment, runs as non-root user `appuser`
- Port 8490 exposed for API access

## Key Implementation Details

- **File validation**: Uses extension-based blocking rather than MIME type validation
- **Temporary file cleanup**: Guaranteed cleanup in finally block after processing
- **Logging**: Structured logging at INFO level for file processing lifecycle
- **Error handling**: Returns 400 for forbidden files, 500 for processing errors
- **Dependencies**: Uses `uv` for faster dependency installation in Docker builds

## Accepted File Types

The API accepts common document formats that MarkItDown can process:
- Documents: doc, docx, pdf, txt
- Spreadsheets: xls, xlsx, csv
- Presentations: ppt, pptx
- Data: json

Refer to the MarkItDown library documentation for the complete list of supported formats.
