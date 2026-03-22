# Your Guide To Creating MCP Connections

This document provides the rules you should follow when establishing new MCP connections for the user. 

Use this to create the connections in the most logical way.

## Gold Standard: Lazy Loading, User Level 

The user does not work on collaborative projects involving MCPs. You can assume that all connections you are provisioning will not be shared. Don't complicate the decision by assuming a federated authentication model. There is none. 

The most useful way for the user to use MCPs is to always have the full array of MCPs available on-demand regardless of context. This offloads the burden of having to maintain repo/project-level MCP definitions. 

In the event that a project level MCP *is* needed, MCP configurations are additive: so create a .mcp.json in the repo. But assume - unless it's obviously not the case - that any MCP the user wishes to create is in the category of "generally useful to have" tools. These should be available as user-level tools. 

## Order Of Preferenence: Claude.ai (Server-Side), Self-Hosted Gateway, Local 

Anthropic supports creating MCP connections via Claude.ai. These are created server-side (by Anthropic) and inherited by Claude Code. They are lazy-loaded to prevent context-bloat.

For the user, this is the most ideal type of MCP connection that can be configured as the connections are persistently available across contexts. 

### When Claude.ai connectors Are Not Desired

1: Multi-Context Is Needed

However, there are some caveats. Or rather, there are situations where this setup still creates friction. 

The main point of friction regardless context-shifting. 

The user uses Claude Code for both personal use and for work (the user is a sefl-employed consultant). The user commonly wishes to use the same MCP tools (say, Google Drive) but to use an MCP server that has that context specific configuration.

E.g.:

- When working on a client project, use the Gmail MCP with the business Workspace so that client emails sent with the MCP are sent from the business address 
- When working on a personal project, use the Gmail MCP with the personal workspace to send a newsletter to friends and family.

The user has multiple Claude accounts but doesn't like switching back and forth all day. 

Claude does not support multiple account configurations at the cloud connector level right now. So this is an edge case in which it's better to either:

1) Create two local MCPs 
2) Create one primary connection in Claude.ai and another local

Note that custom MCP connections can be created using Claude.ai.

2: Custom MCPs

The user often creates custom MCP servers to provide MCP tools that are specific to their use case, or which are scoped precisely to minimize context bloat. As an example, the user created a fal.ai MCP server, which has hardcoded preferences for the user's preferred AI models (NPM fal-nano-mcp).

For ease of installation as well as for open-sourcing the projects, these are published as NPM packages. These can be installed in a managed environment, but not as Claude.ai connectors unless they are provided as SSEs, which requires hosting.

Because Claude.ai is a SaaS platform, for custom-created MCPs that the user requires for work, the preference is that they should be hosted on Meta-MCP unless they require direct file system exposure, in which case they should be created on localhost or environment-bound MCP aggregators.

## Second And Third Tier Preferences: Server-Side Local

In order to streamline the management and exposure of sets of MCP servers, the user uses meta-MCP as an aggregator. The utility of the user having a self-managed MCP aggregator in a managed environment is that these custom MCPs can be created there. The one advantage of doing this over using Claude connectors is that provisioning MCP connections here makes them tool-agnostic.

At a later point in time the user may go back to this model, but at the moment, and while the user uses Claude for most agentic AI work, the convenience of being able to manage preconfigured MCPs outweighs the less flexible architecture. 

So the utility of the self managed MetaMCP instance is currently limited to:

- Having an environment on which to deploy custom MCPs
- Having an environment to aggregate MCPs and which is not bound to any AI agent

Although the situation is fast-changing and is likely to evolve soon, at the current point in time, it is challenging to use MCP tools which require reading from binary files in the local environment.

As an example, consider a transcription MCP which would use an audio file stored on the local environment and send it to a Google API for transcription.  

Contrast this with generative AI MCPs, in which the user may provide instructions, the AI API may provide an authenticated download link, and the user can download this directly on their local machine. In this case, no local *access* is inherently required.

There are mechanisms for using an MCP deployed remotely or on a server and providing local files to the MCP, which proxies them to the API. 

However, at the moment, this requires complicated tooling and setup. So, for the sake of simplicity and reliability until mechanisms for this are more mature, where MCPs require local file exposure, the user creates these on a localhost version of the MCP. The negative of this approach is that a localhost MCP needs to be created in every working environment; i.e., the portability of remotely managed MCPs, whether through Claude or through a self-managed intermediary, is lost.