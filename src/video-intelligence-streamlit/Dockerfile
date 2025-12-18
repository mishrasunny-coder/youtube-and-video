FROM python:3.12

EXPOSE 8080
WORKDIR /app

COPY . ./

# Install Node.js - needed for pytubefix PO token generation
# nodejs-wheel-binaries (pytubefix dependency) needs system nodejs to generate PO tokens
RUN apt-get update && apt-get install -y nodejs && rm -rf /var/lib/apt/lists/*

RUN pip install -r requirements.txt

ENV PORT=8080

# ENTRYPOINT ["streamlit", "run", "app.py", "--server.port=8080", "--server.address=0.0.0.0"]
ENTRYPOINT ["sh", "-c", "streamlit run app.py --server.port=${PORT} --server.address=0.0.0.0"]

