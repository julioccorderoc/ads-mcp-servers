FROM openbridge/amazon-ads-mcp:latest
# Shell-form CMD so sh expands $PORT (injected by Railway) at container startup.
# Falls back to 9080 if PORT is not set.
CMD python -m amazon_ads_mcp.server --transport http --host 0.0.0.0 --port ${PORT:-9080}
