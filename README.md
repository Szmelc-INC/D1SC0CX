# D1SC0CX
> ## Simple Discord client update script for Debian
![image](https://github.com/user-attachments/assets/d860ba0f-372e-4d3b-bf4b-d5c38ab34a78)

### [Disclaimer]
> This project exists only because some dumb fucks in Discord, \
> are too dumb, or too fucked to implement solid update feature.\
> So there you go. This project is pretty much finished as is.

> Setup:
```bash
# Get script, make executable and add to /bin
wget https://raw.githubusercontent.com/Szmelc-INC/D1SC0CX/refs/heads/main/discocx
sudo chmod +x discocx
sudo mv discocx /bin/discocx
```

> Entire source
```bash
#!/bin/bash

# <Error handling like a pro>
rm /tmp/discord*

# vars
export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:$PATH"
INFO_PATH="/usr/share/discord/resources/build_info.json"
BASE_URL="https://stable.dl2.discordapp.net/apps/linux"
USER=$(logname)
HOME_DIR=$(eval echo "~$USER")
START_VER="0.0.78"
  
# Get current version
get_ver() {
    [[ -f $INFO_PATH ]] && jq -r '.version' "$INFO_PATH" || echo ""
}
CUR_VER=$(get_ver)
echo "Current Discord version: ${CUR_VER:-$START_VER}"

# Parse version parts
major=$(echo "$CUR_VER" | cut -d '.' -f 1)
minor=$(echo "$CUR_VER" | cut -d '.' -f 2)
patch=$(echo "$CUR_VER" | cut -d '.' -f 3)
# Find latest version
LATEST=""
while true; do
    patch=$((patch + 1))
    NEXT="$major.$minor.$patch"
    URL="$BASE_URL/$NEXT/discord-$NEXT.deb"
    echo "Checking version $NEXT..."
    if wget --spider "$URL" 2>/dev/null; then
        echo "Found version $NEXT."
        LATEST="$NEXT"
    else
        echo "Version $NEXT not found. Stopping."
        break
    fi
done
# Download and install
if [[ -n "$LATEST" ]]; then
    echo "Latest version: $LATEST."
    DEB="/tmp/discord-$LATEST.deb"
    if ! touch "$DEB" 2>/dev/null; then
        echo "No write access to /tmp. Trying home directory."
        DEB="$HOME_DIR/discord-$LATEST.deb"
    fi
    echo "Downloading $LATEST..."
    URL="$BASE_URL/$LATEST/discord-$LATEST.deb"
    wget -O "$DEB" "$URL" && dpkg -i "$DEB" && rm -f "$DEB" && \
    echo "Discord updated to $LATEST."
else
    echo "No updates available. Current version: $CUR_VER."
fi

# Start Discord
echo "Starting Discord for $USER..."
su - "$USER" -c "sleep 2; DISPLAY=:0 nohup /usr/share/discord/Discord &"
```

![image](https://github.com/user-attachments/assets/4709ada0-2d75-47ee-8384-a9fa0b7b3100)
