# Plex-Seedbox.Docker
Docker config for a Seedbox with Plex

Install Docker
```
https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository
```
Add user to docker group
```
https://docs.docker.com/engine/install/linux-postinstall
```

Create the Directory structure
```
mkdir -p ~/seedbox/config/qbittorrent
mkdir -p ~/seedbox/config/plex
mkdir -p ~/seedbox/data/torrents
mkdir -p ~/seedbox/data/media/movies
mkdir -p ~/seedbox/data/media/tv
```
Create the docker compose
```
cd ~/seedbox/
wget https://raw.githubusercontent.com/xyberviri/Plex-Seedbox.Docker/refs/heads/main/docker-compose.yml
```
Generate a Plex Claim Code
```
https://account.plex.tv/claim
```
NOTE: you have 4 minutes to bring the box up after this

Edit the docker config and replace `YOUR_CLAIM_CODE_HERE` with the code you generated in the previous step

Bring the server up
```
docker compose up -d
```

Accessing Plex
```
http://<your-server-ip>:32400/web
```

Get qBittorrent temp password
```
docker logs qbittorrent
```

Accessing qBittorrent temp
```
http://<your-server-ip>:8080
```

Change the qBittorrent password:
1. Click on Tools > Options (or click the Gear icon).
2. Select the Web UI tab from the left menu.
3. Scroll down to the Authentication section.
4. Type your new password into the Password field.
5. Scroll to the bottom and click Save.

# Trouble Shooting
## Locked Out
If You Are Locked Out of qBittorrent reset the password via Config File.

Stop the container
```
docker compose down
```

Edit the config file
```
nano ~/seedbox/config/qbittorrent/qBittorrent/config/qBittorrent.conf
```

Look under the `[Preferences]` section for the following lines and delete them completely:
	• WebUI\Password_Hashed
	• WebUI\Password_Salt

Restart the container
```
docker compose up -d
```

Get the temp admin password
```
docker logs qbittorrent
```


