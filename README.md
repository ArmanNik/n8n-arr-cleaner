> [!WARNING]  
> I vibe coded this while doom-scrolling youtube so no promises. If it nukes all your "ubuntu distros" don't blame me 🙈


# n8n arr Cleaner Workflow: Auto-Blocklist Torrents for Radarr/Sonarr

This [n8n](https://docs.n8n.io/hosting/) workflow automates the process of finding and removing torrents with undesirable file extensions from qBittorrent. It then blocklists the release in the originating 'arr' application (Radarr or Sonarr) to prevent it from being downloaded again.

<img width="2150" height="596" alt="image" src="https://github.com/user-attachments/assets/acd3d853-03bf-4263-bc12-20af5e9e2ead" />


## How It Works

The workflow executes the following logic on a schedule:

1.  **Trigger:** Runs at a customizable interval (default is every 3 hours).
2.  **Scan:** Connects to qBittorrent, fetches the list of active torrents, and inspects the file extensions within each one.
3.  **Filter:** Checks if any file matches a defined list of `UNSAFE_EXTENSION`s (e.g., `.rar`, `.exe`, `.txt`).
4.  **Identify:** If a match is found, it uses the torrent's category to determine if it belongs to Radarr or Sonarr.
5.  **Action:**
    *   Calls the appropriate Radarr/Sonarr API to remove the item from the queue and blocklist the release.
    *   Sends a notification (optional) detailing the action taken.

## Setup Instructions

### 1. Configure the "ENV" Node

All user-configurable settings are located in the first **"ENV" (Set)** node. Open this node to edit the following values.

*(Note: If n8n is running in a Docker container, use your host machine's IP address, not `localhost`.)*

| Variable               | Description                                           |
| ---------------------- | ----------------------------------------------------- |
| `LOCALHOST`            | The IP address of the machine running your services.  |
| `SONARR_PORT`          | Port for your Sonarr instance.                        |
| `SONARR_API_KEY`       | Your Sonarr API Key (`Settings > General`).           |
| `SONARR_CATEGORY_NAME` | The category Sonarr uses in qBittorrent.              |
| `RADARR_PORT`          | Port for your Radarr instance.                        |
| `RADARR_API_KEY`       | Your Radarr API Key (`Settings > General`).           |
| `RADARR_CATEGORY_NAME` | The category Radarr uses in qBittorrent.              |
| `QBITTORRENT_PORT`     | Port for the qBittorrent Web UI.                      |
| `QBITTORRENT_USERNAME` | Your qBittorrent Web UI username.                     |
| `QBITTORRENT_PASSWORD` | Your qBittorrent Web UI password.                     |


> [!CAUTION]
> This implementation uses http so is not particulary secure. Usenames, passwords, and keys are sent in plain text. Only use this in a safe local network. It might requre more changes before it works with https.


### 2. Define Unsafe Extensions

Within the same **ENV** node, locate the `UNSAFE_EXTENSION` variable. Edit the JavaScript array to add or remove file extensions:

```javascript
// Example
['.lnk', '.rar', '.exe', '.zipx', '.scr', '.txt']
```

> [!NOTE]  
> Any torrent containning a "bad" file format will be removed regardless of the other files in the torrent. To avoid false positives be conservative with what you exclude.

### 3. Configure Notifications

The workflow uses Gotify by default.

1.  In the n8n sidebar, go to **Credentials** and add a new `Gotify` credential with your server URL and app token.
2.  Return to the workflow and select this credential in the **"Send Gotify Alert"** node.
3.  To use a different notification service, delete the Gotify node and replace it with your preferred service's node.



### 4. Activate the Workflow

After saving your configuration, enable the workflow by using the **"Active"** toggle in the top-right corner.



