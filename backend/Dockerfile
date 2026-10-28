FROM python:3.14.10-alpine

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

# Dependencies first so this layer stays cached until requirements.txt changes
COPY requirements.txt .
# pip/setuptools are not needed at runtime and vendor vulnerable libraries, so drop them
RUN pip install --no-cache-dir -r requirements.txt \
    && pip uninstall -y pip setuptools

COPY app.py .

# Run as an unprivileged user instead of root
RUN addgroup -S -g 10001 app \
    && adduser -S -D -H -u 10001 -G app app
USER app

EXPOSE 5000
CMD ["python", "app.py"]
