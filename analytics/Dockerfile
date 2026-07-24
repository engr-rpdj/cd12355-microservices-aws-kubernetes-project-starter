# Slim Python base image keeps the final image small while matching
# a Python 3.8+ runtime as required by the application.
FROM python:3.10-slim

WORKDIR /app

# Copy only the requirements file first so Docker can cache this layer
# and skip reinstalling dependencies when only app code changes.
COPY requirements.txt .

# psycopg2-binary ships precompiled, so no build-essential/libpq-dev
# needed here, keeping the image lean.
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# Default port the Flask app listens on (see app.py), overridable
# via the APP_PORT environment variable.
ENV APP_PORT=5153
EXPOSE 5153

CMD ["python", "app.py"]
