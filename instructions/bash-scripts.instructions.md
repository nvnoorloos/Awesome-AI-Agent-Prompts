## Shell Scripts Development Instructions

Instructions for generating high-quality robust task scripts with consistent structure, defensive validation, and rich logging. Every new or modified script must conform to these conventions so that higher-level tooling can rely on predictable behaviour.

## Setup Commands

- Ensure scripts run under Bash specifically: `bash --version` should be available on the target host.
- Preferred supporting tools include `curl`, `jq`, `mktemp`, and other GNU core utilities; declare any additional tools your script needs via `validate_required_tools`.

## Development Workflow

- Start each script with:
  ```bash
  #!/bin/bash
  set -euo pipefail
  ```
- Organise the file into the mandated sections:
  ```bash
  # =============================================================================
  # PUBLIC METHODS
  # =============================================================================
  # public entrypoints (document intent)
  
  # =============================================================================
  # PRIVATE HELPER METHODS
  # =============================================================================
  # underscore-prefixed helpers grouped by the public function they support
  
  # =============================================================================
  # MAIN EXECUTION
  # =============================================================================
  check_requirements
  execute_action
  ```
- `check_requirements` must validate both environment variables (via `validate_required_variables`) and external commands (via `validate_required_tools`) before execution.
- Keep main entrypoints lightweight: delegate detailed work to private helpers for readability and testability.
- When adding API interactions, wrap `curl` calls with error handling, capture HTTP status codes via `--write-out "\n%{http_code}"`, and abort on non-2xx responses.

## Testing Instructions

- Run scripts in a safe environment after wiring up the required environment variables; validation should fail fast when something is missing.
- Inspect DEBUG-level logs to confirm that intermediate values are correct when diagnosing issues.
- Prefer small, focused helper functions so they can be exercised independently with targeted inputs.

## Code Style Guidelines

- **Variables**
    - Use `UPPER_CASE` for exported/environment variables; use `lower_case` for local, function-scoped values.
    - Quote every expansion (`"${VAR}"`) to avoid unintended globbing and word splitting.
- **Functions**
    - Use `snake_case` names; prefix private helpers with `_`.
    - Provide concise comments for non-obvious logic and document function intent above public entrypoints.
- **Logging**
    - Always emit output through a log function like `log_message "<LEVEL>" "<message>"` (`SUCCESS`, `ERROR`, `STEP`, `WARNING`, `INFO`, `DEBUG`).
    - Never fall back to `echo` for user-facing logs.
- **Error Handling**
    - Check exit codes explicitly where failure needs custom handling (`if ! command; then ... fi`).
    - Return meaningful exit codes (`1` for general failures, `127` for missing requirements, etc.).
- **File Management**
    - Use `mktemp` for transient files and register a trap to clean them up.
    - Keep scripts in kebab-case (`my-task-action.sh`) and mark them executable.

## Build and Deployment

- Scripts are designed to be run directly without a build step. Keep them portable so they can execute both standalone and when orchestrated by higher-level automation.
- When scripts are invoked by other services, rely on environment variables (never hard-coded paths) to remain deployable across hosts.

## Security Considerations

- Treat credentials and tokens as sensitive: inject them through environment variables and avoid logging their values at any level.
- If a script shells out to third-party services, ensure the request payload and logs do not leak secrets.
- Review any hard-coded device paths or mount points before deployment to prevent destructive operations on unintended targets.

## Debugging & Troubleshooting

- Examine structured logs first; they include level, message, and any supplemental context added by the script.
- Confirm requirements by re-running `check_requirements` manually—missing tools or variables surface clear error messages.
- Inspect temporary file handling when diagnosing race conditions or stale artefacts; traps should remove files automatically.
- For API failures, capture and log the HTTP status and response body at DEBUG level to aid diagnosis while keeping secrets redacted.
