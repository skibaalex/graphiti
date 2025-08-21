# syntax=docker/dockerfile:1.9
FROM python:3.12-slim

WORKDIR /app

# Install system dependencies
RUN printf 'Acquire::http::Proxy "false";\nAcquire::https::Proxy "false";\n' > /etc/apt/apt.conf.d/99no-proxy \
    && apt-get update && apt-get install -y --no-install-recommends \
    curl \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Install uv using the installer script
ADD https://astral.sh/uv/install.sh /uv-installer.sh
RUN sh /uv-installer.sh && rm /uv-installer.sh

# Add uv to PATH
ENV PATH="/root/.local/bin:${PATH}"

# Configure uv for optimal Docker usage
ENV UV_COMPILE_BYTECODE=1 \
    UV_LINK_MODE=copy \
    UV_PYTHON_DOWNLOADS=never \
    MCP_SERVER_HOST="0.0.0.0" \
    PYTHONUNBUFFERED=1

# Ensure proxy environment variables are empty to avoid apt/curl proxy issues
ENV http_proxy="" \
    https_proxy="" \
    HTTP_PROXY="" \
    HTTPS_PROXY=""

# Create non-root user
RUN groupadd -r app && useradd -r -d /app -g app app

# Copy project files for dependency installation (better caching)
COPY pyproject.toml uv.lock ./

# Install dependencies first (better layer caching)
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-dev

# Copy application code
COPY graphiti_mcp_server.py ./

# Change ownership to app user
RUN chown -Rv app:app /app

# Switch to non-root user
USER app

# Expose port
EXPOSE 8000

# Command to run the application
CMD ["uv", "run", "graphiti_mcp_server.py"]
