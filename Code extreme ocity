ocity-full-pro/
│── index.js              # Core Bot
│── database.js           # SQLite Datenbank
│── commands/             # Discord Slash Commands
│   ├── addstreamer.js
│   ├── removestreamer.js
│   ├── liststreamer.js
│── utils/                # Hilfsfunktionen
│   ├── liveCheck.js
│   ├── titleCheck.js
│── web/                  # Web Dashboard
│   ├── server.js
│   ├── public/
│       ├── index.html
│       ├── style.css
│       └── script.js
│── package.json
│── .env.example
│── README.md
require('dotenv').config();
const { Client, GatewayIntentBits } = require("discord.js");
const db = require("./database");
const { checkTwitchLive, checkTikTokLive } = require("./utils/liveCheck");
const { validateTitle } = require("./utils/titleCheck");

const client = new Client({ intents: [GatewayIntentBits.Guilds] });

let liveCache = new Set();
let activity = {};

client.once("ready", () => {
    console.log(`🍊 Orange City PRO Bot online: ${client.user.tag}`);
    startLiveChecks();
});

async function startLiveChecks() {
    setInterval(async () => {
        db.all("SELECT * FROM streamers", [], async (err, rows) => {
            if(err) return console.error(err);

            for(const s of rows) {
                if(s.platform === "twitch") {
                    const res = await checkTwitchLive(s.name);
                    if(res.live && !liveCache.has(s.name)) {
                        if(!validateTitle(res.title)) continue;
                        sendLiveMessage(s.name, res.url, res.title, "Twitch");
                        activity[s.name] = (activity[s.name] || 0) + 1;
                        checkStreamerPlus(s.name);
                        liveCache.add(s.name);
                    }
                    if(!res.live) liveCache.delete(s.name);
                }
                if(s.platform === "tiktok") {
                    const live = await checkTikTokLive(s.name);
                    if(live && !liveCache.has(s.name)) {
                        sendLiveMessage(s.name, `https://tiktok.com/@${s.name}`, "TikTok LIVE", "TikTok");
                        activity[s.name] = (activity[s.name] || 0) + 1;
                        checkStreamerPlus(s.name);
                        liveCache.add(s.name);
                    }
                }
            }
        });
    }, 60000);
}

function sendLiveMessage(name, url, title, platform) {
    const channel = client.channels.cache.get(process.env.CHANNEL_ID);
    if(!channel) return;

    channel.send({
        content: `<@&${process.env.ROLE_ID}> 🔴 ${name} ist LIVE auf ${platform}!\n**${title}**\n${url}`
    });
}

function checkStreamerPlus(name) {
    if(activity[name] >= 5) {
        const guild = client.guilds.cache.first();
        const member = guild.members.cache.find(m => m.user.username.toLowerCase() === name.toLowerCase());
        if(member) member.roles.add(process.env.STREAMERPLUS_ROLE).catch(() => {});
    }
}

client.login(process.env.DISCORD_TOKEN);
const sqlite3 = require("sqlite3").verbose();
const db = new sqlite3.Database("./database.db");

db.run(`CREATE TABLE IF NOT EXISTS streamers (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT,
    platform TEXT
)`);

module.exports = db;
const db = require("../database");

module.exports = {
    name: "addstreamer",
    async execute(interaction) {
        const name = interaction.options.getString("name");
        const platform = interaction.options.getString("platform");

        db.run(`INSERT INTO streamers (name, platform) VALUES (?, ?)`, [name, platform]);
        interaction.reply(`✅ ${name} wurde als ${platform}-Streamer hinzugefügt!`);
    }
};
const fetch = require("node-fetch");
const { WebcastPushConnection } = require("tiktok-live-connector");

async function checkTwitchLive(user) {
    const res = await fetch(`https://api.twitch.tv/helix/streams?user_login=${user}`, {
        headers: {
            "Client-ID": process.env.TWITCH_CLIENT_ID,
            "Authorization": `Bearer ${process.env.TWITCH_TOKEN}`
        }
    });
    const data = await res.json();
    if(data.data.length > 0) return { live: true, title: data.data[0].title, url: `https://twitch.tv/${user}` };
    return { live: false };
}

async function checkTikTokLive(user) {
    try {
        const tiktok = new WebcastPushConnection(user);
        await tiktok.connect();
        return true; // Simplified: erkennt wenn live
    } catch { return false; }
}

module.exports = { checkTwitchLive, checkTikTokLive };
function validateTitle(title) {
    const t = title.toLowerCase();
    return t.includes("ocity") || t.includes("orange city");
}

module.exports = { validateTitle };
const express = require("express");
const db = require("../database");
const app = express();

app.use(express.static("public"));

app.get("/streamers", (req, res) => {
    db.all("SELECT * FROM streamers", [], (err, rows) => {
        if(err) return res.status(500).send(err);
        res.json(rows);
    });
});

app.listen(process.env.WEB_PORT || 3000, () => console.log("🌐 Dashboard läuft"));
DISCORD_TOKEN=DEIN_DISCORD_BOT_TOKEN
CHANNEL_ID=STREAMER_CHANNEL_ID
ROLE_ID=STREAMER_PING_ROLE
STREAMERPLUS_ROLE=STREAMERPLUS_ROLE_ID

TWITCH_CLIENT_ID=DEIN_TWITCH_CLIENT_ID
TWITCH_TOKEN=DEIN_TWITCH_ACCESS_TOKEN

WEB_PORT=3000
