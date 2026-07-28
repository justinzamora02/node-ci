# Agent guidance

- Keep actions and reusable workflows minimal and generally applicable.
- Do not add project-specific build behavior.
- Prefer native GitHub Actions features and official actions.
- Pin every external action to a verified full commit SHA and include a version
  comment.
- Apply least-privilege permissions.
- Never expose caller secrets or use implicit secret inheritance.
- Do not use `pull_request_target` for workflows that execute pull-request code.
- Never cache secrets, credentials, configuration containing tokens, or
  `node_modules`.
- Preserve backward compatibility within a major release.
- Document every new input, output, permission, and secret.
- Update examples whenever the action or a workflow interface changes.
- Validate action metadata, YAML, and workflow semantics before declaring work
  complete.
- Avoid adding inputs until a real caller needs the variation.
- Keep Docker, publishing, deployment, and application-release responsibilities
  in separate workflows.
- Do not move stable major tags except through the documented release process.
- Treat `action.yml` as the public interface. The validation and release
  workflows maintain this repository and must not expose `workflow_call`.
