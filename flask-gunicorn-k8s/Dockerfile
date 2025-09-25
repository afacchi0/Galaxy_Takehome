# Use a lightweight Python image
FROM python:3.11-alpine

# Set working directory
WORKDIR /app

# Install Flask
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the app code
COPY app.py .

# Use Gunicorn to run the app
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]