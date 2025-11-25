FROM python:3.12-slim
RUN apt-get update && apt-get install -y iptables && rm -rf /var/lib/apt/lists/*
WORKDIR /app
RUN pip install --no-cache-dir uv
COPY kv /app/kv
COPY pyproject.toml uv.lock /app/
RUN uv sync --frozen --no-dev
EXPOSE 8084 6004
ENTRYPOINT ["uv", "run", "python", "/app/kv/kvstorage_http.py"]
CMD ["8084", "node5:6004", "node2:6001", "node3:6002", "node1:6000", "node4:6003"]
