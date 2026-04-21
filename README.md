{
  "name": "joshuatech-bot",
  "version": "2.0.0",
  "description": "JOSHUA TECH WHATSAPP BOT FULL SYSTEM",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "author": "JOSHUA TECH",
  "license": "MIT",
  "dependencies": {
    "@whiskeysockets/baileys": "^6.7.0",
    "qrcode-terminal": "^0.12.0",
    "chalk": "^5.3.0"
  }
}

module.exports = {
    ownerNumber: "255620511416",
    botName: "JOSHUA TECH BOT",
    ownerName: "JOSHUA TECH",
    channelLink: "https://whatsapp.com/channel/0029VakMPmjCxoB3LtbdOv1D",
    prefix: "."
}
const { default: makeWASocket, useMultiFileAuthState } = require("@whiskeysockets/baileys")
const qrcode = require("qrcode-terminal")
const chalk = require("chalk")
const config = require("./config")

async function startBot() {
    const { state, saveCreds } = await useMultiFileAuthState("session")

    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: false
    })

    sock.ev.on("creds.update", saveCreds)

    sock.ev.on("connection.update", (update) => {
        const { connection, qr } = update

        if (qr) {
            console.log(chalk.green("SCAN QR HAPA 👇"))
            qrcode.generate(qr, { small: true })
        }

        if (connection === "open") {
            console.log(chalk.blue(`✅ ${config.botName} IMEUNGANISHWA`))
        }

        if (connection === "close") {
            console.log(chalk.red("❌ CONNECTION IMEFUNGWA, INAANZA UPYA..."))
            startBot()
        }
    })

    sock.ev.on("messages.upsert", async ({ messages }) => {
        const msg = messages[0]
        if (!msg.message) return

        const text = msg.message.conversation || msg.message.extendedTextMessage?.text
        const from = msg.key.remoteJid

        if (!text) return

        const prefix = config.prefix
        const command = text.startsWith(prefix) ? text.slice(1).split(" ")[0] : null

        if (!command) return

        console.log(chalk.yellow(`COMMAND: ${command}`))

        // MENU
        if (command === "menu") {
            await sock.sendMessage(from, {
                text: `👑 ${config.botName} 👑

📌 COMMANDS:
.menu
.owner
.ping
.channel

💻 Powered by ${config.ownerName}`
            })
        }

        // OWNER
        if (command === "owner") {
            await sock.sendMessage(from, {
                text: `👤 OWNER: ${config.ownerName}
📞 ${config.ownerNumber}`
            })
        }

        // PING
        if (command === "ping") {
            await sock.sendMessage(from, {
                text: "⚡ BOT IPO ACTIVE NA SPEED KALI!"
            })
        }

        // CHANNEL
        if (command === "channel") {
            await sock.sendMessage(from, {
                text: `📢 FOLLOW CHANNEL:
${config.channelLink}`
            })
        }

        // AUTO REPLY
        if (text.toLowerCase() === "hi" || text.toLowerCase() === "hello") {
            await sock.sendMessage(from, {
                text: "👋 Karibu JOSHUA TECH BOT, andika .menu uone commands"
            })
        }
    })
}

startBot()

npm install
npm start
