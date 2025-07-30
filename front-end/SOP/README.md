# OT-MICROSERVICES Frontend Deployment Guide for Ubuntu 22.04 (SOP)

---

## Author Information

| **Author**   | **Created on** | **Version** | **Last updated on** | **Level** | **Reviewer**  |
|--------------|----------------|-------------|---------------------|-----------|---------------|
|Ashutosh Kumar| 2025-07-28     | v1          | 2025-07-29          | Internal  |Siddharth Pawar|

---

## Purpose

Frontend Web is a ReactJS application that provides the main user interface for the OT-Microservices stack. It works across platforms and only needs a JavaScript runtime to run, allowing users to interact with dynamic web pages that update efficiently as data changes.

---

## Pre-requisites
The frontend application have dependencies on other REST API of **[OT-Microservices](https://github.com/OT-MICROSERVICES)**. To run the application successfully, we need these things configured:

* **[Employee API](https://github.com/avengers-p7/Documentation/blob/main/OT%20Micro%20Services/Application/Employee_API.md)**
* **[Attendance API](https://github.com/avengers-p7/Documentation/blob/main/OT%20Micro%20Services/Application/Attendance_API.md)**
* **[Salary API](https://github.com/avengers-p7/Documentation/blob/main/OT%20Micro%20Services/Application/Salary_API.md)**


### Hardware

| Hardware Specifications | Minimum Recommendation |
| ----------------------- | ---------------------- |
| Processor | 1 CPU, t2.micro EC2 | 
| Ram | 1 GiB |
| Disk | 8 GB |
| OS | Ubuntu 22.04 LTS |
***


### Dependencies

#### Build Time
| Dependency | Version | Description               |
|------------|---------|--------------------------|
| nodejs     | >=16    | JS runtime for React     |
| npm        | >=8     | Package manager          |
| git        | latest  | For source code          |

#### Run Time
| Dependency | Version | Description                   |
|------------|---------|------------------------------|
| serve      | latest  | Static file server for build  |

# Important Ports
| Inbound Traffic	 | Description |
| ---------------- | ----------- |
| 22 | For SSH |
| 3000 | Default REACTJS port |

---

## Installation Steps (Detailed & Explained)

### Step 1: Install System Dependencies

```bash
sudo apt update
sudo apt install -y git curl build-essential
```
- **Explanation:** Ensures your system has essential build tools and git for source code management.

#### NodeJS & npm

> **Note:** Node 16 is EOL. For long-term stability, use Node 20.

```bash
# Install Node.js 20 (recommended)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```
- **Explanation:** Node.js 20 provides better security and performance. The project works on Node 16 but upgrade is best practice.

---

### Step 2: Clone the Repository

```bash
git clone https://github.com/OT-MICROSERVICES/frontend.git
cd frontend
```
- **Explanation:** Downloads the latest frontend code to your machine.

---

### Step 3: Install Project Dependencies

```bash
npm install
```
or (recommended for reproducible devops builds):

```bash
make install
```
- **Explanation:** Installs all required JavaScript dependencies. If you see **npm WARN deprecated** messages, these are safe to ignore for now (they’re due to old libraries).

---

### Step 4: Build the Application

```bash
make build
```
or

```bash
npm run build
```
**If you get “JavaScript heap out of memory” error:**  
Run with increased heap allocation:
```bash
NODE_OPTIONS="--max_old_space_size=4096" npm run build
```
*For low-RAM systems, use 512 or 1024 instead of 4096.*

- **Explanation:** React builds can be memory-intensive. Increasing the Node heap avoids build failures.

**Build Output:**
- Warnings like `no-unused-vars`, `jsx-a11y/anchor-is-valid` are **not fatal** and do not affect deployment.  
  - For production, these can be cleaned up later.

---

### Step 5: Serve the Production Build

```bash
sudo npm install -g serve
serve -s build -l 3000
```
- **Explanation:** Runs a static server on port 3000, hosting your built frontend.
- **Network Access:** Your app will be available at:
    - Local: `http://localhost:3000`
    - Network: `http://<your-ec2-ip>:3000`

> **Note:** The “Cannot copy server address to clipboard” error is harmless.  
> If you want to suppress it, install xsel:
> ```bash
> sudo apt install xsel
> ```

---

### Step 6: Access the Application

- Open your browser and go to:  
  `http://localhost:3000`  
  or  
  `http://<your-server-ip>:3000` (from another device on the network)

---

## Monitoring and Logging

- **Process Monitoring:** Use `htop`, `top` to check memory/CPU.
- **Logs:** The output of `serve` in your terminal. For errors, check browser console.
- **Health:** If the page loads, the app is healthy.

---

## Troubleshooting

| Issue                        | Solution                                                         |
|------------------------------|------------------------------------------------------------------|
| JS heap out of memory        | Use `NODE_OPTIONS="--max_old_space_size=4096" npm run build`     |
| Port 3000 in use             | Run `serve -s build -l 3001` or kill previous process            |
| Cannot access externally     | Check firewall/security group for port 3000                      |
| Deprecated/vulnerable deps   | Run `npm audit fix` (optional, not blocking for deployment)      |
| Clipboard error (xsel)       | Ignore or run `sudo apt install xsel`                            |
| Compiled with warnings       | Not fatal; can be fixed during code refactor                     |

---

## Disaster Recovery & High Availability

- **Recovery:** Keep a backup of the `build/` folder and your repo.
- **HA:** Use `pm2` or a systemd service to keep `serve` running if needed.

---

## FAQ

- **Is this application free?**  
  Yes, open-source.

- **Can I deploy on any cloud?**  
  Yes, on VM/container with Node.js.

- **Is there an enterprise version?**  
  No, only open-source.

---

## Contact Information

| Name   | Email                      |
|--------|----------------------------|
| Ashutosh Kumar  | [ashutosh.kumar.snaatak@mygurukulam.co](mailto:ashutosh.kumar.snaatak@mygurukulam.co) |

---

## References

|     Description                  | References  
| ---------------------------------| ------------------------------------------------------------------- |
| Frontend API | https://github.com/OT-MICROSERVICES/frontend |
| Javascript heap out of memory error |https://geekflare.com/fix-javascript-heap-out-of-memory-error/ | 
| Create React App Deployment Docs | https://bit.ly/CRA-deploy |

---
