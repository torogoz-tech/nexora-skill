# Authoring a Nexora skill

Keep each skill focused on one workflow. Write for a person who must understand the boundary before allowing an agent to act.

Every skill should describe:

- prerequisites and required context;
- allowed inputs and trusted sources;
- the exact observable outputs;
- failure and uncertainty handling;
- reversible versus irreversible actions;
- the point where a human must approve or take over;
- verification commands or checks;
- the final evidence report.

Do not embed credentials, private endpoints, hidden approvals, or assumptions about a user's authorization. Do not turn a proposed action into an automatic action merely because a tool is available.

Use examples that distinguish the local static-analysis engine, the authenticated platform API, the bilingual web interface, and remote MCP operations. They are related Nexora boundaries, not interchangeable capabilities.
