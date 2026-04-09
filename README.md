Since we aren't using the host machine, we run the tunnel inside Ubuntu.
Install in VM(for Mac):
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64.deb
sudo dpkg -i cloudflared.deb
Verify the Installation
Once that finishes, check if it's working by typing:
Bash:

cloudflared --version

2. Running the tunnel:
cloudflared tunnel --url http://localhost:3001.   (In place of ‘localhost’, IP Address can also be used.)
3. Result: Cloudflare gives you a URL (e.g., https://my-secure-vault.trycloudflare.com).
