FROM python:3.11-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code into a 'backend' directory
COPY . /app/backend

# Ensure Python can find the 'backend' module
ENV PYTHONPATH=/app

# Expose FastAPI port
EXPOSE 8000

# Run the Uvicorn server
CMD ["python", "-m", "uvicorn", "backend.main:app", "--host", "0.0.0.0", "--port", "8000"]
