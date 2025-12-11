# Node-RED on Raspberry Pi Lab Manual

This manual provides a step-by-step guide for setting up Node-RED on a Raspberry Pi and introduces basic flow creation.

## I. Prerequisites

Ensure you have the following before beginning the installation:

  * **Raspberry Pi:** Any model running Raspberry Pi OS (32-bit or 64-bit).
  * **Operating System Access:** Access to the Pi's terminal (via SSH or the desktop environment).
  * **Network:** A stable internet connection for the Pi.

## II. Installation and Upgrade

Node-RED is often included in Raspberry Pi OS, but it is best practice to run the dedicated installer script to ensure you have the latest stable versions of Node-RED and Node.js.

### 1\. Execute the Installation Script

Open the terminal on your Raspberry Pi and execute the following command:

```bash
bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)
```

### 2\. Follow Prompts

The script will:

  * Install/upgrade Node.js.
  * Install/upgrade Node-RED.
  * Ask if you want to install **Pi-specific nodes** (e.g., GPIO nodes). **Recommendation: Select 'Y'**.
  * Ask if you want to configure Node-RED to **autostart** on boot. **Recommendation: Select 'Y'**.
  * Ask if you want to set up **security/admin login**. **Recommendation: Select 'Y'** and create a secure username and password.

## III. Running and Accessing Node-RED

Once the installation is complete, you can start the service and access the web-based flow editor.

### 1\. Starting the Service

If you did not enable autostart, or if you need to manually start the service:

```bash
node-red-start
```

### 2\. Accessing the Editor

Node-RED runs on the standard port **1880**.

| Access Location | URL to Use | Notes |
| :--- | :--- | :--- |
| **On the Pi itself** | `http://localhost:1880` | Use the Pi's local web browser. |
| **From another computer** | `http://<your_pi_ip_address>:1880` | You must know your Pi's IP address. Use `hostname -I` in the Pi terminal to find it. |

## IV. Basic Flow Creation: "Hello World"

This exercise introduces the core elements: **Inject** (input), **Debug** (output), and **Deployment**.

### Procedure

1.  **Open the Editor:** Navigate to the Node-RED URL in your browser.
2.  **Add an Input Node:** Drag an **`inject`** node from the left-hand palette onto the central workspace (the flow).
      * Double-click the `inject` node.
      * Set the **Payload** type to **`string`** and enter the value: `Hello, world!`
      * Click **Done**.
3.  **Add an Output Node:** Drag a **`debug`** node onto the workspace.
4.  **Connect the Nodes:** Click and drag a wire from the right output port of the `inject` node to the left input port of the `debug` node.
5.  **Deploy:** Click the **`Deploy`** button in the top-right corner.
6.  **Test the Flow:**
      * Open the **Debug sidebar** (the bug icon 🐞 on the far right).
      * Click the small square button on the left side of the `inject` node.
      * You should immediately see the `Hello, world!` string appear in the Debug sidebar.

## V. Intermediate Flow: Formatting a Timestamp

This exercise introduces the **Function** node for custom JavaScript logic. The goal is to convert the raw numerical timestamp injected by default into a human-readable date string.

### Procedure

1.  **Add the Function Node:** Drag a **`function`** node onto the workspace, placing it *between* the existing `inject` and `debug` nodes.

2.  **Rewire the Flow:**

      * Delete the wire between `inject` and `debug`.
      * Connect `inject` → `function`.
      * Connect `function` → `debug`.

3.  **Configure the Function Node:**

      * Double-click the `function` node.
      * Change the payload type of the **`inject`** node back to its default: **`timestamp`**.
      * Enter the following JavaScript code into the function editor:

    <!-- end list -->

    ```javascript
    // msg.payload contains the timestamp (number of milliseconds since epoch)
    var date = new Date(msg.payload);
    // Overwrite the payload with the readable string
    msg.payload = date.toString();
    return msg;
    ```

4.  **Deploy and Test:** Click **`Deploy`**. Click the button on the **`inject`** node. The Debug sidebar should now display a fully formatted date/time string, such as `Thu Dec 11 2025 13:55:38 GMT+0800 (Singapore Standard Time)`.

## VI. Node-RED Service Management Commands

These are essential terminal commands for controlling the Node-RED service on your Raspberry Pi:

| Command | Purpose |
| :--- | :--- |
| `node-red-start` | Starts the Node-RED service. |
| `node-red-stop` | Stops the running service. |
| `node-red-restart` | Restarts Node-RED without having to stop/start manually. |
| `node-red-log` | Displays the log output (useful for troubleshooting flow issues). |
| `sudo systemctl enable nodered.service` | Configures Node-RED to start automatically when the Pi boots (if not enabled during initial install). |


