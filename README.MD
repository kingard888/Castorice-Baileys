<div align="center">

# 🚀 WhatsApp Web API - Baileys

![WhatsApp Web API](https://img.shields.io/badge/WhatsApp%20Web%20API-Baileys-25D366?style=for-the-badge&logo=whatsapp&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![WebSocket](https://img.shields.io/badge/WebSocket-010101?style=for-the-badge&logo=websocket&logoColor=white)

<br>

[![npm version](https://img.shields.io/npm/v/@owensdev1/baileys?style=flat-square&color=CB3837&logo=npm)](https://www.npmjs.com/package/@owensdev1/baileys)
[![npm downloads](https://img.shields.io/npm/dm/@owensdev1/baileys?style=flat-square&color=00d26a)](https://www.npmjs.com/package/@owensdev1/baileys)
[![GitHub stars](https://img.shields.io/github/stars/owensdev1/baileys?style=flat-square&color=FFB400&logo=github)](https://github.com/owensdev1/baileys)
[![GitHub license](https://img.shields.io/github/license/owensdev1/baileys?style=flat-square&color=e74c3c)](LICENSE)

<br>

> **⚡ A powerful WebSocket-based TypeScript library untuk WhatsApp Web API**

[📖 Dokumentasi](#-dokumentasi) · [🚀 Quick Start](#-quick-start) · [💬 Support](#-support) · [🤝 Kontribusi](#-kontribusi)

</div>

---

## ✨ Fitur Unggulan

<table>
<tr>
<td align="center" width="25%">

### 🔐 Multi-Device
**Sinkronisasi Sempurna**

Dukungan multi-device dengan sinkronisasi mulus di semua platform

</td>
<td align="center" width="25%">

### 📱 QR & Pairing
**Fleksibel Koneksi**

Support QR code scanning & pairing code authentication

</td>
<td align="center" width="25%">

### 🎨 Rich Messages
**Interaktif Penuh**

Buttons, polls, location, media attachments & custom messages

</td>
<td align="center" width="25%">

### 🔄 Real-time Events
**WebSocket Live**

Real-time messaging dengan instant delivery notifications

</td>
</tr>
<tr>
<td align="center" width="25%">

### 👥 Group Management
**Admin Lengkap**

Full group administration & member management capabilities

</td>
<td align="center" width="25%">

### 🔒 Privacy Controls
**Keamanan Data**

Block users, privacy settings, comprehensive security management

</td>
<td align="center" width="25%">

### 📊 Message History
**Chat History Access**

Fetch & manage chat history dengan powerful search capabilities

</td>
<td align="center" width="25%">

### 🎯 Custom Functions
**Highly Extensible**

Plugin support & custom event handlers untuk kebutuhan custom

</td>
</tr>
</table>

---

## 🚀 Quick Start

### 📦 Instalasi

```bash
# 🧶 Menggunakan Yarn (recommended)
yarn add @owensdev1/baileys

# 📦 Menggunakan NPM
npm install @owensdev1/baileys
```

### 🔌 Koneksi Dasar

```javascript
const { default: makeWASocket, DisconnectReason, useMultiFileAuthState } = require("@owensdev1/baileys");
const { Boom } = require('@hapi/boom');

async function connectToWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState('auth_info_baileys');
    
    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: true
    });

    // 🔗 Connection Events
    sock.ev.on('connection.update', (update) => {
        const { connection, lastDisconnect } = update;
        
        if(connection === 'close') {
            const shouldReconnect = (lastDisconnect.error as Boom)?.output?.statusCode !== DisconnectReason.loggedOut;
            console.log('❌ Koneksi ditutup. Reconnect:', shouldReconnect);
            
            if(shouldReconnect) {
                connectToWhatsApp();
            }
        } else if(connection === 'open') {
            console.log('✅ Berhasil terhubung ke WhatsApp!');
        }
    });

    // 📨 Message Events
    sock.ev.on('messages.upsert', async (m) => {
        const msg = m.messages[0];
        
        if (!msg.key.fromMe && msg.message) {
            console.log('📩 Pesan baru:', JSON.stringify(m, undefined, 2));
            
            // 🤖 Echo response
            await sock.sendMessage(msg.key.remoteJid, { 
                text: 'Hello! Terima kasih pesanmu 👋' 
            });
        }
    });

    sock.ev.on('creds.update', saveCreds);
}

connectToWhatsApp();
```

---

## 📖 Dokumentasi

### 🔌 Setup & Koneksi

#### 📱 QR Code Connection

```javascript
const { default: makeWASocket, Browsers } = require("@owensdev1/baileys");

const sock = makeWASocket({
    browser: Browsers.ubuntu('My Awesome Bot'),
    printQRInTerminal: true
});
```

> **💡 Tip:** Customize browser name menggunakan `Browsers` constant untuk pengalaman lebih personal.

#### 🔢 Pairing Code Connection

> [!IMPORTANT]
> **Pairing Code** memungkinkan koneksi WhatsApp Web **TANPA QR Code!**  
> Format nomor: country code + number (tanpa +, (), atau -)

```javascript
const sock = makeWASocket({
    printQRInTerminal: false // WAJIB false untuk pairing code
});

// Standard Pairing
if (!sock.authState.creds.registered) {
    const number = '1234567890'; // Nomor WhatsApp
    const code = await sock.requestPairingCode(number);
    console.log('🔑 Pairing Code:', code);
}

// Custom Pairing (8 digit/karakter)
if (!sock.authState.creds.registered) {
    const customPair = "BAILEYS01";
    const number = '1234567890';
    const code = await sock.requestPairingCode(number, customPair);
    console.log('🔑 Custom Code:', code);
}
```

#### 💾 Simpan & Restore Sesi

```javascript
const { useMultiFileAuthState } = require("@owensdev1/baileys");

const { state, saveCreds } = await useMultiFileAuthState('auth_info_baileys');

const sock = makeWASocket({ auth: state });

// 🔄 Auto-save credentials saat update
sock.ev.on('creds.update', saveCreds);
```

> [!WARNING]
> **KRITIS:** Selalu simpan auth keys saat update untuk memastikan message delivery yang stabil!

---

### 📨 Pengiriman Pesan

#### 📝 Text Message

```javascript
await sock.sendMessage(jid, { 
    text: 'Hello World! 🌍' 
});
```

#### 🔘 Button Message

```javascript
await sock.sendMessage(jid, {
    text: "Silakan pilih opsi di bawah:",
    footer: "© 2025 Baileys Bot",
    buttons: [
        {
            buttonId: 'btn1',
            buttonText: { displayText: '✅ Opsi 1' },
            type: 1
        },
        {
            buttonId: 'btn2',
            buttonText: { displayText: '❌ Opsi 2' },
            type: 1
        }
    ],
    headerType: 1
});
```

#### 🎯 Interactive Menu dengan Flow

```javascript
await sock.sendMessage(jid, {
    text: "📋 Menu Interaktif",
    footer: "© 2025 Bot",
    buttons: [
        {
            buttonId: 'menu',
            buttonText: { displayText: '📋 Tampilkan Menu' },
            type: 4,
            nativeFlowInfo: {
                name: 'single_select',
                paramsJson: JSON.stringify({
                    title: 'Pilih Opsi',
                    sections: [{
                        title: 'Opsi Tersedia',
                        highlight_label: '⭐',
                        rows: [
                            {
                                header: 'OPSI 1',
                                title: 'Pilihan Pertama',
                                description: 'Deskripsi untuk opsi 1',
                                id: 'opt1'
                            },
                            {
                                header: 'OPSI 2', 
                                title: 'Pilihan Kedua',
                                description: 'Deskripsi untuk opsi 2',
                                id: 'opt2'
                            }
                        ]
                    }]
                })
            }
        }
    ]
});
```

#### 📋 Poll/Vote Message

```javascript
await sock.sendMessage(jid, {
    poll: {
        name: 'Warna favorit Anda? 🎨',
        values: ['🔴 Merah', '🔵 Biru', '🟢 Hijau', '🟡 Kuning'],
        selectableCount: 1
    }
});
```

#### 🎬 Media Messages

**🖼️ Image:**
```javascript
await sock.sendMessage(jid, {
    image: { url: './image.jpg' },
    caption: '📸 Gambar yang indah!'
});
```

**🎥 Video:**
```javascript
await sock.sendMessage(jid, {
    video: { url: './video.mp4' },
    caption: '🎬 Video keren!',
    ptv: false
});
```

**🎵 Audio:**
```javascript
await sock.sendMessage(jid, {
    audio: { url: './audio.mp3' },
    mimetype: 'audio/mp4'
});
```

---

### 👥 Manajemen Grup

#### 🆕 Buat Grup

```javascript
const group = await sock.groupCreate('🎉 Grup Keren', [
    '1234567890@s.whatsapp.net',
    '0987654321@s.whatsapp.net'
]);

console.log('✅ Grup dibuat:', group.id);
await sock.sendMessage(group.id, { text: '👋 Selamat datang!' });
```

#### 👤 Add/Remove Members

```javascript
await sock.groupParticipantsUpdate(
    groupJid,
    ['1234567890@s.whatsapp.net'],
    'add'  // 'remove', 'promote', 'demote'
);
```

#### ⚙️ Ubah Setting Grup

```javascript
// 📝 Ubah nama
await sock.groupUpdateSubject(groupJid, '🚀 Nama Baru');

// 📋 Ubah deskripsi
await sock.groupUpdateDescription(groupJid, '📝 Deskripsi baru');

// 🔒 Hanya admin kirim pesan
await sock.groupSettingUpdate(groupJid, 'announcement');

// ✅ Semua orang kirim pesan
await sock.groupSettingUpdate(groupJid, 'not_announcement');
```

---

### 🔒 Privacy Controls

#### 🚫 Block/Unblock Users

```javascript
// 🔴 Block
await sock.updateBlockStatus(jid, 'block');

// 🟢 Unblock  
await sock.updateBlockStatus(jid, 'unblock');
```

#### ⚙️ Privacy Settings

```javascript
// 👁️ Last Seen Privacy
await sock.updateLastSeenPrivacy('contacts');  // 'all', 'contacts', 'none'

// 🌐 Online Status
await sock.updateOnlinePrivacy('all');  // 'all', 'match_last_seen'

// 📸 Profile Picture
await sock.updateProfilePicturePrivacy('contacts');

// 📹 Status Privacy
await sock.updateStatusPrivacy('contacts');

// ✓ Read Receipts
await sock.updateReadReceiptsPrivacy('all');  // 'all', 'none'
```

---

### 📊 Data Store Implementation

```javascript
const { makeInMemoryStore } = require("@owensdev1/baileys");

const store = makeInMemoryStore({});

// 📂 Load dari file
store.readFromFile('./baileys_store.json');

// 💾 Auto-save setiap 10 detik
setInterval(() => {
    store.writeToFile('./baileys_store.json');
}, 10_000);

// 🔗 Bind ke socket
const sock = makeWASocket({});
store.bind(sock.ev);

// 📖 Akses data
sock.ev.on('chats.upsert', () => {
    console.log('💬 Chats:', store.chats.all());
});
```

---

### 🐛 Debug Mode

```javascript
const sock = makeWASocket({
    logger: P({ level: 'debug' })
});

// 📡 Custom WebSocket Events
sock.ws.on('CB:edge_routing', (node) => {
    console.log('📡 Edge routing:', node);
});

// 🎯 Specific listeners
sock.ws.on('CB:edge_routing,id:abcd', (node) => {
    console.log('🎯 Message:', node);
});
```

---

## 💬 Support

<div align="center">

| 📞 Support | 💬 Komunitas | 🐛 Issues |
|:---:|:---:|:---:|
| **WhatsApp Direct** | **WhatsApp Group** | **GitHub Issues** |
| [Chat: 6285358977442](https://wa.me/6285358977442) | [Join Group](https://chat.whatsapp.com) | [Report Bug](https://github.com/owensdev1/baileys/issues) |

</div>

---

## 🤝 Kontribusi

Kami sangat menghargai setiap kontribusi! 🙌

### Cara Berkontribusi:

```
🍴 Fork → 🌟 Branch → 💻 Code → 📤 Push → 🔄 Pull Request
```

1. **Fork** repository ini
2. **Buat branch** fitur (`git checkout -b feature/AmazingFeature`)
3. **Commit** perubahan (`git commit -m 'Add AmazingFeature'`)
4. **Push** ke branch (`git push origin feature/AmazingFeature`)
5. **Buka Pull Request**

---

## 📄 Lisensi

<div align="center">

**GPL v3 License**

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg?style=for-the-badge&logo=gnu&logoColor=white)](https://www.gnu.org/licenses/gpl-3.0)

</div>

---

## 🙏 Terima Kasih

<table>
<tr>
<td align="center">

### 👑 Owensdev1
**Project Owner**  
*Lead Developer*

</td>
<td align="center">

### 🛠️ KyzRyzz
**Technical Support**  
*Community Lead*

</td>
<td align="center">

### 🧪 Yousoo
**Development**  
*Testing & QA*

</td>
<td align="center">

### 💻 Valzy
**Base Source**  
*Core Architecture*

</td>
</tr>
</table>

---

## ⭐ Jangan Lupa Star!

**Jika project ini membantu Anda, berikan ⭐ star!**

[![GitHub stars](https://img.shields.io/github/stars/owensdev1/baileys?style=social&label=Stars)](https://github.com/owensdev1/baileys/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/owensdev1/baileys?style=social&label=Forks)](https://github.com/owensdev1/baileys/network)
[![GitHub watchers](https://img.shields.io/github/watchers/owensdev1/baileys?style=social&label=Watch)](https://github.com/owensdev1/baileys/watchers)

---

<div align="center">

**💻 Made with ❤️ and ☕ by the Community**

**© 2025 Baileys WhatsApp API - Building the Future of WhatsApp Automation**

</div>