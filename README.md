# ld-toggle

A CLI utility for toggling LaunchDarkly feature flags for specific users/contexts.

## Features

- Interactive fzf-based interface for browsing and toggling feature flags
- Individual user targeting (doesn't affect other users)
- Support for multiple environments (development, staging, production)
- Persistent configuration for project and user contexts
- Shows both current user-specific value and default value for each flag
- Only supports boolean feature flags

## Prerequisites

- Ruby (installed by default on macOS)
- [ldcli](https://github.com/launchdarkly/ldcli) - LaunchDarkly CLI tool
- [fzf](https://github.com/junegunn/fzf) - Fuzzy finder for terminal

### Dependencies

You'll need `fzf` and `ldcli` (the LaunchDarkly CLI client) installed

```bash
brew install fzf
brew install ldcli
```

## Installation

1. Login to LaunchDarkly
```bash
ldcli login
```

2. (Optional) Add to your PATH:

Create a symlink in your user bin directory:
```bash
mkdir -p ~/.local/bin
ln -s ~/code/ld-toggle/ld-toggle ~/.local/bin/ld-toggle
```

Then add `~/.local/bin` to your PATH in your shell profile (~/.zshrc, ~/.bashrc, etc.):
```bash
export PATH="$HOME/.local/bin:$PATH"
```

## Usage

### First Run

On first run, you'll be prompted to:
1. Enter your LaunchDarkly project key
2. Enter a user email to search for context

The tool will store this configuration in `~/.ld-toggle-config`.

The email search uses LaunchDarkly's `user.email` filter for efficient, server-side filtering.

### Basic Usage

```bash
# Use default environment (development)
ld-toggle

# Use staging environment
ld-toggle --env staging

# Use production environment
ld-toggle --env production
```

### Options

- `--env ENVIRONMENT` - Specify environment (development, staging, production). Default: development
- `--project PROJECT_KEY` - Override the default project key
- `--reconfigure` - Reconfigure the user context for the current environment
- `-h, --help` - Show help message

### Interactive Interface

The fzf interface shows:
- **User Status**: Current value for your context (ON/OFF/DEFAULT)
- **Default Value**: The flag's fallthrough/default value
- **Flag Key**: The technical key
- **Flag Name**: Human-readable name

Example display:
```
DEFAULT  | DEFAULT: true  | my-feature-flag - My Feature Flag
ON       | DEFAULT: false | another-flag - Another Feature Flag
OFF      | DEFAULT: true  | third-flag - Third Flag
```

### Toggling Flags

1. Use arrow keys to navigate through flags
2. A preview at the bottom shows what action will be taken
3. Press ENTER to toggle the selected flag immediately

### Toggle Behavior

- **Default → Override**: Adds individual targeting for your context
- **Override → Different Override**: Changes targeting to new value
- **Override → Default**: Removes individual targeting (resets to default)

## Configuration File

The tool stores configuration in `$XDG_CONFIG_HOME/ld-toggle/config.json` (or `~/.config/ld-toggle/config.json` if XDG_CONFIG_HOME is not set):

```json
{
  "project_key": "your-project-key",
  "contexts": {
    "development": {
      "key": "auth0|user-key-123",
      "kind": "user"
    },
    "staging": {
      "key": "auth0|user-key-456",
      "kind": "user"
    },
    "production": {
      "key": "auth0|user-key-789",
      "kind": "user"
    }
  }
}
```

You can manually edit this file or use `--reconfigure` to update.

## Examples

### Scenario 1: First time setup

```bash
$ ld-toggle
Enter LaunchDarkly project key: my-project
Enter user email to search: john@example.com
Searching for users with email: john@example.com...
Found user: user-123abc
Context configured: user-123abc for development
Loading feature flags...
[fzf interface appears]
```

### Scenario 2: Switching environments

```bash
$ ld-toggle --env production
Enter user email to search: john@example.com
[... context setup for production ...]
```

### Scenario 3: Reconfiguring context

```bash
$ ld-toggle --reconfigure
Enter user email to search: jane@example.com
[... new context setup ...]
```

### Scenario 4: Using different project

```bash
$ ld-toggle --project another-project --env staging
[... works with different project ...]
```

## How It Works

1. **Configuration**: Stores project key and user context per environment
2. **Flag Listing**: Uses `ldcli flags list` to fetch all boolean flags
3. **Flag Evaluation**: Uses `ldcli flags get` to check individual targeting
4. **Display**: Shows both user-specific value and default in fzf
5. **Toggling**: Uses `ldcli flags update` with `addTargets`/`removeTargets` instructions

## Troubleshooting

### "ldcli not found"

Make sure ldcli is installed and in your PATH:
```bash
which ldcli
ldcli --version
```

### "fzf not found"

Make sure fzf is installed:
```bash
which fzf
```

### "No users found"

The email search uses exact match. Make sure:
- The email is correct
- The user exists in the specified environment
- You have access to view contexts in that environment

### Authentication issues

Make sure ldcli is configured with a valid access token:
```bash
ldcli config --list
```

### Permission errors

Make sure your API token has the following permissions:
- Read flags
- Update flags
- Read contexts

## Limitations

- Only supports boolean feature flags (flags with true/false variations)
- Only supports user/context targeting (not rules or segments)
- Requires ldcli to be configured with appropriate permissions

## Contributing

This is a simple wrapper script. Feel free to modify it for your needs!

## License

MIT
