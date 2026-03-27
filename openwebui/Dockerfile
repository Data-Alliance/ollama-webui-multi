FROM ghcr.io/open-webui/open-webui:main

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
       locales \
       fonts-nanum \
       tesseract-ocr \
       tesseract-ocr-kor \
       tesseract-ocr-eng \
    && echo 'ko_KR.UTF-8 UTF-8' > /etc/locale.gen \
    && locale-gen \
    && rm -rf /var/lib/apt/lists/*

RUN pip install --no-cache-dir --upgrade \
    torch \
    torchvision \
    torchaudio \
    --index-url https://download.pytorch.org/whl/cu128

ENV LANG=ko_KR.UTF-8
ENV LC_ALL=ko_KR.UTF-8
ENV PYTHONIOENCODING=utf-8
ENV DATA_DIR=/workplace/service_v1/data
ENV UPLOAD_DIR=/workplace/service_v1/uploads
ENV OLLAMA_BASE_URL=http://localhost:11434
ENV OLLAMA_REQUEST_TIMEOUT=1800
ENV DEFAULT_MODELS=deepseek-r1:8b
ENV TIMEOUT=3600
ENV UPLOAD_TIMEOUT=3600
ENV VECTOR_DB=chroma
ENV ENABLE_RAG_WEB_SEARCH=false
ENV RAG_EMBEDDING_ENGINE=ollama
ENV RAG_EMBEDDING_MODEL=nomic-embed-text
ENV RAG_EMBEDDING_CHUNK_SIZE=1000
ENV RAG_EMBEDDING_BATCH_SIZE=2
ENV RAG_FAIL_ON_ERROR=false
ENV ENABLE_OCR=true
ENV OCR_ENGINE=tesseract
ENV OCR_LANG=kor+eng
ENV PDF_EXTRACT_IMAGES=false
ENV FILE_UPLOAD_MAX_SIZE=2147483648
ENV MAX_CONTENT_LENGTH=2147483648
ENV ENABLE_CHUNKED_UPLOAD=true

EXPOSE 8080
