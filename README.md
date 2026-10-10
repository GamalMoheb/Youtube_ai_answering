# 🎬 YouTube AI Assistant

A Streamlit application that lets you load a YouTube video, retrieve its available captions, and ask questions about the transcript using a Retrieval-Augmented Generation (RAG) pipeline.

The app supports English and Arabic transcripts, semantic search with FAISS, and answer generation using an instruction-tuned language model.

## ✨ Features

- **YouTube transcript extraction** from supported video URLs.
- **English and Arabic support** when captions are available.
- **Video URL parsing** for standard YouTube links, shortened links, Shorts, embeds, and live links.
- **Text chunking with overlap** to preserve context between transcript segments.
- **Multilingual semantic embeddings** using `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2`.
- **FAISS similarity search** to retrieve transcript chunks relevant to a question.
- **LLM-powered Q&A** using a selectable instruction model:
  - `Qwen/Qwen2.5-3B-Instruct`
  - `mistralai/Mistral-Nemo-Instruct-2407`
- **Streamlit chat interface** with adjustable chunking and retrieval settings.
- **Source transparency** by showing the transcript chunks retrieved for each answer.
- **4-bit quantization on CUDA** to reduce language-model memory usage when supported.

## 🧠 How It Works

1. **Enter a YouTube URL.** The app extracts the video ID.
2. **Fetch captions.** It requests captions in the selected language(s). The video must have accessible captions; this app does not transcribe raw audio.
3. **Chunk the transcript.** The transcript is split into overlapping word-based chunks.
4. **Create embeddings.** A multilingual Sentence Transformers model converts each chunk into a vector.
5. **Build a FAISS index.** Normalized embeddings are indexed for cosine-similarity search.
6. **Retrieve context.** When you ask a question, the app retrieves the most relevant chunks.
7. **Generate an answer.** The selected instruction-tuned LLM answers using the retrieved context.
8. **Inspect sources.** You can expand the source panel to review the chunks used.

## 🛠️ Tech Stack

- Python
- [Streamlit](https://streamlit.io/)
- [YouTube Transcript API](https://github.com/jdepoix/youtube-transcript-api)
- [Hugging Face Transformers](https://huggingface.co/docs/transformers)
- [Sentence Transformers](https://www.sbert.net/)
- [FAISS](https://github.com/facebookresearch/faiss)
- [PyTorch](https://pytorch.org/)
- [Accelerate](https://github.com/huggingface/accelerate)
- [BitsAndBytes](https://github.com/bitsandbytes-foundation/bitsandbytes)
- [pyngrok](https://pyngrok.readthedocs.io/)

## 📋 Requirements

- Python 3.10+ recommended.
- A GPU runtime is strongly recommended for practical LLM inference.
- Internet access to download models and fetch YouTube captions.
- An ngrok account and authentication token if exposing the app through a public tunnel from Colab.

> **Note:** CPU inference may be very slow. Model loading and generation memory requirements depend on the selected model and available hardware. Four-bit quantization is configured when CUDA is available.

## 🚀 Run in Google Colab

The notebook workflow is intended for Google Colab. The app file is named `app.py`.

### 1. Install dependencies

Run this in a Colab code cell:

```python
!pip -q install \
    "transformers==4.52.4" \
    "youtube-transcript-api" \
    "sentence-transformers" \
    "faiss-cpu" \
    "streamlit" \
    "accelerate" \
    "bitsandbytes" \
    "pyngrok"
```

### 2. Create `app.py`

Put the Streamlit application code in a Colab cell beginning with:

```python
%%writefile app.py
```

Then paste the application code below that line and run the cell.

### 3. Start Streamlit and expose it with ngrok

Run the launch cell from your notebook. It should:
- Prompt for your ngrok authentication token without displaying it.
- Start Streamlit on port `8501`.
- Wait for Streamlit's health endpoint to respond.
- Create an ngrok tunnel and print a public URL.

**Security:** Never commit your ngrok token to GitHub or paste it into public code. If a token has been exposed, revoke it in your ngrok dashboard and create a new one. Use `getpass.getpass()` or Colab Secrets to enter it privately.

### 4. Open the application

Open the public ngrok URL printed by the notebook. On ngrok's free tier, a browser warning page may appear; click **Visit Site** to continue.

Keep the Colab runtime running while using the app. The public tunnel URL is temporary and may change after restarting the runtime.

## 💬 Using the App

1. Open the app URL.
2. Select the transcript language: English, Arabic, or English + Arabic.
3. Select a language model.
4. Paste a YouTube URL.
5. Click **Fetch transcript and build index**.
6. Wait for the transcript to be processed and indexed.
7. Ask questions in the chat box.
8. Expand **View retrieved transcript chunks** to inspect the evidence used for an answer.
9. Adjust chunk size, overlap, and the number of retrieved chunks in the sidebar as needed.

## ⚙️ Configuration

| Setting | Description |
|---|---|
| Transcript language | Chooses which available caption language(s) to request. |
| Language model | Chooses Qwen 2.5 3B or Mistral Nemo Instruct. |
| Chunk size | Approximate number of words per transcript chunk. |
| Chunk overlap | Number of words repeated between adjacent chunks. |
| Retrieved chunks (`top_k`) | Number of relevant transcript chunks supplied as context. |

The overlap must be smaller than the chunk size.

## ⚠️ Limitations

- The app depends on captions that the YouTube Transcript API can access. Videos without accessible captions cannot be processed.
- It does not perform speech recognition or create transcripts directly from audio.
- The answer quality depends on caption quality, retrieval relevance, the selected model, and available context.
- Questions requiring information spread across an entire long video may need a dedicated whole-transcript summarization workflow.
- The model may still produce incorrect answers. Check the retrieved transcript excerpts and verify important information.
- GPU availability and memory vary across Colab sessions. Mistral Nemo is larger and may exceed available memory even with quantization.
- The ngrok public URL is temporary. Anyone with access to the URL may be able to access the running app, so avoid loading sensitive content.

## 🔐 Security

- Do not store API tokens or passwords in `app.py`, notebooks committed to GitHub, or README files.
- Use a private input prompt or Colab Secrets for the ngrok token.
- Revoke and replace any token that has been exposed.
- Consider who can access the public tunnel before sharing it.

## 📁 Suggested Repository Structure

```text
youtube-ai-assistant/
├── app.py
├── README.md
└── requirements.txt
```

You can place the installation dependencies in a `requirements.txt` file:

```text
transformers==4.52.4
youtube-transcript-api
sentence-transformers
faiss-cpu
streamlit
accelerate
bitsandbytes
pyngrok
```

The `pyngrok` dependency is only needed for the Colab tunnel launcher; it is not required when deploying Streamlit on a platform that provides its own public URL.

## 📜 License

Add a license file if you want others to reuse, modify, or distribute this project. Make sure to review the licenses of the models and libraries you use.
