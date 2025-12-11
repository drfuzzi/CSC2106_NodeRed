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

That's great\! Integrating sections on MQTT, REST, and BLE will significantly enhance the practical value of this lab manual for students exploring IoT.

I will add these three new sections (VII, VIII, IX) to the Markdown structure.


## VII. Messaging with MQTT Broker

MQTT (Message Queuing Telemetry Transport) is the backbone of many IoT systems. This exercise involves installing a local broker and setting up a basic publish/subscribe flow.

### 1\. Installing the Mosquitto Broker

The Mosquitto software will turn your Raspberry Pi into a local MQTT message broker.

Open the terminal on the Raspberry Pi and run:

```bash
sudo apt update
sudo apt install -y mosquitto mosquitto-clients
```

### 2\. Setting up the Service

Enable and start the service so it runs automatically:

```bash
sudo systemctl enable mosquitto
sudo systemctl start mosquitto
```

### 3\. Creating a Publish Flow (Client 1: Publisher)

This flow simulates a sensor publishing data to the broker.

| Step | Action | Node(s) Used | Description |
| :--- | :--- | :--- | :--- |
| 1 | **Create Input** | **Inject** | Drag an **`inject`** node. Set the Payload to **`string`** and the value to **`TEMP: 25.5 C`**. |
| 2 | **Add MQTT Output** | **mqtt out** | Drag an **`mqtt out`** node and connect the `inject` node to it. |
| 3 | **Configure MQTT Node** | **mqtt out** | Double-click the `mqtt out` node. Click the pencil icon to add a new server connection. Set the **Server** to `localhost` and the **Port** to `1883`. Click **Add**. |
| 4 | **Set Topic** | **mqtt out** | Back in the node configuration, set the **Topic** to: `sensors/pi/temperature`. Click **Done**. |
| 5 | **Deploy** | (Deploy) | Click the **Deploy** button. |

### 4\. Creating a Subscribe Flow (Client 2: Subscriber)

This flow subscribes to the topic and displays any received messages.

| Step | Action | Node(s) Used | Description |
| :--- | :--- | :--- | :--- |
| 1 | **Add MQTT Input** | **mqtt in** | Drag an **`mqtt in`** node. |
| 2 | **Configure MQTT Node** | **mqtt in** | Double-click and select the `localhost:1883` server connection you created earlier. |
| 3 | **Set Topic** | **mqtt in** | Set the **Topic** to the exact same one used by the publisher: `sensors/pi/temperature`. Click **Done**. |
| 4 | **Add Debug** | **Debug** | Connect the **`mqtt in`** node to a **`debug`** node. |
| 5 | **Test** | **Inject** | Click **Deploy**. Then, click the `inject` button on the publisher flow. The message will travel `Inject` → `MQTT Out` → **Mosquitto Broker** → `MQTT In` → `Debug` and appear in the debug sidebar. |

-----

## VIII. Interacting with REST APIs

Node-RED excels at interacting with web services using HTTP requests. This section shows how to retrieve external data.

### 1\. Retrieving Data from an External API

We will use a public API (like `jsonplaceholder.typicode.com`) to fetch a test user post.

| Step | Action | Node(s) Used | Description |
| :--- | :--- | :--- | :--- |
| 1 | **Create Trigger** | **Inject** | Drag an **`inject`** node. Set Payload to **`timestamp`** (default). |
| 2 | **Add HTTP Request** | **http request** | Drag an **`http request`** node and connect the `inject` node to it. |
| 3 | **Configure HTTP Request** | **http request** | Double-click the node. Set the **Method** to **`GET`**. Set the **URL** to: `https://jsonplaceholder.typicode.com/posts/1` |
| 4 | **Set Output** | **http request** | Set the **Return** property to **`a parsed JSON object`**. Click **Done**. |
| 5 | **Add Output** | **Debug** | Connect the **`http request`** node to a **`debug`** node. |
| 6 | **Test** | **Inject** | Click **Deploy**. Click the `inject` button. The debug sidebar should show the received JSON object, including `msg.payload.title` and `msg.payload.body`. |

### 2\. Displaying Parsed Data

To make the output cleaner, use a `function` node to extract just the title.

1.  Place a **`function`** node between the `http request` and `debug` nodes.
2.  In the `function` node, enter the following code:
    ```javascript
    // Extract the title from the JSON payload
    msg.payload = "Post Title: " + msg.payload.title;
    return msg;
    ```
3.  **Deploy and Test:** Click the `inject` button again. The debug output will now only show the formatted title.

-----

## IX. Basic BLE Interaction (Optional: Hardware Dependent)

Interacting with Bluetooth Low Energy (BLE) requires the correct hardware (most Raspberry Pi models have built-in BLE) and the installation of specific Node-RED nodes.

### 1\. Install BLE Nodes

Open the terminal on the Raspberry Pi and run:

```bash
cd ~/.node-red
npm install node-red-contrib-noble-bluetooth
# Or use the Node-RED Palette Manager (Menu -> Manage Palette -> Install)
```

### 2\. Example: Scanning for BLE Devices

This flow continuously scans for nearby BLE peripherals and outputs their names and identifiers.

| Step | Action | Node(s) Used | Description |
| :--- | :--- | :--- | :--- |
| 1 | **Create Trigger** | **Inject** | Drag an **`inject`** node. Set it to **repeat** every 10 seconds. |
| 2 | **Add BLE Scanner** | **ble scan** | Drag a **`ble scan`** node and connect the `inject` node to it. |
| 3 | **Configure Scanner** | **ble scan** | Double-click the node. Ensure **Start** is selected as the operation. Set a reasonable **Duration** (e.g., 5 seconds). Click **Done**. |
| 4 | **Add Debug** | **Debug** | Connect the output of the **`ble scan`** node to a **`debug`** node. |
| 5 | **Test** | **Inject** | Click **Deploy**. The `ble scan` node will output an array of nearby devices to the debug sidebar whenever triggered. (Ensure Bluetooth is enabled on the Pi). |


