# Install Docker
```
https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository
```
# Add user to docker group
```
https://docs.docker.com/engine/install/linux-postinstall
```

# Create the Directory structure
```
mkdir seedbox
cd seedbox

mkdir -p npm/data npm/letsencrypt plex/config qbittorrent/config media/downloads/completed
chown -R $USER:$USER npm plex qbittorrent media
```
#Create the docker compose
```
cd ~/seedbox/
wget https://raw.githubusercontent.com/xyberviri/Plex-Seedbox.Docker/refs/heads/main/docker-compose.yml
```

# Pull Docker Images
```
#pull docker images
docker compose pull
```
# Generate a Plex Claim Code
```
https://account.plex.tv/claim
```
`NOTE: you have 4 minutes to bring the box up after this`

Edit the docker config and replace `YOUR_CLAIM_CODE_HERE` with the code you generated in the previous step

# Bring the server up
```
docker compose up -d
```


# set nginx proxy manager password:
```
http://<your-server-ip>:81
```
If you have to enter a password the follow is the default for nginx:

Email: `admin@example.com`
Password: `changeisgood`

# Create the DNS Records for nginx, qbtorrent and plex
Add an A Record: In your DNS provider's dashboard for yourdomain.com, create an A record pointing to your server's IP address for nginx, qbtorrent and plex.

For the purpose of this guide it will be assumed those domains are `nginx.<mydomain>`, `qb.<mydomain>` and `plex.<mydomain>`

# Configure the Nginx Proxy Host in Nginx Proxy Manager
1. Navigate to Proxy Hosts: Click Hosts -> Proxy Hosts -> Add Proxy Host.
2. Details Tab:  
    • Domain Names: Enter `nginx.<mydomain>` and press Enter.  
    • Scheme: Select http.  
    • Forward Hostname / IP: Enter your server's IP address (or 127.0.0.1 / nginx-proxy-manager since it is routing to itself inside Docker).  
    • Forward Port: Enter 81.  
    • Options: Toggle on Block Common Exploits and Websockets Support.  
3. SSL Tab:  
    • SSL Certificate: Select Request a new SSL Certificate.  
    • Security Settings: Toggle on Force SSL and HTTP/2 Support.  
    • Terms: Enter your email address and agree to the Let's Encrypt Terms of Service.  
4. Save:  
    • Click Save to generate the certificate and activate the route.  
5. Secure Your Server Firewall (Recommended)  
    • Verify Access: Open a new browser tab and confirm you can log in via https://`nginx.<mydomain>`.  
    • Close Port 81: Once confirmed, block external traffic to port 81 in your server's firewall.  
   ```
   sudo ufw delete allow 81
   ```
# Set Up the qBittorrent Proxy Host & SSL
1. Go to the Hosts tab and click Proxy Hosts, then select Add Proxy Host.  
2. Details Tab:  
 	• Domain Names: Type your subdomain `qb.<mydomain>` and press Enter.  
 	• Scheme: Select http.  
 	• Forward Name/IP: Enter your server's local IP address (do not use localhost or 127.0.0.1, use the actual physical interface IP like 192.168.1.X or your public seedbox IP).  
 	• Forward Port: 8443 (the port specified for qBittorrent in your compose file).  
 	• Options: Toggle on Block Common Exploits.  
3. SSL Tab:
 	• SSL Certificate: Select Request a new SSL Certificate.  
 	• Toggle on Force SSL and HTTP/2 Support.  
 	• Enter your email address for the Let's Encrypt registration and agree to the Terms of Service.  
4. Click Save.
    • Nginx Proxy Manager will communicate with Let's Encrypt, generate your certificate, and activate the secure proxy.  

# Set Up the Plex Proxy Host & SSL
1. Click Add Proxy Host.
2. Details Tab:
	• Domain Names: Enter your Plex domain `plex.<mydomain>`.  
	• Scheme: Select http.  
	• Forward Name/IP: Enter your server's IP address.  
	• Forward Port: 32400 (the standard Plex media interface port).  
	• Options: Toggle on Block Common Exploits AND Websockets Support. Websockets support is required for Plex to properly handle real-time playback states.  
3. SSL Tab:
	• SSL Certificate: Select Request a new SSL Certificate.  
	• Toggle on Force SSL and HTTP/2 Support.  
	• Agree to the Let's Encrypt terms.  
4. Click Save.
    • Nginx Proxy Manager will communicate with Let's Encrypt, generate your certificate, and activate the secure proxy.  

# setup qbittorrent
1. get the default password
    • From your host get type `docker logs qbittorrent`  
2. login with "admin" username and above password:
    • Browse to qbittorrent `http://<your-server-ip>:8443` or https://`qb.<mydomain>`  
3. Click on Tools > Options (or click the Gear icon).
4. Select the Web UI tab from the left menu.
5. Scroll down to the Authentication section.
6. Type your new password into the Password field.
7. Configure Default Save Paths  
    • Under the Saving Management section, set the following parameters:  
    • Default Torrent Management Mode: Select Automatic. This allows qBittorrent to dynamically move files when categories or completion states change.  
    • When Torrent Category changed: Select Relocate torrent.  
    • When Default Save Path changed: Select Relocate affected torrents.  
    • Default Save Path: Set this to `/data/completed`.  
    • Keep incomplete torrents in: Check this box and set the path to `/data/incomplete`.
8. Scroll to the bottom and click Save.
9. Set Up Media Categories (Optional, Recommended for Organization)  
    • If you want to separate Movies and TV Shows so Plex scans them cleanly into distinct libraries:
    • In the main qBittorrent window, right-click the Categories section on the left sidebar and select Add Category.
    • Create a category named movies and set its Save Path to `/data/completed/movies`.
    • Create a second category named tv and set its Save Path to `/data/completed/tv`.
    Note: When adding a torrent, simply assign it to the movies or tv category.
    qBittorrent will download it to `/data/incomplete` and automatically move it to `/data/completed/movies` (or `tv`) upon completion while continuing to seed.

# Finalize the Plex Web Config
Now that your secure proxy is handling incoming connections on port 443, you must tell Plex to accept traffic from that address.  
1. Navigate to your new custom URL: https://`plex.<mydomain>`.
2. Go to Settings -> Network (ensure Show Advanced is turned on).
3. Find the Custom server access URLs box and enter your full proxy URL:text
    • https://`plex.<mydomain>`  
4. Scroll to the bottom and click Save Changes.

# Point Plex to the Completed Folders
1. Open your Plex web interface https://`plex.<mydomain>`.
2. Go to Settings -> Manage -> Libraries and click Add Library.
3. Choose Movies (or TV Shows), name it, and click Next.
4. Click Browse for Media Folder and navigate to `/data/media` (`/data/media/tv` or `/data/media/movies` if you used categories).
5. Click Add and then Add Library.

# Enable Automatic Scanning in Plex
1. Open Library Settings: Click the Settings wrench icon in the top right corner, scroll down to the Settings section on the left sidebar, and select Library.
2. Scan my library automatically: Check this box so Plex listens for Linux filesystem events (inotify) when qBittorrent moves a completed file into /data/media.
3. Run a partial scan when changes are detected: Check this box so Plex only scans the specific folder that changed rather than re-indexing your entire collection.
4. Scan my library periodically: Check this box and set the Library scan interval to Hourly or Every 2 hours as a reliable backup in case a filesystem notification is ever missed.
5. Empty trash automatically after every scan: Keep this unchecked for a seedbox setup so Plex does not delete your metadata and watch history if a mount point temporarily hiccups.
6. Save Changes: Scroll to the bottom of the page and click Save Changes.

# Verify Linux inotify Limits (Important for Large Libraries)  
Linux defaults to a low limit on how many folders an application can watch simultaneously for automatic updates. If you plan to store thousands of media folders on your seedbox, increase this limit on your host server:  
    • Check current limit: Run `cat /proc/sys/fs/inotify/max_user_watches` in your server terminal to see your current ceiling.
    • Increase the limit: Run `echo "fs.inotify.max_user_watches=524288" | sudo tee -a /etc/sysctl.conf && sudo sysctl -p` to raise the limit so Plex never stops detecting new downloads.

# Private Tracker changes
1. Click on the cog wheel,
2. go to the bittorrent tab
3. disable prohibited settings
    • DHT  
    • Peer Exchange  
    • Local Peer Discovery  
    • anonymous mode

# Trouble Shooting
## Locked Out of qbittorrent
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
