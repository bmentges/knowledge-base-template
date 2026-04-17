# Credits

This plugin is an instantiation of the **LLM Wiki** pattern. It stands on two shoulders.

## Primary source

- **Andrej Karpathy — LLM Wiki idea file**
  https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
  Original statement of the pattern: three layers (raw, wiki, schema) and three operations (ingest, query, lint). See `origin/llm-wiki.md` in this plugin for a copy.

## Scaling extensions

- **agentmemory — persistent memory engine for AI agents**
  https://github.com/rohitg00/agentmemory
  Source of the lifecycle, graph, automation, and quality-control patterns layered on top of the original. See `origin/llm-wiki-v2.md` in this plugin for a copy.

## This plugin

Packaged by **Bruno Mentges** as a Claude Code plugin that turns the pattern into four invokable skills: `wiki-bootstrap`, `wiki-ingest`, `wiki-query`, `wiki-lint`.

## License

MIT. See `LICENSE` at the plugin root.
