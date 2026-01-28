# Galaxy MCP Gotchas

Common pitfalls and solutions when using Galaxy MCP.

## Retrieving API Key from macOS Keychain

```bash
# Find keychain entry names
security dump-keychain | grep -i galaxy -A 5 -B 5

# Retrieve the password (use svce and acct values from above)
security find-generic-password -s "usegalaxy.org" -a "galaxy-api" -w
```

## Finding Histories by URL

Galaxy URLs use slugs that differ from actual history names:
- URL: `usegalaxy.org/u/user/h/my-analysis-run` -> slug is `my-analysis-run`
- Actual name: `My Analysis Run` (title case, spaces)

The `get_histories(name=...)` filter is case-sensitive. To find a history from a URL:
1. Use `list_history_ids()` to get all histories
2. Match case-insensitively, treating hyphens as spaces

## Empty History Contents

**Problem**: `get_history_contents` returns empty but history has datasets.

**Solution**: Default only shows visible, non-deleted datasets:
```
get_history_contents(
    history_id="...",
    deleted=true,
    visible=false
)
```

## Dataset ID vs HID

- `hid` = human-readable number shown in UI (e.g., 13437)
- `id` = hex hash used in API calls (e.g., "f9cad7b01a472135...")

All MCP functions use `id` (the hex hash), not `hid`.

## Tool ID Formats

Galaxy tool IDs can have multiple formats:
- Simple: `Cut1`, `cat1`
- ToolShed: `toolshed.g2.bx.psu.edu/repos/iuc/hyphy_fel/hyphy_fel/2.5.84+galaxy0`

Always use the full ToolShed format in workflows for reproducibility.

## Workflow Input Mapping

When invoking workflows, inputs use step indices:
```python
invoke_workflow(
    workflow_id="...",
    inputs={
        "0": {"id": "DATASET_ID", "src": "hda"},  # Step 0
        "1": {"id": "DATASET_ID2", "src": "hda"}  # Step 1
    },
    history_id="..."
)
```

`src` values:
- `hda` = HistoryDatasetAssociation (standard dataset)
- `hdca` = HistoryDatasetCollectionAssociation (collection)
- `ldda` = LibraryDatasetDatasetAssociation

## Connection Issues

```python
# Check connection
get_server_info()

# If fails, reconnect
connect(url="https://usegalaxy.org", api_key="YOUR_KEY")
```

## URL Trailing Slash

Galaxy URLs should end with `/`:
- Correct: `https://usegalaxy.org/`
- May fail: `https://usegalaxy.org`

## Large Histories

Don't request all datasets at once. Use pagination:
```python
# First 100
get_history_contents(history_id="...", limit=100, offset=0)

# Next 100
get_history_contents(history_id="...", limit=100, offset=100)
```

## Order Options

- `hid-asc` - oldest first (default)
- `hid-dsc` - newest first (usually what you want)
- `create_time-dsc` - most recently created
- `update_time-dsc` - most recently modified
- `name-asc` - alphabetical
