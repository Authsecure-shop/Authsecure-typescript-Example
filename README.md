बिलकुल ❤️
यह रहा इसका पूरा **English README का हिंदी अनुवाद (GitHub-ready format)** —
साफ-सुथरा और पब्लिक यूज़ के लिए तैयार 👇

---

# ⚡ AuthSecure TypeScript Example (अंग्रेज़ी संस्करण का हिंदी रूप)

> यह एक पूरा **TypeScript (Node.js)** उदाहरण है जो दिखाता है कि
> आप अपने ऐप को **AuthSecure API** से सुरक्षित रूप से कैसे जोड़ सकते हैं।
> इसमें Login, Register, और License Login पूरी तरह तैयार हैं ✅

---

## 🚀 मुख्य विशेषताएं

✅ AuthSecure API इंटीग्रेशन (Init / Login / Register / License Login)
✅ Windows Hardware ID (HWID) सपोर्ट (PowerShell के माध्यम से)
✅ सुरक्षित HTTPS रिक्वेस्ट्स (Axios का उपयोग करके)
✅ क्लास-बेस्ड साफ-सुथरा TypeScript कोड
✅ Windows, Linux, macOS पर काम करता है
✅ आधुनिक Node.js (ESM) सपोर्ट

---

## 📁 प्रोजेक्ट स्ट्रक्चर

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

## ⚙️ सेटअप गाइड (Step-by-Step)

### 🧱 Step 1 — Node.js इंस्टॉल करें

अगर आपके सिस्टम में Node.js नहीं है,
तो इसे यहाँ से डाउनलोड करें 👉 [https://nodejs.org/en/download](https://nodejs.org/en/download)

फिर टर्मिनल में चेक करें:

```bash
node -v
npm -v
```

---

### 🧰 Step 2 — नया प्रोजेक्ट बनाएँ

अब नया फोल्डर बनाएं और उसमें जाएं 👇

```bash
mkdir authsecure_ts && cd authsecure_ts
```

अब यह कमांड्स चलाएँ:

```bash
npm init -y
npm install axios
npm install --save-dev typescript @types/node
npx tsc --init
```

---

### 📦 Step 3 — `package.json` अपडेट करें

`package.json` को इस तरह एडिट करें 👇

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

### ⚙️ Step 4 — `tsconfig.json` सेट करें

`tsconfig.json` को इस तरह सेट करें 👇

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

## 💻 कोड फाइल्स

### 🧩 `src/authsecure.ts`

यह मुख्य क्लास है जो AuthSecure API से कनेक्शन बनाती है 👇

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
    console.log("\n👤 यूज़र जानकारी:");
    console.log(" यूज़रनेम:", info.username);
    console.log(" HWID:", info.hwid);
    console.log(" IP:", info.ip);
    if (info.subscriptions) {
      console.log(" सब्सक्रिप्शन:");
      info.subscriptions.forEach((sub: any) =>
        console.log(`  - ${sub.subscription} | Expires: ${sub.expiry}`)
      );
    }
  }
}
```

---

### 🧩 `src/main.ts`

यह फाइल CLI (कमांड लाइन) के जरिए यूज़र इंटरैक्शन के लिए है 👇

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
  rl.question("एक विकल्प चुनें: ", async (choice) => {
    switch (choice) {
      case "1":
        rl.question("यूज़रनेम: ", (username) => {
          rl.question("पासवर्ड: ", async (password) => {
            await client.Login(username.trim(), password.trim());
            rl.close();
          });
        });
        break;

      case "2":
        rl.question("यूज़रनेम: ", (username) => {
          rl.question("पासवर्ड: ", (password) => {
            rl.question("लाइसेंस: ", async (license) => {
              await client.Register(username.trim(), password.trim(), license.trim());
              rl.close();
            });
          });
        });
        break;

      case "3":
        rl.question("लाइसेंस: ", async (license) => {
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

## 🧮 प्रोजेक्ट रन करने का तरीका

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
एक विकल्प चुनें: 1
यूज़रनेम: lufy
पासवर्ड: 12345
✅ Logged in!

👤 यूज़र जानकारी:
 यूज़रनेम: lufy
 HWID: S-1-5-21-3116590451-4259102588-3214189088-1001
 IP: 2a09:bac5:3c0b:1a96::2a6:65
 सब्सक्रिप्शन:
  - default | Expires: 1762788300
```

---

## 🧠 महत्वपूर्ण बातें

| विषय            | विवरण                                             |
| --------------- | ------------------------------------------------- |
| 🔒 HTTPS API    | Axios से सुरक्षित रिक्वेस्ट                       |
| 💻 HWID         | Windows User SID से HWID जेनरेट होता है           |
| 🧱 TypeScript   | साफ, Type-safe और Maintainable                    |
| 🔧 Ready to Use | किसी भी App या Game में इंटीग्रेट किया जा सकता है |

---

## 🪪 लाइसेंस

**MIT License © 2025 — Created with ❤️ by Lufy**

---

क्या चाहो तो मैं इसका **ZIP Template (GitHub-ready)** बना दूँ
जिसमें यह पूरा कोड + README + सेटअप पहले से होगा (बस `npm start` चलाना होगा)?
