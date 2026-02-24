

# 🚀 Castorice-Baileys

<div align="center">

  

</div>
---

📦 Installation

```npm install @kingard/baileys```

atau

```yarn add @kingard/baileys```


---

🔌 Basic Usage

```const { 
  default: makeWASocket, 
  DisconnectReason, 
  useMultiFileAuthState 
} = require("@kingard/baileys");

const { Boom } = require("@hapi/boom");

async function connectToWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState("auth_info");

    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: true
    });

    sock.ev.on("connection.update", (update) => {
        const { connection, lastDisconnect } = update;

        if (connection === "close") {
            const shouldReconnect =
                (lastDisconnect.error)?.output?.statusCode !== DisconnectReason.loggedOut;

            if (shouldReconnect) connectToWhatsApp();
        }

        if (connection === "open") {
            console.log("✅ Connected to WhatsApp!");
        }
    });

    sock.ev.on("messages.upsert", async (m) => {
        const msg = m.messages[0];

        if (!msg.key.fromMe && msg.message) {
            await sock.sendMessage(
                msg.key.remoteJid,
                { text: "Hello from Castorice-Baileys 👑" }
            );
        }
    });

    sock.ev.on("creds.update", saveCreds);
}

connectToWhatsApp();
```

---

💾 Save Session

const { useMultiFileAuthState } = require("@kingard/baileys");

const { state, saveCreds } = await useMultiFileAuthState("auth_info");
const sock = makeWASocket({ auth: state });

sock.ev.on("creds.update", saveCreds);


---

📨 Send Message

await sock.sendMessage(jid, { 
  text: "Hello World 🌍" 
});


---

👥 Group Example

const group = await sock.groupCreate("Castorice Group 👑", [
  "1234567890@s.whatsapp.net"
]);

await sock.sendMessage(group.id, { text: "Welcome!" });


---

🐛 Bug Report

👉 https://github.com/kingard888/Castorice-Baileys/issues

---