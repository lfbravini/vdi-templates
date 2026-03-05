# Multi-Tenant DaaS Satellite Domain (Portainer Template)

This repository contains the master blueprints for a "1-click" deployment of isolated Samba 4 Active Directory Domain Controllers. It is designed specifically to act as the backend infrastructure for multi-tenant Desktop-as-a-Service (DaaS) environments, such as VMware Horizon.

By utilizing Portainer's App Templates (JSON) engine, this repository generates a dynamic, user-friendly GUI that allows administrators to deploy fully configured, isolated tenant domains in seconds.

## 🏗️ Architecture & Features

This stack deploys two containers working in tandem to solve the notorious initialization quirks of containerized Samba deployments:

1. **The Init-Container (Setup Crew):** A lightweight Alpine Linux container that runs a conditional bash script to check for the existence of the Samba database and `smb.conf`. If it is a fresh deployment, it safely initializes the configuration file. If it is a server reboot, it gracefully bypasses the initialization, protecting the timestamps and preventing boot loops.
2. **The Main Engine (Samba DC):** The `nowsci/samba-domain` image, which waits for the init-container to finish before booting. It acts as the dedicated Active Directory controller, DNS server, and authentication boundary for the tenant's virtual desktops.

**Key Features:**
* **True Multi-Tenancy:** Each client gets a dedicated, isolated 50MB container acting as their Active Directory forest.
* **Macvlan Networking:** Assigns true, static IP addresses to the containers so they can integrate cleanly with traditional Windows Server infrastructure.
* **Idempotent Design:** Safe to reboot, update, or restart without destroying the Active Directory database.

## 🚀 How to Use (Portainer Integration)

To attach this "assembly line" to your Portainer instance:

1. Open your Portainer Dashboard.
2. Navigate to **Settings** (gear icon in the bottom left).
3. Scroll down to the **App Templates** section.
4. Replace the default URL with the raw JSON link of this repository:
   `https://raw.githubusercontent.com/YOUR-USERNAME/YOUR-REPO-NAME/main/templates.json`
5. Click **Save settings**.

Navigate to **App Templates** in your Portainer sidebar. You will now see the **DaaS Satellite Domain** icon. Clicking it will automatically generate the deployment form.

## ⚙️ Environment Variables

When deploying through the Portainer GUI, the template will ask for the following parameters:

| Variable | Description | Example |
| :--- | :--- | :--- |
| `CLIENT_NAME` | The unique ID for the client (used for container naming and volume mapping). | `client02` |
| `CLIENT_DOMAIN` | The internal FQDN for the tenant's Active Directory. | `client02.local` |
| `ADMIN_PASSWORD` | The Domain Administrator password. | `StrongPass123!` |
| `STATIC_IP` | The dedicated IP address on the Macvlan network. | `172.69.100.75` |
| `ANCHOR_IP` | The IP of your central Management/Anchor Domain Controller. Dns forwarder. | `172.16.6970` |
| `NETWORK_NAME` | The exact name of your external Macvlan network in Docker. | `vdi_macvlan` |

## 💾 Data Persistence

The stack automatically maps persistent volumes to the host machine. By default, it stores the Active Directory database and configuration files at:
`/mnt/samba-vdi/${CLIENT_NAME}/...`

*(Ensure your Docker host has the appropriate permissions to write to this directory).*
