<h1><b>Remote FILE ACCESS SYSTEM</b></h1>
<hr>


<h2><b>Create VM in Multipass</b></h2>
<p>Bash<br>multipass launch &#45;name primary &#45;cpus 1 &#45;memory 2G &#45;disk 6G</p>
<h2><b>VM and Enviroment Setup</b></h2>
<p><b>1. Enter the VM</b></p>
<p>Bash<br>multipass shell primary</p>
<p><b>2. Create the "Safe Zone" folder</b></p>
<p>Bash<br>mkdir &#45;p &#126;&#47;remote&#45;vault</p>
<p><b>3. Setup Node.js environment</b></p>
<p>Bash<br>sudo apt update && sudo apt install -y nodejs npm<br>
 mkdir &#126;&#47;server &amp;&amp; cd &#126;&#47;server<br>
 npm init &#45;y<br>
 npm install express cors<br>
<br>
<p><b>Install Multer (handles the incoming file data):</b></p>
 <p>Bash<br>cd &#126;&#47;server<br>
  npm install multer<br>
 <br>
 (When your server.js says const multer = require('multer');<br> Node.js looks specifically in that local node_modules folder to find the code that knows how to save files to your disk.)
<br>
<p><b>Verify installation:</b></p>
<p>Bash<br>ls node_modules | grep multer
</p>
<h2>"Master" Code (server.js)</h2>
<p>Create this file inside ~/server. It is hardcoded to only look at /home/ubuntu/remote-vault.<br>
This version combines Jailer Security, Password Protection, Navigation, Downloading, and Uploading.<br>

<b>Use the source code from server.js.</b></p>



<h2><b>With very strong security:</b></h2>

Bash: <br>
npm install express cors multer helmet express-rate-limit jsonwebtoken<br>
<br>
server.js file with this version assumes:<br><br>
&nbsp;Single user<br>
&nbsp;JWT sessions<br>
&nbsp;AES-256-GCM encryption<br>
&nbsp;Password required for decrypt<br>
&nbsp;No plaintext saved<br>
&nbsp;Secure headers<br>
&nbsp;Rate limiting<br>
&nbsp;Logging<br>
&nbsp;Strict CORS (Cloudflare domain only)<br><br>

<b>Use the source code from sec_server.js</b>
</p>

<p><b>NOTE: The most secure way to do it is by keeping the files strictly inside the VM and not mounting a host (Mac) folder,<br>
 you create a "Hard Isolation" boundary. If the VM is compromised, the attacker is stuck inside a virtual disk and cannot <br>
 touch your Mac's photos, passwords, or system files. <br>
<br>
The Architecture (Strict Isolation)<br>
In this setup, your files live on the Virtual Disk (.img or .qcow2) managed by Multipass.</b></p>

<h2><b>Create the Test File</b></h2>
<p>Now that the pipe is open, let's put something in the vault so you can see it. <br>
Open a new terminal into your VM and run:<br>
<b>1. Go to your dedicated folder</b><br>
Bash<br>cd ~/remote-vault<br>
<b>2. Create a test file</b><br>
Bash<br>echo "Hello from your Ubuntu VM!" > test_file.txt<br>
<b>3. Create a test folder</b><br>
Bash<br>mkdir Secrets<br>
echo "This is inside a folder" > Secrets/note.txt</p>

<h2><b>Check if the Backend is Alive</b></h2>
<p>Open a new terminal tab in your Multipass VM and run:<br>
Bash<br>curl http://localhost:3001/api/files<br><br>
Restart Everything<br>
&nbsp;&nbsp;Kill the old tunnel: Go to the terminal where cloudflared is running and hit Ctrl + C.<br>
&nbsp;&nbsp;Start the Server: In one window, run <i>node server.js.</i><br>
&nbsp;&nbsp;Start the Tunnel: In another window, run <i>cloudflared tunnel --url http://localhost:3001.</i><br>
&nbsp;&nbsp;Wait 10 seconds: Give Cloudflare a moment to "wake up" the new URL.<br>
<br>
How to find your link<br>
Look at the terminal window where you run <i>cloudflared tunnel --url http://localhost:3001</i>. <br>
You are looking for a section that looks like a box:<br>
&nbsp;&nbsp;Find the line that says: Your quick Tunnel has been created! Visit it at:<br>
&nbsp;&nbsp;The link will look like: https://something-random-words.trycloudflare.com<br>
&nbsp;&nbsp;Copy that exact link.</p>


<h2><b>Making it Global (Cloudflare Tunnel)</b></h2>
<p>Since we aren't using the host machine, we run the tunnel inside Ubuntu.<br>
<br><b>Install in VM(for Mac):</b><br>
Bash<br>
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64.deb<br>
sudo dpkg -i cloudflared.deb<br>
<br><b>Verify the Installation</b><br>
Once that finishes, check if it's working by typing:<br>
Bash:<br>
cloudflared --version<br>
<br>
<b>Running the tunnel:</b><br>
Bash<br>cloudflared tunnel --url http://localhost:3001.   (In place of "localhost", IP Address can also be used.)<br>
<b>Result: </b>Cloudflare gives you a URL (e.g., https://my-secure-vault.trycloudflare.com).<br>
</p>


<h2><b>The Shutdown Chain Reaction</b></h2>
<p>When you shut down the VM (or close your Mac/put it to sleep):<br>
&nbsp;&nbsp;server.js stops: The Node.js process is killed. The "File Indexer" is gone.<br>
&nbsp;&nbsp;Cloudflared stops: The tunnel "pipe" is disconnected.<br>
&nbsp;&nbsp;The Link Breaks: If you try to visit your trycloudflare.com URL,<br>
&nbsp;&nbsp;you will get a "502 Bad Gateway" or a "Cloudflare Error" because there is no server on the other end to answer the call.<br>
<br>
<b>The "Quick Tunnel" Problem</b><br>
Because we are using a Quick Tunnel (the free version that doesn't require a Cloudflare account),<br>
 the link is temporary.<br>
&nbsp;&nbsp;Every time you restart the tunnel, Cloudflare gives you a new, random URL.<br>
&nbsp;&nbsp;If you want the same link every time (like my-private-vault.com), you would eventually need to create a <br>
&nbsp;&nbsp;free Cloudflare account and link a domain.</p>

<h2><b>How to make it "Always On" (Persistence)</b></h2>
<p>If you want this to run smoothly without you manually typing node server.js every single time you open your VM,<br>
 you need to set up Services.
In Ubuntu, we use a tool called PM2 (Process Manager) or systemd. These tools act like a "Manager" that:<br>
Starts your server automatically when the VM boots up.
Restarts the server immediately if it crashes.<br><br>
To set up PM2 in your VM:<br>
1. Install PM2 globally<br>
Bash<br>sudo npm install &#45;g pm2<br>
2. Start your server and the tunnel with PM2<br>
Bash<br>pm2 start server.js &#45;name &quot;server&quot;<br>
pm2 start "cloudflared tunnel &#45;&#45;url http://localhost:3001" &#45;&#45;name "tunnel"<br>
3. Tell PM2 to run on VM startup<br>
Bash<br>pm2 startup<br>
<b>(Copy and paste the command it gives you)</b><br>
Bash<br>pm2 save<br><br>
<b>The Concept: Hardcoded vs. Environment</b><br>
Hardcoded (Insecure): c<i>onst MASTER_PASSWORD = "my-secret-123";</i><br>
If you share the file, you share the password.<br>
Environment Variable (Secure): <i>const MASTER_PASSWORD = process.env.PASSWORD;</i><br>
The code says: "Look at the system settings to find the value for 'PASSWORD'."</p>

<h2><b> How to update your server.js</b></h2>
<p>In your Ubuntu VM, change that one line in your server.js to this:<br>
<i>const MASTER_PASSWORD = process.env.VAULT_PASS || “default-password";</i><br>
(The || "default-password" is a backup just in case you forget to set one.)<br>
<br>

How to run it now<br>
Now, when you start your server, you "inject" the password into the command like this:<br>
Bash<br>
<i>VAULT_PASS=your-real-password-here node server.js</i><br>
By doing this, the password lives in your terminal history, but it is not written inside the code that you upload to GitHub.</p>


<h2><b>Vault_Pass Error</b></h2>
<p>If it is defaulting to default password, it means the VAULT_PASS variable isn't reaching the Node.js process correctly.<br> 
This usually happens for one of two reasons:<br> 
A Syntax Error in the Command: If there is a space between the name and the value.
Environment Pollution:<br> <br>
The server is being restarted by a background manager (like PM2) that is stuck on the old settings.<br>
<b>Step 1:</b> The "Clean" Run Command<br> 
In Linux, the variable must be attached directly to the command with no spaces. Try running it exactly like this:<br> 
Bash<br> 
VAULT_PASS=my-secret-123 <br> 
node server.js<br> 
<br>
<b>Step 2:</b> Clear "Ghost" Processes<br> 
If you have PM2 running in the background, it will keep restarting the server with its own saved settings, <br> 
ignoring your new command. Let's kill everything to be sure:<br> 
Stop any background node processes<br> 
Bash<br>sudo killall -9 node<br> 
If you used PM2 earlier, stop it too<br> 
Bash<br>pm2 kill<br> 
</p>


<h2><b>The "Export" Trick</b></h2>
<p>If the command-line variable still isn't working, you can "lock" the password into your terminal session first:<br>
Bash<br>
export VAULT_PASS=12345<br>
node server.js<br>
<br>
By using export, the variable stays active in that terminal window until you close it.<br>
Here is how to set up PM2 correctly so it respects your VAULT_PASS.<br>
<br>
<b>1. The "Clean Slate" Start</b>
If you have PM2 running already, kill it first to ensure no "ghost" settings are lingering:<br>
Bash<br>
pm2 kill
<br>
<b>2. Starting with the Variable</b>
To give PM2 your password, you pass the variable before the start command, just like you did with Node:<br>
Bash<br>
VAULT_PASS=your_secret_password pm2 start server.js --name “vault-system"<br>
<br>
<b>3. How to Update the Password later</b><br>
If you decide to change your password, you can't just run the command again. You have to use the --update-env flag:<br>
Bash<br>
VAULT_PASS=new_password_123 pm2 restart vault-system --update-env<br>
<br>
<b>Note: Without --update-env, PM2 will restart the server but keep using the old password.</b>
</p>
