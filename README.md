# Local agent development environment

This Compose setup runs Edvabe plus a private MediaWiki MCP endpoint for local
E2B sandboxes.

## Configure the wiki

```bash
cp .env.example .env
```

Edit `.env` with the wiki base URL, article and script paths, and a dedicated
least-privilege BotPassword. `.env` is git-ignored. The BotPassword is mounted
only into the host-side MCP service; it is not sent to Edvabe sandboxes.
If the wiki's `MW_SERVER` resolves to a private address, set
`MCP_TRUSTED_HOSTS` to that exact wiki hostname so the MCP server's outbound
SSRF guard permits it.

## Start the services

```bash
/usr/local/bin/proxy.sh docker compose up -d
docker compose logs -f mediawiki-mcp
```

The MCP endpoint is bound to the Docker bridge gateway (default `172.17.0.1`) on
port `18080`, so sibling Edvabe sandbox containers can reach it without exposing
it on the host's LAN interfaces:

```text
http://172.17.0.1:18080/mcp
```

Add that URL as an HTTP MCP server in the agent running inside the sandbox. The
ProfessionalWiki server provides MediaWiki page/search tools and, when the wiki
has the NeoWiki extension, its NeoWiki tools as well.

For Qwen Code, add an entry like this to its `settings.json` in the sandbox:

```json
{
  "mcpServers": {
    "private-wiki": {
      "httpUrl": "http://172.17.0.1:18080/mcp",
      "includeTools": [
        "search-page",
        "get-page",
        "get-pages",
        "get-category-members",
        "parse-wikitext",
        "compare-pages",
        "update-page",
        "neowiki-list-schemas",
        "neowiki-get-schema",
        "neowiki-search-subjects",
        "neowiki-get-subject",
        "neowiki-validate-subject",
        "neowiki-update-subject"
      ]
    }
  }
}
```

The `includeTools` list exposes page edits and NeoWiki subject updates but omits
delete, move, and upload tools. Remove any NeoWiki names that your installation
does not provide.

This is a trusted, single-user development endpoint using a shared bot identity.
Keep it on the Docker bridge; do not publish it to the LAN or the public
internet. For multi-user access, configure MediaWiki OAuth and the MCP server's
hosted OAuth mode instead of sharing a BotPassword.
