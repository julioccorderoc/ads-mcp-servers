FROM openbridge/amazon-ads-mcp:latest
# Override CMD with shell form so Railway's $PORT env var is expanded at runtime.
CMD ["sh", "-c", "python -m amazon_ads_mcp.server --transport http --host 0.0.0.0 --port $PORT"]
