

---

# ⚡ AuthSecure TypeScript Example

> This is a complete **TypeScript (Node.js)** example that shows how to
> securely connect your application with the **AuthSecure API**.
> It includes fully functional **Login**, **Register**, and **License Login** systems ✅

---

## 🚀 Key Features

✅ AuthSecure API Integration (Init / Login / Register / License Login)
✅ Windows Hardware ID (HWID) Support (via PowerShell)
✅ Secure HTTPS Requests (using Axios)
✅ Clean, Class-Based TypeScript Code
✅ Works on Windows, Linux, and macOS
✅ Modern Node.js (ESM) Compatible

---

## 📁 Project Structure

```
authsecure_ts/
│
├── package.json
├── tsconfig.json
└── src/
    ├── authsecure.ts
    └── main.ts
```

---

## ⚙️ Setup Guide (Step-by-Step)

### 🧱 Step 1 — Install Node.js

If you don’t have Node.js installed,
download it from 👉 [https://nodejs.org/en/download](https://nodejs.org/en/download)

Then verify it in your terminal:

```bash
node -v
npm -v
```

---

### 🧰 Step 2 — Create a New Project

Create a folder and navigate into it 👇

```bash
mkdir authsecure_ts && cd authsecure_ts
```

Now run the following commands:

```bash
npm init -y
npm install axios
npm install --save-dev typescript @types/node
npx tsc --init
```

---

### 📦 Step 3 — Update `package.json`

Edit your `package.json` as follows 👇

```json
{
  "name": "authsecure_ts",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "build": "npx tsc",
    "start": "node dist/main.js"
  },
  "dependencies": {
    "axios": "^1.7.7"
  },
  "devDependencies": {
    "@types/node": "^22.9.0",
    "typescript": "^5.7.2"
  }
}
```

---

### ⚙️ Step 4 — Configure `tsconfig.json`

Set your `tsconfig.json` like this 👇

```json
{
  "compilerOptions": {
    "target": "es2020",
    "module": "esnext",
    "moduleResolution": "node",
    "outDir": "./dist",
    "rootDir": "./src",
    "esModuleInterop": true,
    "strict": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

---

## 💻 Source Code

### 🧩 `src/authsecure.ts`

This is the main class that connects and communicates with the AuthSecure API 👇

```ts
import axios from "axios";
import { execSync } from "child_process";

interface AuthConfig {
  name: string;
  ownerid: string;
  secret: string;
  version: string;
}

export class AuthSecure {
  private name: string;
  private ownerid: string;
  private secret: string;
  private version: string;
  private sessionid: string | null = null;
  private BASE_URL = "https://authsecure.shop/post/api.php";

  constructor(config: AuthConfig) {
    this.name = config.name;
    this.ownerid = config.ownerid;
    this.secret = config.secret;
    this.version = config.version;
  }

  private async sendRequest(payload: Record<string, string>) {
    try {
      const params = new URLSearchParams(payload);
      const response = await axios.post(this.BASE_URL, params);
      return response.data;
    } catch (error: any) {
      console.error("❌ HTTP Error:", error.message);
      process.exit(1);
    }
  }

  private getHWID(): string {
    try {
      const output = execSync(
        `powershell -Command "[System.Security.Principal.WindowsIdentity]::GetCurrent().User.Value"`,
        { encoding: "utf8" }
      ).trim();
      return output || "UNKNOWN_HWID";
    } catch {
      return "UNKNOWN_HWID";
    }
  }

  async Init() {
    console.log("Connecting...");
    const resp = await this.sendRequest({
      type: "init",
      name: this.name,
      ownerid: this.ownerid,
      secret: this.secret,
      ver: this.version,
    });

    if (resp.success) {
      this.sessionid = resp.sessionid;
      console.log("✅ Initialized Successfully!");
    } else {
      console.log("❌ Init Failed:", resp.message);
      process.exit(1);
    }
  }

  async Login(username: string, password: string) {
    const resp = await this.sendRequest({
      type: "login",
      sessionid: this.sessionid!,
      username,
      pass: password,
      hwid: this.getHWID(),
      name: this.name,
      ownerid: this.ownerid,
    });

    if (resp.success) {
      console.log("✅ Logged in!");
      this.printUserInfo(resp.info);
    } else {
      console.log("❌ Login Failed:", resp.message);
    }
  }

  async Register(username: string, password: string, license: string) {
    const resp = await this.sendRequest({
      type: "register",
      sessionid: this.sessionid!,
      username,
      pass: password,
      license,
      hwid: this.getHWID(),
      name: this.name,
      ownerid: this.ownerid,
    });

    if (resp.success) {
      console.log("✅ Registered Successfully!");
      this.printUserInfo(resp.info);
    } else {
      console.log("❌ Register Failed:", resp.message);
    }
  }

  async License(license: string) {
    const resp = await this.sendRequest({
      type: "license",
      sessionid: this.sessionid!,
      license,
      hwid: this.getHWID(),
      name: this.name,
      ownerid: this.ownerid,
    });

    if (resp.success) {
      console.log("✅ License Login Successful!");
      this.printUserInfo(resp.info);
    } else {
      console.log("❌ License Login Failed:", resp.message);
    }
  }

  private printUserInfo(info: any) {
    console.log("\n👤 User Info:");
    console.log(" Username:", info.username);
    console.log(" HWID:", info.hwid);
    console.log(" IP:", info.ip);
    if (info.subscriptions) {
      console.log(" Subscriptions:");
      info.subscriptions.forEach((sub: any) =>
        console.log(`  - ${sub.subscription} | Expires: ${sub.expiry}`)
      );
    }
  }
}
```

---

### 🧩 `src/main.ts`

This file provides a simple Command-Line Interface (CLI) for users 👇

```ts
import readline from "readline";
import { AuthSecure } from "./authsecure.js";

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
});

const client = new AuthSecure({
  name: "XD",
  ownerid: "3ezshCmkXrn",
  secret: "7a8bfeb28afcd690812ee5de010a6860",
  version: "1.0",
});

(async () => {
  await client.Init();

  console.log("\n[1] Login\n[2] Register\n[3] License Login\n[4] Exit");
  rl.question("Choose option: ", async (choice) => {
    switch (choice) {
      case "1":
        rl.question("Username: ", (username) => {
          rl.question("Password: ", async (password) => {
            await client.Login(username.trim(), password.trim());
            rl.close();
          });
        });
        break;

      case "2":
        rl.question("Username: ", (username) => {
          rl.question("Password: ", (password) => {
            rl.question("License: ", async (license) => {
              await client.Register(username.trim(), password.trim(), license.trim());
              rl.close();
            });
          });
        });
        break;

      case "3":
        rl.question("License: ", async (license) => {
          await client.License(license.trim());
          rl.close();
        });
        break;

      default:
        console.log("Goodbye!");
        rl.close();
    }
  });
})();
```

---

## 🧮 How to Run

```bash
npm run build
npm start
```

---

## 🖥️ Example Output

```
Connecting...
✅ Initialized Successfully!

[1] Login
[2] Register
[3] License Login
[4] Exit
Choose option: 1
Username: lufy
Password: 12345
✅ Logged in!

👤 User Info:
 Username: lufy
 HWID: S-1-5-21-3116590451-4259102588-3214189088-1001
 IP: 2a09:bac5:3c0b:1a96::2a6:65
 Subscriptions:
  - default | Expires: 1762788300
```

---

## 🧠 Key Highlights

| Feature         | Description                            |
| --------------- | -------------------------------------- |
| 🔒 HTTPS API    | Secure requests using Axios            |
| 💻 HWID         | Generated from Windows User SID        |
| 🧱 TypeScript   | Clean, type-safe, and maintainable     |
| 🔧 Ready to Use | Can be integrated into any app or game |

---

## 🪪 License

**MIT License © 2025 — Created with ❤️ by Lufy**

---


