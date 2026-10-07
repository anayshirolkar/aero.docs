# Aero Docs

A FastAPI service for indexing PDF documentation and related media, then answering questions about the indexed content with citations.

The service uses [Embedchain](https://github.com/embedchain/embedchain) for retrieval and storage, OpenAI for image descriptions and hosted model access, and can also be configured to use a local Ollama instance.

## Features

- Upload and index PDF files with title, comments, and related media metadata.
- Describe related images with `gpt-4o-mini` and index those descriptions.
- Ask questions against the indexed documentation.
- Return source citations with generated answers.
- Serve uploaded files from the local `files/` directory.
- Deploy as a Python application on Vercel using the included `vercel.json`.

## Requirements

- Python 3.10 or newer
- An OpenAI API key when using the default OpenAI configuration
- Ollama running locally when using `config-local.yaml`

## Installation

```bash
git clone https://github.com/anayshirolkar/aero.docs.git
cd aero.docs

python -m venv env
```

Activate the virtual environment:

**Windows PowerShell**

```powershell
.\env\Scripts\Activate.ps1
```

**macOS/Linux**

```bash
source env/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Create a `.env` file in the project root:

```dotenv
OPENAI_API_KEY=your-openai-api-key
```

Do not commit `.env` or API keys. `.env` is excluded by `.gitignore`.

## Configuration

The application loads `config.yaml` by default. The checked-in configuration uses OpenAI for the language model and embeddings:

```yaml
llm:
  provider: openai
embedder:
  provider: openai
```

For local development with Ollama, update the application to load `config-local.yaml` and make sure Ollama is running at `http://localhost:11434`. The local configuration uses:

- `wizardlm2:7b` for the language model
- `znbang/bge:small-en-v1.5-q8_0` for embeddings

## Running locally

Start the development server with:

```bash
uvicorn app:app --reload
```

The API will be available at <http://127.0.0.1:8000>. Interactive API documentation is available at:

- Swagger UI: <http://127.0.0.1:8000/docs>
- ReDoc: <http://127.0.0.1:8000/redoc>

## API

### Upload a PDF

`POST /upload_pdf`

Send a JSON body containing the PDF path or URL accepted by Embedchain, optional metadata, and a JSON-encoded list of related media:

```json
{
  "file": "https://example.com/manual.pdf",
  "title": "Product Manual",
  "comments": "Installation and maintenance instructions",
  "videoUrls": "[{\"type\":\"video\",\"url\":\"https://example.com/overview.mp4\"},{\"type\":\"image\",\"url\":\"https://example.com/diagram.png\"}]"
}
```

The endpoint indexes the PDF, title and comments, and image descriptions. It returns:

```json
{
  "file_name": "https://example.com/manual.pdf"
}
```

### Ask a question

`GET /ask_question`

Query parameters:

- `message`: The question to ask.
- `type`: Optional metadata type filter.
- `imageUrls`: A JSON-encoded list of image URLs to describe and include with the question.

Example:

```text
/ask_question?message=How%20do%20I%20install%20the%20device%3F&imageUrls=[]
```

Response:

```json
{
  "answer": "Generated answer",
  "sources": []
}
```

### Retrieve a file

`GET /files/{file_name}`

Returns a file from the local `files/` directory.

## Project structure

```text
.
├── app.py              # FastAPI application and API endpoints
├── config.yaml         # Default Embedchain/OpenAI configuration
├── config-local.yaml   # Optional Ollama configuration
├── requirements.txt    # Python dependencies
├── vercel.json         # Vercel deployment configuration
├── database.db         # Local application database
└── db/                 # Embedchain/Chroma persistence data
```

The local database and vector-store files are environment-specific generated data. Review whether they should be retained before deploying to a new environment.

## Deployment

The included `vercel.json` configures Vercel to run `app.py` with the Python runtime. Configure `OPENAI_API_KEY` as a Vercel environment variable before deploying.

```bash
vercel
```

For production deployments, use a persistent external vector store and object storage rather than relying on local filesystem state.

## License

No license is currently specified for this project.
