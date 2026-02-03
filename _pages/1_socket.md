---
layout: page
permalink: /assignments/assignment1
title: "Assignment 1: Socket Programming and Measurement"
---
#### **Released:** 01/22/2026 <br/> **Due:** 02/03/2026
{: .no_toc}

* (The list will be replaced with the table of contents.)
{:toc}

### Part 0: Overview and Setup
#### Overview
Your task is to complete [`src/iperfer.c`](https://github.com/utcs356/assignment1/blob/main/src/iperfer.c) to meet the specifications in Part 1 and perform measurements using your program in Part 2.

**Note**: Please set up your CloudLab experiment in the same way as in Assignment 0. Make sure to use profile `cs356-base`

**Note**: Don’t forget to run the following commands for every CloudLab experiment instantiation.
* After ssh to the reserved node, type the commands below.
    `$ sudo usermod -aG docker $USER`
* Then **restart the SSH session** to make sure the changes are applied.

#### Setup
**Make sure to save changes you make to your private GitHub repo.** Otherwise, you will lose the changes when your CloudLab experiment ends.
1. **Get the skeleton code for A1 and setup your private repository.**
To get the skeleton code, create a **private** repository by clicking `Use this template> Create a repository` on the [GitHub repository](https://github.com/utcs356/assignment1.git). Don't forget to select `Private` while creating the repository.

2. **Setup GitHub authentication on a CloudLab node**
When you want to make changes to your private repository on a CloudLab node, you have to authenticate every time you instantiate an experiment. This is because your authentication information is cleaned up upon the end of the experiment. There are three ways to authenticate your account on the reserved node.

    <details>
    <summary markdown="span"> With SSH key and SSH agent forwarding (recommended) </summary>

    1. Prepare a key pair that is registered for your GitHub account.
        * Refer to [this](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account) to add a new ssh key to your GitHub account.
        * Check if the key is added properly with `$ ssh -T git@github.com` on your local machine.
    2. Setup a local `ssh-agent` and SSH to the CloudLab node with the SSH agent forwarding option (`-A`).
        <details>
        <summary markdown="span"> For macOS and Linux: </summary>

        1. Launch a local SSH agent.
        `$ ssh-agent`
        2. Add your private key to the agent.
        `$ ssh-add <path_to_your_private_key>`
        3. SSH to the CloudLab node with the `-A` option.
        `$ [ssh command] -A`
        * To automate step a and b, append the below to your shell startup file (e.g., `.bashrc`, `.zshrc`). After this, you do not have to repeat step a and b every time.
            ```
            eval "$(ssh-agent -s)"
            ssh-add <path_to_your_private_key>
            ```
        </details>

        <details>
        <summary markdown="span"> For Windows: </summary>

        * Open a terminal as an administrator and follow the steps below.
        1. Launch a local SSH agent and automate the launch.
        `> Get-Service ssh-agent | Set-Service -StartupType Automatic`
        `> Start-Service ssh-agent`
        2. Add your private key to the agent.
        `> ssh-add <path_to_your_private_key>`
        3. SSH to the CloudLab node with the `-A` option.
        `> [ssh command] -A`
        </details>
    3. You should be able to clone your private repository with the SSH URL on the Cloudlab node.
        * Check if the ssh-agent forwarding works properly with `$ ssh -T git@github.com` on the CloudLab node.
    </details>

    <details>
    <summary markdown="span"> With VS Code </summary>

    * Once you install the `GitHub Pull Requests and Issues` [extension](https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-pull-request-github) on VS Code, you can authenticate the remote server through VS Code and a web browser. You also can clone the repository on VS Code to the remote server.
    * Refer to [this](https://code.visualstudio.com/docs/sourcecontrol/github) for more details.
    </details>

    <details>
    <summary markdown="span"> With personal access tokens </summary>

    * You can generate a personal access token for the account/repositories to access your repositories over HTTPS with the token.
    * Refer to [this](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) for more details.
    </details>

### Part 1: Implement iperfer.c
Your task is to complete the source code for `iperfer`. `iperfer` is a program to measure the throughput between two hosts. It should be executed on one host in the server mode and then executed on the other in the client mode. Argument parsing is already implemented in the skeleton code. Refer to the below specifications for implementation details:
* **Client mode**: Send TCP packets to a specific host for a specified time window and track how much data was sent during the time window.
    * Usage: `./iperfer -c -h <server_host_ipaddr> -p <server_tcp_port> -t <duration>`
        * `-c` indicates the client mode.
        * `-h <server_host_ipaddr>` specifies the IP address of a server to connect.
        * `-p <server_tcp_port>` specifies the TCP port number of a server to connect.
        * `-t <duration>` specifies the duration in seconds for which the program will send data. It should be a positive integer.
    * Specification
        * Argument check: Check the given port number is between 1 and 65535 (inclusive). Check the given duration is positive.
        * Create a socket with `socket` then `connect` to the server specified by command line arguments. (`<server_host_ipaddr>` and `<server_tcp_port>`)
        * Data should be sent in chunks of 1000bytes and the data should be all zeros.
        * The program should close the connection and stop after the specified time (`<duration>`) has passed.
        * When the connection is closed, the program should print out the elapsed time, the total number of bytes sent (in kilobytes), and the rate at which the program sent data (in Mbps) (1kilobyte=1000bytes, 1megabit = 1,000,000 bits = 125,000 bytes)

* **Server mode**: Wait for the client connection on the specified port. After the connection is established, receive TCP packets until the connection ends.
    * Usage: `./iperfer -s -p <server_tcp_port>`
        * `-s` indicates the server mode.
        * `-p <server_tcp_port>` specifies the TCP port number to which the server will bind.
    * Specification
        * Argument check: check the given port number is between 1 and 65535. (inclusive)
        * Create socket with `socket`.
        * `bind` socket to the given port (`<server_tcp_port>`) and `listen` for TCP connections.
        * Then wait for the client connection with `accept`.
        * After the connection is established, received data in chunks of 1000bytes
        * When the connection is closed, the program should print out the elapsed time, the total number of bytes received (in kilobytes), and the rate at which the program received data (in Mbps) (1kilobyte=1000bytes, 1megabit = 1,000,000 bits = 125,000 bytes)

**Report** your iperfer client results when running it for 10 seconds in the given `two_hosts_direct` topology.
* Two hosts, h1 and h2, are directly connected to each other in this topology.
* You can start a Kathara lab and connect to a virtual network device as in [A0](https://utcs356.github.io/sp26/assignments/assignment0#part-2-create-a-virtual-network-with-kathar%C3%A1).
* In the terminal for each host, your binary is located in `/shared` directory. This is a mirror of files located in `labs/two_hosts_direct/shared` directory on the CloudLab node.

#### Notes
* **Compile**: `Makefile` is included in the directory, so you can compile your code with `$ make` and remove the compiled binary with `$ make clean`. The compiled binary will be located in the `bin` and `labs/<lab_name>/shared` directories.
* **Local Testing**: We recommend you test your program in local environment prior to Kathara experiments.
  * Set the host IP address (`<server_host_ipaddr>` argument when running the client) to `127.0.0.1`.
  * Choose a port in the range of 1024 to 49151 when starting the server (e.g., `./iperfer -s -p 8000`).

### Part 2: Measurement on a Virtual Network
In this part of the assignment, you will use the tool you implemented (`iperfer`) and `ping` to measure a given virtual network's end-to-end bandwidth and latency, respectively. We will use `six_hosts_two_routers` topology. The virtual network topology is formed as follows:
* There are 6 hosts, `h[1-6]`, and 2 routers, `r[1-2]`.
* `h1`,`h2`,`h3` are connected to `r1`, and `h4`,`h5`,`h6` are connected to `r2`. `r1` and `r2` are connected with a single link.
![Part 2 Topology]({{site.baseurl}}/assets/img/assignments/assignment1/P2_topology.png)

**NOTE**: To measure the average RTT (or latency), use `$ ping -c [number_of_pings] [remote_ip_address]`.
For example, if you want to ping to `h4` 10 times, the command is `$ ping -c 10 30.1.1.7`.

**NOTE**: To measure the bandwidth (or throughput) between two hosts (say `h1` and `h4`), execute `iperfer` as a client mode on one host then execute `iperfer` as a server mode on the other host.

**NOTE**: When you change Kathara startup files (e.g., `r1.startup`), you must stop the running Kathara lab with `$ kathara lclean` before every change.

#### **Q1**: Basic measurements
* Measure and report average RTT and throughput between two adjacent routers, `r1` and `r2`.
* Measure and report average RTT and throughput between two hosts, `h1` and `h4`.

#### **Q2**: Impact of multiplexing on latency.
* Measure and report average RTT between two hosts, `h1` and `h4`, while measuring bandwidth between `h2` and `h5`.
* How does it compare to the measured latency in Q1 (RTT between `h1` and `h4`)?

#### **Q3**: Impact of multiplexing on throughput
* Report the throughput between a pair of hosts varying the number of host pairs that conduct measurements.
    * The host pairs are (`h1`,`h4`), (`h2`,`h5`), (`h3`,`h6`).
    * e.g., First, measure throughput between (`h1`,`h4`). Then measure throughput between (`h1`, `h4`) and throughput between (`h2`,`h5`) simultaneously. Finally, do the measurements on (`h1`,`h4`), (`h2`,`h5`), and (`h3`,`h6`) simultaneously.
    * You do not need to test all possible combinations. It is sufficient to measure throughput once for each number of active host pairs, as illustrated above.

* How do the three measurements varying the number of concurrent host pairs compare to the measured throughput in Q1 (between `h1` and `h4`)?
* What's the trend between measured throughput and the number of host pairs?

#### **Q4**: Impact of link capacity on end-to-end throughput and latency.
* Decrease link rate between `r1` and `r2` to 1Mbps. This can be done by uncommenting line #5 in the `labs/six_hosts_two_routers/r1.startup` and `labs/six_hosts_two_routers/r2.startup` files. You have to clean up the previous Kathara lab before every change. Relaunch it after the changes.
* Measure and report path latency (average RTT) and throughput between two hosts, `h1` and `h4`.
<!-- **Edit**: For this experiment, please use `h1` as a client and `h4` as a server as the provided Kathara lab configuration only limits the one-way bandwidth.
**Edit**: Alternatively, you can add the below to `r2.startup` in addition to uncommenting line #5 to change the bi-directional link bandwidth. Make sure commenting it out after this experiment.
`tc qdisc add dev eth1 root tbf rate 10mbit buffer 10mb latency 10ms`   -->
* How does it change compared to Q1?

#### **Q5**: Impact of link latency on end-to-end throughput and latency.
* Comment out the lines that you uncommented for the previous question.
* For each case below, measure and report path latency (average RTT) and throughput between two hosts, `h1` and `h4`.
    * Change the link delay between `r1` and `r2` to 10ms by uncommenting line #6 of the `r1.startup` file.
    * Change the link delay between `r1` and `r2` to 100ms by uncommenting line #7 of the `r1.startup` file. Comment out its line #6.
    * Change the link delay between `r1` and `r2` to 1s by uncommenting line #8 of the `r1.startup` file. Comment out its line #7.
* What's the trend between the measured throughput and latency?

### Submission
You must submit:
* A tarball file of the modified assignment1 directory.
    * `$ tar -zcvf assign1_[firstname]_[lastname].tar.gz assignment1`
* A pdf file contains the Part 1 results and the measurement results for Part 2.

**Report Format**:

* Please include your name and EID in the report.
* You may use any tool you prefer to create your report (LaTeX, Google Docs, Microsoft Word, etc.), but **you must submit the final report as a PDF file**.

**Report Content**:

* For Part 1, include both screenshots and typed explanations when reporting your measurement results.
* For Part 2, for each question, provide the measurement results (screenshots and typed text) along with your answers to the corresponding analysis questions.

### Grading
* `iperfer.c` implementation
	* Command line argument verification: 5%
    * Server mode implementation: 15%
    * Client mode implementation: 15%
* Part 1 results: 15%

* Part 2 results
	* Q1: 10%
    * Q2: 10%
	* Q3: 10%
	* Q4: 10%
    * Q5: 10%


### Appendix. Kathara Basics
A Kathara lab is a set of preconfigured (virtual) devices. It is basically a directory tree containing:
* lab.conf ⇒ describes the network topology and the devices to be started
    * Syntax: `machine[arg]=value`
        * machine is the name of the device
        * if arg is a number ⇒ value is the name of a collision domain to which eth arg should be attached
        * if arg is not a number, then it must be an option and value the argument
* Subdirectories: contains the configuration settings for each device
* `<device_name>.startup` files that describe actions performed by devices when they are started.

To deploy a virtual network, move to the Kathara lab directory then type `$ kathara lstart`. Then the XTerm terminal connected to each virtual network device would appear. To terminate the deployment, type `$ kathara lclean`. Some useful kathara commands are summarized [here](https://www.kathara.org/man-pages/kathara.1.html).

### Tips

* **Socket Example**: It might be helpful to review the [socket-example](https://github.com/utcs356/socket-example) implementation to ensure the socket connection is properly established.
* **Running commands simultaneously**: If you’re concerned about running commands simultaneously, tmux features like `setw synchronize-panes` can help. After typing the commands in each pane, enabling synchronization lets you execute them all at once with a single press of Enter.
* **Range of Mbps**: In Part 2, the measured throughput is expected to be in the range of several hundred Mbps.
* **Data Sent/Received Discrepancy**: If you notice mismatches between the amount of data sent and received by the server and client, consider adding the `MSG_WAITALL` flag to the server's `recv` calls. Because TCP does not guarantee that data will be received in the same chunk sizes as it was sent, the flag forces the server to wait until the buffer is fully filled.
* **Error Handling**: When a function in `handle_client` (e.g., `connect`) returns an error, please handle it properly by cleaning up any allocated resources (e.g., socket file descriptors) and exiting the process with an appropriate error code (e.g., `EXIT_FAILURE`).
