<h1><b>Remote FILE ACCESS SYSTEM</b></h1>
<hr>
<h2><b>Create VM in Multipass</b></h2>
<p>multipass launch &#45;name primary &#45;cpus 1 &#45;memory 2G &#45;disk 6G</p>
<br>
<h2><b>VM and Enviroment Setup</b></h2>
<h3>1. Enter the VM</h3>
<p>multipass shell primary</p>
<h3>2. Create the "Safe Zone" folder</h3>
<p>mkdir &#45;p &#126;&#47;remote&#45;vault</p>
<h3>3. Setup Node.js environment</h3>
<p>sudo apt update && sudo apt install -y nodejs npm<br>
 mkdir &#126;&#47;server &amp;&amp; cd &#126;&#47;server<br>
 npm init &#45;y<br>
 npm install express cors<br>
<br>
<p><b>Install Multer (handles the incoming file data):</b></p>
 <p>cd &#126;&#47;server<br>
 npm install multer<br>
 <br>
 (When your server.js says const multer = require('multer');<br> Node.js looks specifically in that local node_modules folder to find the code that knows how to save files to your disk.)
<br>
<h4>Verify installation:</h4>
<p>ls node_modules | grep multer
</p>
<h2>"Master" Code (server.js)</h2>
<p>Create this file inside ~/server. It is hardcoded to only look at /home/ubuntu/remote-vault.<br>
This version combines Jailer Security, Password Protection, Navigation, Downloading, and Uploading.<br>

<b>Copy paste the source code from server.js.</b></p>

<h2><b>With very strong security:</b></h2>

Bash: <br>
npm install express cors multer helmet express-rate-limit jsonwebtoken<br>
<br>
server.js file with this version assumes:<br>
Single user<br>
JWT sessions<br>
AES-256-GCM encryption<br>
Password required for decrypt<br>
No plaintext saved<br>
Secure headers<br>
Rate limiting<br>
Logging<br>
Strict CORS (Cloudflare domain only)<br>

Copy paste the source code from sec_server.js 
</p>
<h2><b>Making it Global (Cloudflare Tunnel)</b></h2>
<p>Since we aren't using the host machine, we run the tunnel inside Ubuntu.<br>
<br>Install in VM(for Mac):<br>
Bash<br>
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64.deb<br>
sudo dpkg -i cloudflared.deb<br>
<br>Verify the Installation<br>
Once that finishes, check if it's working by typing:<br>
Bash:<br>
cloudflared --version<br>
<br>
Running the tunnel:<br>
cloudflared tunnel --url http://localhost:3001.   (In place of "localhost", IP Address can also be used.)<br>
3. Result: Cloudflare gives you a URL (e.g., https://my-secure-vault.trycloudflare.com).<br>
</p>
