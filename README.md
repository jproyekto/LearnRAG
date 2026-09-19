# LearnRAG
Learning RAG using python and ollama.


## convo with copilot

Bro, this is actually a PERFECT beginner project for learning RAG.

The good news is that your Resume Analyzer is much simpler than a full ChatGPT-style RAG system. You can start with a very small version and gradually make it smarter.

First: What is RAG?

RAG = Retrieval Augmented Generation

Think of it like this:

Without RAG:

Resume → LLM


The model only knows what is in its training data.

With RAG:

Resume
   ↓
Text Extraction
   ↓
Chunking
   ↓
Embeddings
   ↓
Vector Database
   ↓
Relevant Chunks Retrieved
   ↓
LLM
   ↓
Answer


Instead of sending the entire document to the model, we:

Store document information in pieces
Search the most relevant pieces
Give those pieces to the LLM
LLM answers using retrieved information

This is why it is called:

Retrieval + Augmented + Generation

Resume Analyzer Architecture

Let's build this:

resume.docx

        ↓

Extract Text

        ↓

Split into Chunks

        ↓

Create Embeddings

        ↓

Store in ChromaDB

        ↓

User asks question

        ↓

Find Relevant Chunks

        ↓

Send Chunks + Question

        ↓

Llama2-Uncensored

        ↓

Answer

Step 1: Install Required Packages

Create a virtual environment:

python -m venv venv


Activate:

Windows:

venv\Scripts\activate


Linux/Mac:

source venv/bin/activate


Install packages:

pip install chromadb
pip install python-docx
pip install ollama
pip install sentence-transformers

Step 2: Read Resume

Let's read a Word document.

from docx import Document

def read_docx(path):
    doc = Document(path)

    text = "\n".join(
        para.text for para in doc.paragraphs
    )

    return text


Usage:

text = read_docx("resume.docx")

print(text)

Step 3: Create Chunks

Don't store one giant document.

Split it.

def chunk_text(text, chunk_size=500):

    chunks = []

    for i in range(0, len(text), chunk_size):

        chunks.append(
            text[i:i + chunk_size]
        )

    return chunks


Example:

chunks = chunk_text(text)


Result:

[
 "John Doe...",
 "Worked as software...",
 "Skills include..."
]

Step 4: Create Embeddings

Embeddings convert text into numbers.

Example:

"Python Developer"

↓

[0.24, 0.73, 0.12, ...]


These vectors allow similarity search.

For beginners, use:

from sentence_transformers import SentenceTransformer

model = SentenceTransformer(
    "all-MiniLM-L6-v2"
)


Generate embeddings:

embeddings = model.encode(chunks)

Step 5: Store in ChromaDB

Create database:

import chromadb

client = chromadb.Client()

collection = client.create_collection(
    name="resume"
)


Store chunks:

for idx, chunk in enumerate(chunks):

    collection.add(
        ids=[str(idx)],
        documents=[chunk]
    )


Simple.

Step 6: Search Resume

Suppose user asks:

What programming languages does this candidate know?


Search:

results = collection.query(
    query_texts=[
        "What programming languages does this candidate know?"
    ],
    n_results=3
)


Get chunks:

print(results["documents"])


Output:

[
 "Skills: Python, Java...",
 "Experience using Django...",
 "Built Flask API..."
]


These are the relevant chunks.

Step 7: Send to Ollama

Now feed retrieved chunks to llama2-uncensored.

import ollama


Prompt:

context = "\n".join(
    results["documents"][0]
)

prompt = f"""
You are a resume analyzer.

Resume Information:

{context}

Question:

What programming languages
does this candidate know?
"""


Call Ollama:

response = ollama.chat(
    model='llama2-uncensored',
    messages=[
        {
            'role':'user',
            'content':prompt
        }
    ]
)

print(
    response['message']['content']
)


Boom.

You just built a basic RAG.

Complete Beginner Flow
resume.docx
       ↓

Extract Text
       ↓

Chunk Text
       ↓

Store Chunks
       ↓

User Question
       ↓

Retrieve Chunks
       ↓

Send Context + Question
       ↓

LLM Answer

Resume Analyzer Features You Can Build
Version 1

Upload resume

Ask:

What skills does this candidate have?

Version 2

Extract:

Skills
Experience
Education
Certifications
Projects


Automatically.

Version 3

Compare Resume vs Job Description

Input:

Resume.pdf

Job Description.txt


Output:

Match Score: 82%

Missing Skills:
- Docker
- AWS

Strengths:
- Python
- FastAPI
- SQL


This is where RAG becomes very powerful.

Better Architecture (What I'd Recommend)

Instead of:

SentenceTransformer


Use Ollama embeddings directly.

Pull embedding model:

ollama pull nomic-embed-text


Generate embeddings:

embedding = ollama.embeddings(
    model='nomic-embed-text',
    prompt=text
)


Now:

Resume
     ↓
nomic-embed-text
     ↓
ChromaDB
     ↓
Retrieve
     ↓
llama2-uncensored


Everything stays local.

No OpenAI API needed.

Learning Roadmap

Week 1:

Learn DOCX reading
Learn chunking
Learn ChromaDB

Week 2:

Learn embeddings
Learn semantic search

Week 3:

Connect Ollama

Week 4:

Build Resume Analyzer UI with Flask or Streamlit

A very practical stack would be:

Ollama
Llama2-Uncensored
nomic-embed-text
ChromaDB
Python
Streamlit


That's enough to build a fully local AI Resume Analyzer that feels surprisingly professional and teaches you the core concepts behind modern RAG systems.