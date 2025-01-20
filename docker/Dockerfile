FROM python:3.10-slim AS base

# Install system dependencies
RUN apt-get update && \
    apt-get install --no-install-recommends -y build-essential git libportaudio2 pandoc && \
    rm -rf /var/lib/apt/lists/*

# Create app user with UID 1000
RUN useradd -m -u 1000 -s /bin/bash appuser

WORKDIR /app

# Create virtual environment
RUN python -m venv /venv
ENV PATH="/venv/bin:$PATH"

# Playwright browser settings
ENV PLAYWRIGHT_BROWSERS_PATH=/home/appuser/pw-browsers
ENV PLAYWRIGHT_SKIP_BROWSER_GC=1

# Create directories with proper permissions
RUN mkdir -p /home/appuser/.aider /home/appuser/.cache /home/appuser/pw-browsers && \
    chown -R appuser:appuser /home/appuser /app /venv

# So git doesn't complain about unusual permissions
RUN git config --system --add safe.directory /app

#########################
FROM base AS deps-full

# Install core dependencies without the actual package
RUN /venv/bin/python -m pip install --upgrade --no-cache-dir pip && \
    /venv/bin/python -m pip install --no-cache-dir \
        'playwright>=1.39' \
        'torch' \
        'transformers' \
        --extra-index-url https://download.pytorch.org/whl/cpu

# Install playwright browsers
RUN /venv/bin/python -m playwright install --with-deps chromium

# Fix site-packages permissions
RUN find /venv/lib/python3.10/site-packages \( -type d -exec chmod a+rwx {} + \) -o \( -type f -exec chmod a+rw {} + \)

#########################
FROM deps-full AS aider-full

ENV AIDER_DOCKER_IMAGE=paulgauthier/aider-full

# Now copy the actual code and install it
COPY . /tmp/aider/
RUN /venv/bin/python -m pip install --no-cache-dir /tmp/aider[help,browser,playwright] && \
    rm -rf /tmp/aider

# Switch to appuser
USER appuser

ENTRYPOINT ["/venv/bin/aider"]

#########################
FROM base AS deps-min

# Install minimal dependencies without the actual package
RUN /venv/bin/python -m pip install --upgrade --no-cache-dir pip && \
    /venv/bin/python -m pip install --no-cache-dir \
        'playwright>=1.39' \
        --extra-index-url https://download.pytorch.org/whl/cpu

# Install playwright browsers
RUN /venv/bin/python -m playwright install --with-deps chromium

# Fix site-packages permissions
RUN find /venv/lib/python3.10/site-packages \( -type d -exec chmod a+rwx {} + \) -o \( -type f -exec chmod a+rw {} + \)

#########################
FROM deps-min AS aider

ENV AIDER_DOCKER_IMAGE=paulgauthier/aider

# Now copy the actual code and install it
COPY . /tmp/aider/
RUN /venv/bin/python -m pip install --no-cache-dir /tmp/aider[playwright] && \
    rm -rf /tmp/aider

# Switch to appuser
USER appuser

ENTRYPOINT ["/venv/bin/aider"]
