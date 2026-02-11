
# Add server (stdio)
claude mcp add <name> --env KEY=val -- <command> [args...]

# Add server (HTTP)
claude mcp add --transport http <name> <url>
claude mcp add --transport http <name> <url> --header "Authorization: Bearer TOKEN"

# Add JSON config (v2.1.1+)
claude mcp add-json <name> '{"type":"http","url":"...","headers":{...}}'

# Scope flags
--scope user      # ~/.claude.json (default for user-wide)
--scope project   # .mcp.json (team-shared)
--scope local     # project-specific, not committed

# Other commands
claude mcp list
claude mcp remove <name>
claude mcp get <name>  # verify config
