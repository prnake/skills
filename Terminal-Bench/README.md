## Harbor

Compare to droid, model in claude code is lazy to load skills, so it's better to put the prompt in CLAUDE.md

```diff
diff --git a/src/harbor/agents/installed/claude_code.py b/src/harbor/agents/installed/claude_code.py
index 1f8c7e1..07f4fb0 100644
--- a/src/harbor/agents/installed/claude_code.py
+++ b/src/harbor/agents/installed/claude_code.py
@@ -1,3 +1,5 @@
+from __future__ import annotations
+
 import json
 import os
 import shlex
@@ -5,6 +7,7 @@ from pathlib import Path
 from typing import Any
 
 from harbor.agents.installed.base import BaseInstalledAgent, ExecInput
+from harbor.environments.base import BaseEnvironment
 from harbor.models.agent.context import AgentContext
 from harbor.models.agent.name import AgentName
 from harbor.models.trajectories import (
@@ -27,12 +30,14 @@ class ClaudeCode(BaseInstalledAgent):
         self,
         max_thinking_tokens: int | None = None,
         max_turns: int | None = None,
+        skills_dir: str | None = None,
         *args,
         **kwargs,
     ):
         super().__init__(*args, **kwargs)
         self._max_thinking_tokens = max_thinking_tokens
         self._max_turns = max_turns
+        self._skills_dir = Path(skills_dir) if skills_dir else None
 
     @staticmethod
     def name() -> str:
@@ -42,6 +47,21 @@ class ClaudeCode(BaseInstalledAgent):
     def _install_agent_template_path(self) -> Path:
         return Path(__file__).parent / "install-claude-code.sh.j2"
 
+    async def setup(self, environment: BaseEnvironment) -> None:
+        await super().setup(environment)
+
+        if self._skills_dir:
+            if not self._skills_dir.is_dir():
+                raise FileNotFoundError(
+                    f"Skills directory not found: {self._skills_dir}"
+                )
+
+            await environment.exec(command="mkdir -p /root/.claude/skills")
+            await environment.upload_dir(
+                source_dir=self._skills_dir,
+                target_dir="/root/.claude/skills",
+            )
+
     def _get_session_dir(self) -> Path | None:
         """Identify the Claude session directory containing the primary JSONL log"""
         sessions_root = self.logs_dir / "sessions"
@@ -825,9 +845,9 @@ class ClaudeCode(BaseInstalledAgent):
             if small_model_region:
                 env["ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION"] = small_model_region
 
-            # Optional: disable prompt caching (not available in all regions)
-            if os.environ.get("DISABLE_PROMPT_CACHING", "").strip() == "1":
-                env["DISABLE_PROMPT_CACHING"] = "1"
+        # Optional: disable prompt caching (not available in all regions)
+        if os.environ.get("DISABLE_PROMPT_CACHING", "").strip() == "1":
+            env["DISABLE_PROMPT_CACHING"] = "1"
 
         # Remove empty auth credentials to allow Claude CLI to prioritize the available method
         # When both are empty, Claude CLI will fail with a clear authentication error
@@ -862,6 +882,8 @@ class ClaudeCode(BaseInstalledAgent):
         # Disable non-essential traffic (telemetry, etc.)
         env["CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC"] = "1"
 
+        env["CLAUDE_CODE_ATTRIBUTION_HEADER"] = "0"
+
         # Allow bypassPermissions mode when running as root inside containers
         env["IS_SANDBOX"] = "1"
 
@@ -880,7 +902,11 @@ class ClaudeCode(BaseInstalledAgent):
             "$CLAUDE_CONFIG_DIR/todos && "
             "if [ -d ~/.claude/skills ]; then "
             "cp -r ~/.claude/skills $CLAUDE_CONFIG_DIR/skills 2>/dev/null || true; "
-            "fi"
+            "fi && "
+            "if [ -d ~/.claude/skills ]; then "
+            "for f in ~/.claude/skills/*/SKILL.md; do "
+            '[ -f "$f" ] && { echo ""; cat "$f"; echo ""; } >> /app/CLAUDE.md 2>/dev/null || true; '
+            "done; fi"
         )
 
         mcp_command = self._build_register_mcp_servers_command()
```
