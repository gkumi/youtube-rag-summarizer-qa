# AI-Powered YouTube Summarizer & RAG Q&A

An AI-powered application that summarizes YouTube videos and answers questions about their content using **LangChain**, **IBM watsonx.ai**, **FAISS**, **Retrieval-Augmented Generation (RAG)**, and **Gradio**.

The application extracts a YouTube video's transcript, generates a concise summary using an IBM Granite language model, and allows users to ask questions that are answered using relevant transcript sections retrieved from a FAISS vector index.

## Features

* Extract transcripts from YouTube videos
* Prefer manually created English transcripts when available
* Process transcript text and timestamps
* Generate concise AI-powered video summaries
* Split transcripts into overlapping chunks
* Generate vector embeddings with IBM Granite Embeddings
* Store transcript embeddings in FAISS
* Perform semantic similarity search
* Retrieve transcript sections relevant to a user's question
* Generate context-grounded answers using IBM Granite
* Interactive Gradio web interface
* Temporary public Gradio sharing link

## Technologies

* **Python**
* **LangChain**
* **IBM watsonx.ai**
* **IBM Granite**
* **IBM Granite Embeddings**
* **FAISS**
* **Gradio**
* **YouTube Transcript API**

## Application Architecture

The application contains two AI workflows:

```text
                         YouTube Video URL
                                │
                                ▼
                    YouTube Transcript API
                                │
                                ▼
                       Video Transcript
                                │
                                ▼
                     Transcript Processing
                         │              │
              ┌──────────┘              └──────────┐
              ▼                                    ▼
     SUMMARIZATION                         RAG QUESTION ANSWERING
              │                                    │
              ▼                                    ▼
       Summary Prompt                      Transcript Chunking
              │                                    │
              ▼                                    ▼
       IBM Granite LLM                    Granite Embeddings
              │                                    │
              ▼                                    ▼
        Video Summary                         FAISS Index
                                                   │
                                            User Question
                                                   │
                                                   ▼
                                          Similarity Search
                                                   │
                                                   ▼
                                      Relevant Transcript Chunks
                                                   │
                                                   ▼
                                         Prompt Augmentation
                                                   │
                                                   ▼
                                          IBM Granite LLM
                                                   │
                                                   ▼
                                              Final Answer
```

## How the RAG Pipeline Works

The question-answering functionality uses **Retrieval-Augmented Generation (RAG)**.

### 1. Transcript Extraction

The application receives a YouTube URL and extracts the video ID.

The **YouTube Transcript API** is then used to retrieve an English transcript. A manually created transcript is preferred when available; otherwise, an auto-generated English transcript is used.

### 2. Transcript Processing

Each transcript segment contains spoken text and a timestamp.

The application converts the transcript into formatted text such as:

```text
Text: The vehicle uses a hybrid powertrain. Start: 418.56
Text: The vehicle produces 240 horsepower. Start: 425.03
Text: The vehicle costs approximately $54,000. Start: 524.48
```

### 3. Transcript Chunking

For question answering, the full transcript is divided into smaller overlapping chunks using LangChain's `RecursiveCharacterTextSplitter`.

```text
Full Transcript
      │
      ▼
   Chunk 1
   Chunk 2
   Chunk 3
   Chunk 4
      ...
```

Chunking makes semantic retrieval more precise and avoids sending the complete video transcript to the LLM for every question.

### 4. Embedding Generation

Each transcript chunk is converted into a numerical vector using:

```text
ibm/granite-embedding-278m-multilingual
```

Conceptually:

```text
Transcript Chunk
       │
       ▼
Embedding Model
       │
       ▼
[0.14, -0.32, 0.71, ...]
```

Embeddings allow transcript sections with similar meanings to be located even when they do not contain the exact same words as the user's question.

### 5. FAISS Vector Index

The transcript embeddings are stored in a **FAISS** vector index.

```text
Transcript Chunks
       │
       ▼
Embeddings
       │
       ▼
FAISS Vector Index
```

FAISS enables fast similarity searches over the transcript.

### 6. Semantic Retrieval

When a user asks:

```text
What is the price of the car?
```

the question is compared with the transcript embeddings.

FAISS retrieves the most relevant transcript chunks, which may include:

```text
This particular one, the way it's optioned is right at $54,000.
```

### 7. Prompt Augmentation

The retrieved transcript chunks are added to the user's question.

```text
User Question
      +
Retrieved Transcript Context
      +
Prompt Instructions
      │
      ▼
Augmented Prompt
```

This is the **Augmented** portion of Retrieval-Augmented Generation.

### 8. Response Generation

The augmented prompt is sent to:

```text
ibm/granite-4-h-small
```

through IBM watsonx.ai.

Granite generates an answer based on the retrieved video context.

```text
Question
    │
    ▼
FAISS Retrieval
    │
    ▼
Relevant Transcript Context
    │
    ▼
IBM Granite
    │
    ▼
Grounded Answer
```

## Summarization Workflow

The summarization feature does not require FAISS retrieval.

Instead, it follows:

```text
YouTube URL
    │
    ▼
Transcript
    │
    ▼
Transcript Processing
    │
    ▼
Summary Prompt
    │
    ▼
IBM Granite
    │
    ▼
Video Summary
```

The model is instructed to:

* Produce a concise summary
* Ignore timestamps
* Focus on the spoken content
* Capture the main points of the video

## Project Structure

```text
youtube-rag-summarizer-qa/
│
├── ytbot.py
├── README.md
├── requirements.txt
└── .gitignore
```

### `ytbot.py`

The main application containing:

* YouTube URL parsing
* Transcript extraction
* Transcript preprocessing
* Text chunking
* IBM watsonx.ai configuration
* IBM Granite LLM initialization
* IBM Granite embedding initialization
* FAISS index creation
* Similarity retrieval
* Summary generation
* RAG question answering
* Gradio interface

## Installation

Clone the repository:

```bash
git clone https://github.com/gkumi/youtube-rag-summarizer-qa.git
```

Navigate to the project directory:

```bash
cd youtube-rag-summarizer-qa
```

Create a virtual environment:

```bash
python3 -m venv my_env
```

Activate it:

```bash
source my_env/bin/activate
```

Install the required dependencies:

```bash
pip install -r requirements.txt
```

## Running the Application

Start the Gradio application:

```bash
python ytbot.py
```

The application runs on:

```text
http://0.0.0.0:7860
```

Because Gradio sharing is enabled, a temporary public URL is also generated:

```text
https://xxxxxxxxxxxxxxxx.gradio.live
```

The public URL remains available only while the application is running and is temporary.

## Using the Application

### Summarize a Video

1. Paste a supported YouTube video URL.
2. Click **Summarize Video**.
3. The transcript is retrieved and processed.
4. IBM Granite generates a concise summary.
5. The summary appears in the **Video Summary** field.

### Ask Questions About a Video

Enter a question such as:

```text
What is the price of the car?
```

or:

```text
What engine does the vehicle use?
```

Then click **Ask a Question**.

The application:

```text
Question
   ↓
Transcript Chunking
   ↓
Granite Embeddings
   ↓
FAISS Similarity Search
   ↓
Relevant Transcript Chunks
   ↓
IBM Granite
   ↓
Answer
```

## Example

### YouTube Video

A vehicle review discussing a 2026 Toyota Crown Signia.

### Generated Summary

The application can summarize information such as:

* Vehicle design
* Hybrid powertrain
* Fuel economy
* Interior features
* Driving experience
* Comparison with competing vehicles

### Example Question

```text
What is the price of the car?
```

### Retrieved Context

```text
This particular one, the way it's optioned is right at $54,000.
```

### Generated Answer

```text
The vehicle is priced at approximately $54,000 as configured in the video.
```

## RAG Components

The RAG implementation can be summarized as:

```text
Retrieval
FAISS retrieves transcript chunks relevant to the question.

Augmentation
The retrieved chunks are inserted into the LLM prompt.

Generation
IBM Granite generates an answer using the retrieved context.
```

Or simply:

```text
R = FAISS
A = Retrieved transcript context added to the prompt
G = IBM Granite
```

## Models

### Generation Model

```text
ibm/granite-4-h-small
```

Used for:

* Video summarization
* Question answering

### Embedding Model

```text
ibm/granite-embedding-278m-multilingual
```

Used for:

* Transcript chunk embeddings
* Query embeddings
* Semantic retrieval

## Current Limitations

* Requires a YouTube transcript to be available
* Currently focuses on English transcripts
* YouTube URL parsing currently expects standard `youtube.com/watch?v=` URLs
* FAISS index is created in memory for the current transcript
* Public Gradio links are temporary
* Very long transcripts may require more advanced summarization strategies

## Future Improvements

* Support `youtu.be` and YouTube Shorts URLs
* Cache transcripts and FAISS indexes
* Add conversational Q&A history
* Add timestamps to retrieved answers
* Allow users to jump to the relevant video segment
* Add streaming LLM responses
* Add source citations for retrieved transcript chunks
* Improve transcript chunk size and overlap tuning
* Add similarity score thresholds
* Add multi-query or MMR retrieval
* Replace deprecated LangChain `LLMChain` usage with LCEL / RunnableSequence
* Deploy the Gradio application permanently
* Add automated testing and improved error handling

## Learning Objectives

This project demonstrates practical experience with:

* Retrieval-Augmented Generation
* Vector embeddings
* Semantic similarity search
* Vector indexing
* FAISS
* LangChain
* Prompt engineering
* Transcript processing
* IBM watsonx.ai
* IBM Granite
* Gradio
* Building end-to-end generative AI applications

## Project Goal

The goal of this project is to demonstrate how unstructured multimedia content can be transformed into searchable knowledge.

By combining transcript extraction, embeddings, semantic retrieval, RAG, and an LLM, the application allows users to interact with long-form YouTube content without manually searching through an entire video.
