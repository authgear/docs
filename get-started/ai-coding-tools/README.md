# Skills for AI Agent

[Authgear Skills](https://github.com/authgear/authgear-skills) are installable skills for AI coding agents (Cursor, Claude Code, Codex, OpenCode, Gemini CLI, and similar) to integrate Authgear into your projects.

To install all Authgear Skills, in your project use:

```shellscript
npx skills add authgear/skills
```

Or clone the repo and point the skills path of your coding agent to the `skills` folder.

### Available skills

Once installed, these skills will be available for your project.

<table><thead><tr><th width="221.4140625">Skill</th><th>When to use</th></tr></thead><tbody><tr><td>/<strong>authgear-integration</strong> </td><td>Integrate Authgear authentication into web, mobile, and backend applications</td></tr><tr><td><strong>/authgear-custom-ui</strong> (coming soon)</td><td>Build <a href="../../customization/ui-customization/custom-ui/">custom UI</a> for signup/login</td></tr><tr><td><strong>/authgear-authflow</strong> (coming soon)</td><td>Set up <a href="../../customization/ui-customization/custom-ui/authentication-flow-api.md">custom authentication flow</a></td></tr></tbody></table>

### Example prompts

You can try our these example prompts in your project:

* "Add Authgear authentication to my React app with login/logout buttons and a protected dashboard page"
* "Integrate Authgear into my React Native app with login flow and user setting screen"
* "Protect my Express.js API endpoints with JWT token validation using Authgear"
* "Create a user profile page that displays email and phone number from Authgear"
