# ld-toggle

A CLI utility for toggling LaunchDarkly feature flags for specific users/contexts.

## Features

- Interactive fzf-based interface for browsing and toggling feature flags
- Individual user and driver targeting (doesn't affect other users/drivers)
- Support for multiple environments (development, staging, production)
- Persistent configuration for project and user/driver contexts
- Shows both current context-specific value and default value for each flag
- Automatic UUID to ULID conversion for driver contexts
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
- `--reconfigure` - Reconfigure the user/driver context for the current environment
- `-d, --driver [KEY]` - Toggle flags for a driver context. Optionally provide driver UUID/ULID
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
  },
  "driver_contexts": {
    "development": {
      "key": "01HZQK7XVZQD6XGZQK7XVZQK7X",
      "kind": "driver"
    },
    "staging": {
      "key": "01HZQK8YVZQD6XGZQK8YVZQK8Y",
      "kind": "driver"
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

### Scenario 5: Toggling driver context (interactive)

```bash
$ ld-toggle --driver
Enter driver UUID or ULID: 550e8400-e29b-41d4-a716-446655440000
Driver context found!
Driver context configured: 2N1T201RMV87AAE5J4CSAM8000 (kind: driver) for development
Loading feature flags...
[fzf interface appears]
```

### Scenario 6: Toggling driver context (with key)

```bash
# Using UUID (automatically converted to ULID)
$ ld-toggle --driver 550e8400-e29b-41d4-a716-446655440000

# Using ULID directly
$ ld-toggle --driver 01HZQK7XVZQD6XGZQK7XVZQK7X

# With different environment
$ ld-toggle --driver 01HZQK8YVZQD6XGZQK8YVZQK8Y --env staging
```

### Scenario 7: Reconfiguring driver context

```bash
$ ld-toggle --driver --reconfigure
Enter driver UUID or ULID: [new driver UUID]
[... new driver context setup ...]
```

## How It Works

1. **Configuration**: Stores project key and user/driver contexts per environment
2. **UUID/ULID Conversion**: Automatically converts driver UUIDs to ULIDs using Crockford Base32 encoding
3. **Flag Listing**: Uses `ldcli flags list` to fetch all boolean flags
4. **Flag Evaluation**: Uses `ldcli flags get` to check individual targeting
5. **Display**: Shows both context-specific value and default in fzf
6. **Toggling**: Uses `ldcli flags update` with `addTargets`/`removeTargets` instructions

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

### Driver context not found

If you get an error when configuring a driver context:
- Verify the driver UUID/ULID is correct
- Ensure the driver context exists in the specified environment
- Check that you have permissions to view driver contexts

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

## Driver Context Support

The tool supports both user and driver contexts:

- **User contexts**: Configured via email search or direct key entry
- **Driver contexts**: Configured via UUID/ULID entry
- **UUID to ULID conversion**: Automatically converts driver UUIDs to ULIDs for compatibility

Driver IDs are stored internally as UUIDs but presented as ULIDs. The tool handles this conversion automatically using Crockford Base32 encoding, matching the format used in the LaunchDarkly application.

## Limitations

- Only supports boolean feature flags (flags with true/false variations)
- Only supports user/driver context targeting (not rules or segments)
- Requires ldcli to be configured with appropriate permissions

## Contributing

This is a simple wrapper script. Feel free to modify it for your needs!

## License

MIT
