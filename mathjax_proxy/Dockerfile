# Combined Dockerfile for Slobby + MathJax Proxy
# Runs both daemons in a single container

FROM debian:bookworm AS builder

# Copy Slobby source files
COPY Wikipedia/slob /opt/slob
COPY Wikipedia/slobby /opt/slobby

# Install build dependencies and compile Slobby + Slobby
RUN apt-get update && \
    apt-get install -y python3 python3-virtualenv libicu-dev pkg-config && \
    cd /opt/slob && \
    python3 -m virtualenv env && \
    . env/bin/activate && \
    pip install --no-cache-dir . && \
    cd ../slobby && \
    pip install --no-cache-dir . && \
    rm -rf /var/lib/apt/lists/*

# Stage 2: Create a minimal runtime image with both services
FROM debian:bookworm

# Install runtime dependencies
RUN apt-get update && \
    apt-get install -y python3 libicu72 python3-pip && \
    apt clean && rm -rf /var/lib/apt/lists/*

# Copy compiled Slobby from builder
COPY --from=builder /opt /opt

# Install MathJax proxy dependencies
COPY requirements.txt /app/requirements.txt
RUN pip3 install --no-cache-dir --break-system-packages -r /app/requirements.txt

# Copy MathJax proxy application files
COPY mathjax_proxy.py /app/
COPY filter_logic.py /app/

# Create startup script that launches both daemons
RUN cat <<'EOF' > /opt/start-services.sh
#!/bin/bash
set -e

echo "Starting Slobby server on port 8013..."
/opt/slob/env/bin/slobby -i 0.0.0.0 ${SLOBBY_FILE:-/opt/aard/enwiki.slob} &
SLOBBY_PID=$!

echo "Waiting for Slobby to be ready..."
sleep 2

echo "Starting MathJax proxy on port 8014..."
export TARGET_HOST=${TARGET_HOST:-127.0.0.1}
export TARGET_PORT=${TARGET_PORT:-8013}
cd /app && python3 mathjax_proxy.py &
PROXY_PID=$!

echo "Both services started:"
echo "  - Slobby:   http://127.0.0.1:8013"
echo "  - Proxy:    http://127.0.0.1:8014"
echo "Press Ctrl+C to stop."

# Wait for both processes
wait $SLOBBY_PID $PROXY_PID
EOF
RUN chmod +x /opt/start-services.sh

# Expose both ports
EXPOSE 8013 8014

# Volume for the Slobby database
VOLUME ["/opt/aard"]

# Environment variables
ENV TARGET_HOST=127.0.0.1
ENV TARGET_PORT=8013
ENV SLOBBY_FILE=/opt/aard/enwiki-20260401.slob

# Start both services
CMD ["/opt/start-services.sh"]
