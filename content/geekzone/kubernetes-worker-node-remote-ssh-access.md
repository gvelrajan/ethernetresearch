# Kubernetes Worker Node Remote SSH Access
Kubernetes is a very popular and widely deployed container management and orchestration platform, preferred by devops engineers worldwide today.

Usually Kubernetes clusters and their worker nodes are not exposed to the public Internet but the apps running in them are.

SocketXP TLS VPN solution is a lightweight VPN that provides secure remote SSH access to private Kubernetes Clusters in your on-prem cloud or public cloud or multi-cloud or hybrid cloud.

::: tip Note: 
We at SocketXP are looking for beta customers to evaluate and provide feedback for our Kubernetes Remote Access Solution.  Please feel free to reach out to us at: support@socketxp.com
:::

## Overall Strategy -- In a nutshell
We'll install SocketXP agent in your worker nodes and configure it to function as an SSH server.  SocketXP agent will also establish a secure TLS VPN connection with the SocketXP Cloud Gateway.  You could then, remote SSH into your Kubernetes worker nodes from the SocketXP Cloud Gateway Portal using your browser.  No SSH client is required to SSH into your worker nodes.

Let's get started!

### Step 1:  Download and Install
[Download and install](https://www.socketxp.com/download/) the SocketXP agent on your Kubernetes Worker Node.

### Step 2: Get your Authentication Token
Sign up at [https://portal.socketxp.com](https://portal.socketxp.com) and get your authentication token.

![SocketXP AuthToken](https://dev-to-uploads.s3.amazonaws.com/i/8h8dtakf5nl1c85kem9z.jpg)

Use the following command to authenticate you node with the SocketXP Cloud Gateway using the auth token.
``` bash
$ socketxp login <your-auth-token-goes-here>
```

### Step 3: Create SocketXP TLS VPN Tunnel for Remote SSH Access
Use the following command to create a secure and private TLS tunnel VPN connection to the SocketXP Cloud Gateway.

``` bash
$ socketxp connect tcp://localhost:22  --iot-device-id "kube-worker-node-001"  --enable-ssh --ssh-username "test-user" --ssh-password "password123"

TCP tunnel [test-user-gmail-com-34445] created.
Access the tunnel using SocketXP agent in IoT Slave Mode
```
Where TCP port 22 is the default port at which the SocketXP agent would listen for SSH connections from any SSH clients.  The "--iot-device-id" represents a unique identifier assigned to the Kubernetes worker node within your organization.  It could be any string value but it must be unique for each of your worker node.

::: warning Security Info:
SocketXP <strong>does not</strong> create any public TCP tunnel endpoints that can be connected and accessed by anyone in the internet using an SSH client. SocketXP TCP tunnel endpoints are not exposed to the internet and can be accessed only using the SocketXP agent (using the auth token of the user) or through the XTERM terminal in the SocketXP Portal page.

SocketXP also has the option to setup and use your private/public keys to remote SSH into your worker nodes.
:::

You could now remote SSH into your Kubernetes worker node by clicking the terminal icon as shown in the screenshot below. 

![SSH Kubernetes Worker Node](https://dev-to-uploads.s3.amazonaws.com/i/bnlax2jo0tb5v0s5kta1.png)

Next, you'll will be prompted to provide your SSH login and password.

Once your credentials are authenticated with your SSH server you'll be logged into your device's shell prompt.

The screen capture below shows the "htop" shell command output from an SSH session created using the XTERM window in the SocketXP Portal page.

![SSH Kubernetes Worker Node](https://dev-to-uploads.s3.amazonaws.com/i/tcg50e43i1dbnleqyka1.jpg)

## Configuring SocketXP agent to run in slave mode
This is an alternate method for SSH into your private worker node from a remote location using the SocketXP Remote SSH Access solution.

If you don't want to access your worker node using a browser(via SocketXP Portal) and you want to access it using an SSH client (such as PuTTy) installed on your laptop or desktop, follow the instructions below.

First download and install the regular SocketXP agent software on your access device (such as a laptop running Windows or Mac OS). Next, configure the agent to run in slave mode using the command option "--iot-slave" as shown in the example below. Also, specify the name of the private TCP tunnel you want to connect to, using the  `--tunnel-name` option.

``` bash
$ socketxp connect tcp://localhost:3000 --iot-slave --tunnel-name test-user-gmail-com-34445

Listening for TCP connections at:
Local URL -> tcp://localhost:3000
Accessing the IoT device from your laptop
```

::: tip Why this is important?
SocketXP IoT Agent when run in Slave Mode acts like a localproxy server.  It proxies all connections to a user-specified local port (10111 in the example above) in your laptop/PC to the SocketXP Cloud Gateway using a secure SSL/TLS tunnel.  Also the SocketXP Agent authenticates itself with the SocketXP Cloud Gateway using your auth token.  This ensures that only legitimate, authenticated users are permitted to access your private worker nodes. SocketXP ensures Zero-Trust security on all connected devices.
:::

Now you can SSH access your Kubernetes Worker Node using the above SocketXP local endpoint, as shown below.
``` bash
$ ssh -i ~/.ssh/test-user-private.key test-user@localhost -p 3000
```
::: tip Tip:
You can also use [PuTTY](https://www.putty.org/) SSH client to remote SSH into your device using the same parameters show above.  Similarly, you can use PuTTY or [FileZilla](https://filezilla-project.org/) to perform SFTP actions such as file upload and file download to your private Kubernetes Worker Nodes.
:::

::: tip Want to be a Beta customer?: 
We at SocketXP are looking for Beta customers to evaluate and provide feedback for our Kubernetes Remote Access Solution that includes Worker Node/Pod SSH access/Microservice Remote Access/Database Remote access.  Please feel free to connect with us at: [support@socketxp.com](support@socketxp.com)
:::