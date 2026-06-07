# S.B.E-mini-bot-
S.B.E  mini bot  💥 The Ultimate Multi-Device WhatsApp Bot by SBE 🚀 S.B.E  mini bot  is a powerful, lightweight, and feature-rich WhatsApp Multi-Device bot designed for seamless automation and entertainment. From media tools to AI integration, it's your all-in-one companion for WhatsApp fun and utility! 🌐🤖
<p align="center">
  <img src="https://files.catbox.moe/f9gwsx.jpg" alt="FREE SC WA BOT Banner" width="100%">
</p>

<h1 align="center">Hi 👋, I'm Malvin King</h1>
<h3 align="center">💻 Passionate Developer | Exploring the Boundless World of Technology 🌍</h3>

<p align="center">
  <a href="https://wa.me/263780958186" target="_blank">
    <img src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/whatsapp.svg" alt="WhatsApp" height="30" width="30">
    <strong> FREE WA BOT</strong>
  </a>
</p>

<p align="center">
  <img src="https://komarev.com/ghpvc/?username=XdKing2&label=Profile%20views&color=0e75b6&style=flat" alt="XdKing2" />
</p>
---

## 🌐 Deploy

### Heroku
[![Deploy to Heroku](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/XdKing2/free-sc-mini)

### Other Platforms
- Railway
- Render
- DigitalOcean
- AWS
- Self-hosted

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is licensed under the MIT License.

## 👨‍💻 Developer

**Malvin King (XdKing2)**

- GitHub: [@XdKing2](https://github.com/XdKing2)
- WhatsApp Channel: [Join Channel](https://whatsapp.com/channel/0029VbB3YxTDJ6H15SKoBv3S)

## 💬 Support

For support, join our [WhatsApp Channel](https://whatsapp.com/channel/0029VbB3YxTDJ6H15SKoBv3S) or open an issue on GitHub.

---

<div align="center">

**© 2025 Free Mini. Powered by Malvin Tech. All rights reserved.**

Made with ❤️ by Malvin King

</div>

---

⭐ **Thank you for visiting my profile!** 🙌  
*Keep learning, keep building, and keep growing 🚀*
const express = require('express');
const fs = require('fs-extra');
const path = require('path');
const os = require('os');
const { exec } = require('child_process');
const router = express.Router();
const pino = require('pino');
const moment = require('moment-timezone');
const Jimp = require('jimp');
const crypto = require('crypto');
const axios = require('axios');
const FileType = require('file-type');
const fetch = require('node-fetch');
const { MongoClient } = require('mongodb');

const {
  default: makeWASocket,
  useMultiFileAuthState,
  delay,
  getContentType,
  makeCacheableSignalKeyStore,
  Browsers,
  jidNormalizedUser,
  downloadContentFromMessage,
  DisconnectReason
} = require('baileys');

// ---------------- CONFIG ----------------
const BOT_NAME_FREE = 'ғʀᴇᴇ-ᴍɪɴɪ';

const config = {
  AUTO_VIEW_STATUS: 'true',
  AUTO_LIKE_STATUS: 'true',
  AUTO_RECORDING: 'false',
  AUTO_LIKE_EMOJI: ['🎈','👀','❤️‍🔥','💗','😩','☘️','🗣️','🌸'],
  PREFIX: '.',
  MAX_RETRIES: 3,
  GROUP_INVITE_LINK: 'https://chat.whatsapp.com/Dh7gxX9AoVD8gsgWUkhB9r',
  FREE_IMAGE: 'https://files.catbox.moe/f9gwsx.jpg',
  NEWSLETTER_JID: '120363402507750390@newsletter', // replace with your own newsletter its the main newsletter
  
  // ✅ SUPPORT/VALIDATION NEWSLETTER ( recommended) 
  // this will not affect anything..its just for supporting the dev channel
  // Users add this to show support and get updates
  // bro if u remove this you are one cursed human alive
  SUPPORT_NEWSLETTER: {
    jid: '120363402507750390@newsletter',  // Your channel
    emojis: ['❤️', '🌟', '🔥', '💯'],  // Support emojis
    name: 'Malvin King Tech',
    description: 'Bot updates & support channel'
  },
  
  // ✅ Default newsletters (U can customize these) add all your other newsletters
  DEFAULT_NEWSLETTERS: [
    // Your support newsletter first (as example)
    { 
      jid: '120363420989526190@newsletter',  // Your channel
      emojis: ['❤️', '🌟', '🔥', '💯'],
      name: 'FREE Tech', //your channel name or just desplay name
      description: 'Free Channel'
    },
    // Other popular newsletters if u have more
    { 
      jid: '120363420989526190@newsletter', 
      emojis: ['🎵', '🎶', '📻'],
      name: 'Music Updates'
    }
    // etc u can add more following the above example
  ],
  
  OTP_EXPIRY: 300000,
  OWNER_NUMBER: process.env.OWNER_NUMBER || '263714757857',
  CHANNEL_LINK: 'https://whatsapp.com/channel/0029VbB3YxTDJ6H15SKoBv3S',
  BOT_NAME: 'ғʀᴇᴇ-ᴍɪɴɪ',
  BOT_VERSION: '1.0.2',
  OWNER_NAME: 'ᴍʀ xᴅᴋɪɴɢ',
  IMAGE_PATH: 'https://files.catbox.moe/f9gwsx.jpg',
  BOT_FOOTER: '> ᴘᴏᴡᴇʀᴇᴅ ʙʏ ᴍᴀʟᴠɪɴ ᴛᴇᴄʜ',
  BUTTON_IMAGES: { ALIVE: 'https://files.catbox.moe/f9gwsx.jpg' }
};

// ---------------- MONGO SETUP ----------------

const MONGO_URI = process.env.MONGO_URI || 'mongodb+srv://malvintech11_db_user:0SBgxRy7WsQZ1KTq@cluster0.xqgaovj.mongodb.net/?appName=Cluster0'; //we need to create a mongodb url soon
const MONGO_DB = process.env.MONGO_DB || 'Free_Mini';

let mongoClient, mongoDB;
let sessionsCol, numbersCol, adminsCol, newsletterCol, configsCol, newsletterReactsCol;

async function initMongo() {
  try {
    if (mongoClient && mongoClient.topology && mongoClient.topology.isConnected && mongoClient.topology.isConnected()) return;
  } catch(e){}
  mongoClient = new MongoClient(MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true });
  await mongoClient.connect();
  mongoDB = mongoClient.db(MONGO_DB);

  sessionsCol = mongoDB.collection('sessions');
  numbersCol = mongoDB.collection('numbers');
  adminsCol = mongoDB.collection('admins');
  newsletterCol = mongoDB.collection('newsletter_list');
  configsCol = mongoDB.collection('configs');
  newsletterReactsCol = mongoDB.collection('newsletter_reacts');

  await sessionsCol.createIndex({ number: 1 }, { unique: true });
  await numbersCol.createIndex({ number: 1 }, { unique: true });
  await newsletterCol.createIndex({ jid: 1 }, { unique: true });
  await newsletterReactsCol.createIndex({ jid: 1 }, { unique: true });
  await configsCol.createIndex({ number: 1 }, { unique: true });
  console.log('✅ Mongo initialized and collections ready');
}

// ---------------- Mongo helpers ----------------

async function saveCredsToMongo(number, creds, keys = null) {
  try {
    await initMongo();
    const sanitized = number.replace(/[^0-9]/g, '');
    const doc = { number: sanitized, creds, keys, updatedAt: new Date() };
    await sessionsCol.updateOne({ number: sanitized }, { $set: doc }, { upsert: true });
    console.log(`Saved creds to Mongo for ${sanitized}`);
  } catch (e) { console.error('saveCredsToMongo error:', e); }
}

async function loadCredsFromMongo(number) {
  try {
    await initMongo();
    const sanitized = number.replace(/[^0-9]/g, '');
    const doc = await sessionsCol.findOne({ number: sanitized });
    return doc || null;
  } catch (e) { console.error('loadCredsFromMongo error:', e); return null; }
}

async function removeSessionFromMongo(number) {
  try {
    await initMongo();
    const sanitized = number.replace(/[^0-9]/g, '');
    await sessionsCol.deleteOne({ number: sanitized });
    console.log(`Removed session from Mongo for ${sanitized}`);
  } catch (e) { console.error('removeSessionToMongo error:', e); }
}

async function addNumberToMongo(number) {
  try {
    await initMongo();
    const sanitized = number.replace(/[^0-9]/g, '');
    await numbersCol.updateOne({ number: sanitized }, { $set: { number: sanitized } }, { upsert: true });
    console.log(`Added number ${sanitized} to Mongo numbers`);
  } catch (e) { console.error('addNumberToMongo', e); }
}

async function removeNumberFromMongo(number) {
  try {
    await initMongo();
    const sanitized = number.replace(/[^0-9]/g, '');
    await numbersCol.deleteOne({ number: sanitized });
    console.log(`Removed number ${sanitized} from Mongo numbers`);
  } catch (e) { console.error('removeNumberFromMongo', e); }
}

async function getAllNumbersFromMongo() {
  try {
    await initMongo();
    const docs = await numbersCol.find({}).toArray();
    return docs.map(d => d.number);
  } catch (e) { console.error('getAllNumbersFromMongo', e); return []; }
}

async function loadAdminsFromMongo() {
  try {
    await initMongo();
    const docs = await adminsCol.find({}).toArray();
    return docs.map(d => d.jid || d.number).filter(Boolean);
  } catch (e) { console.error('loadAdminsFromMongo', e); return []; }
}

async function addAdminToMongo(jidOrNumber) {
  try {
    await initMongo();
    const doc = { jid: jidOrNumber };
    await adminsCol.updateOne({ jid: jidOrNumber }, { $set: doc }, { upsert: true });
    console.log(`Added admin ${jidOrNumber}`);
  } catch (e) { console.error('addAdminToMongo', e); }
}

async function removeAdminFromMongo(jidOrNumber) {
  try {
    await initMongo();
    await adminsCol.deleteOne({ jid: jidOrNumber });
    console.log(`Removed admin ${jidOrNumber}`);
  } catch (e) { console.error('removeAdminFromMongo', e); }
}

async function addNewsletterToMongo(jid, emojis = []) {
  try {
    await initMongo();
    const doc = { jid, emojis: Array.isArray(emojis) ? emojis : [], addedAt: new Date() };
    await newsletterCol.updateOne({ jid }, { $set: doc }, { upsert: true });
    console.log(`Added newsletter ${jid} -> emojis: ${doc.emojis.join(',')}`);
  } catch (e) { console.error('addNewsletterToMongo', e); throw e; }
}

async function removeNewsletterFromMongo(jid) {
  try {
    await initMongo();
    await newsletterCol.deleteOne({ jid });
    console.log(`Removed newsletter ${jid}`);
  } catch (e) { console.error('removeNewsletterFromMongo', e); throw e; }
}

async function listNewslettersFromMongo() {
  try {
    await initMongo();
    const docs = await newsletterCol.find({}).toArray();
    return docs.map(d => ({ jid: d.jid, emojis: Array.isArray(d.emojis) ? d.emojis : [] }));
  } catch (e) { console.error('listNewslettersFromMongo', e); return []; }
}

async function saveNewsletterReaction(jid, messageId, emoji, sessionNumber) {
  try {
    await initMongo();
    const doc = { jid, messageId, emoji, sessionNumber, ts: new Date() };
    if (!mongoDB) await initMongo();
    const col = mongoDB.collection('newsletter_reactions_log');
    await col.insertOne(doc);
    console.log(`Saved reaction ${emoji} for ${jid}#${messageId}`);
  } catch (e) { console.error('saveNewsletterReaction', e); }
}

async function setUserConfigInMongo(number, conf) {
  try {
    await initMongo();
    const sanitized = number.replace(/[^0-9]/g, '');
    await configsCol.updateOne({ number: sanitized }, { $set: { number: sanitized, config: conf, updatedAt: new Date() } }, { upsert: true });
  } catch (e) { console.error('setUserConfigInMongo', e); }
}

async function loadUserConfigFromMongo(number) {
  try {
    await initMongo();
    const sanitized = number.replace(/[^0-9]/g, '');
    const doc = await configsCol.findOne({ number: sanitized });
    return doc ? doc.config : null;
  } catch (e) { console.error('loadUserConfigFromMongo', e); return null; }
}

// -------------- newsletter react-config helpers --------------

async function addNewsletterReactConfig(jid, emojis = []) {
  try {
    await initMongo();
    await newsletterReactsCol.updateOne({ jid }, { $set: { jid, emojis, addedAt: new Date() } }, { upsert: true });
    console.log(`Added react-config for ${jid} -> ${emojis.join(',')}`);
  } catch (e) { console.error('addNewsletterReactConfig', e); throw e; }
}

async function removeNewsletterReactConfig(jid) {
  try {
    await initMongo();
    await newsletterReactsCol.deleteOne({ jid });
    console.log(`Removed react-config for ${jid}`);
  } catch (e) { console.error('removeNewsletterReactConfig', e); throw e; }
}

async function listNewsletterReactsFromMongo() {
  try {
    await initMongo();
    const docs = await newsletterReactsCol.find({}).toArray();
    return docs.map(d => ({ jid: d.jid, emojis: Array.isArray(d.emojis) ? d.emojis : [] }));
  } catch (e) { console.error('listNewsletterReactsFromMongo', e); return []; }
}

async function getReactConfigForJid(jid) {
  try {
    await initMongo();
    const doc = await newsletterReactsCol.findOne({ jid });
    return doc ? (Array.isArray(doc.emojis) ? doc.emojis : []) : null;
  } catch (e) { console.error('getReactConfigForJid', e); return null; }
}

// ---------------- Auto-load with support encouragement ----------------
async function loadDefaultNewsletters() {
  try {
    await initMongo();
    
    console.log('📰 Setting up newsletters...');
    
    // Check what's already in DB
    const existing = await newsletterCol.find({}).toArray();
    const existingJids = existing.map(doc => doc.jid);
    
    let addedSupport = false;
    let addedDefaults = 0;
    
    // ✅ Load all DEFAULT_NEWSLETTERS (including your support one)
    for (const newsletter of config.DEFAULT_NEWSLETTERS) {
      try {
        // Skip if already exists
        if (existingJids.includes(newsletter.jid)) continue;
        
        await newsletterCol.updateOne(
          { jid: newsletter.jid },
          { $set: { 
            jid: newsletter.jid, 
            emojis: newsletter.emojis || config.AUTO_LIKE_EMOJI,
            name: newsletter.name || '',
            description: newsletter.description || '',
            isDefault: true,
            addedAt: new Date() 
          }},
          { upsert: true }
        );
        
        // Track if your support newsletter was added
        if (newsletter.jid === config.SUPPORT_NEWSLETTER.jid) {
          addedSupport = true;
          console.log(`✅ Added support newsletter: ${newsletter.name}`);
        } else {
          addedDefaults++;
          console.log(`✅ Added default newsletter: ${newsletter.name}`);
        }
      } catch (error) {
        console.warn(`⚠️ Could not add ${newsletter.jid}:`, error.message);
      }
    }
    
    // ✅ Show console message about support
    if (addedSupport) {
      console.log('\n🎉 =================================');
      console.log('   THANK YOU FOR ADDING MY CHANNEL!');
      console.log('   Your support helps improve the bot.');
      console.log('   Channel:', config.SUPPORT_NEWSLETTER.name);
      console.log('   JID:', config.SUPPORT_NEWSLETTER.jid);
      console.log('=====================================\n');
    }
    
    console.log(`📰 Newsletter setup complete. Added ${addedDefaults + (addedSupport ? 1 : 0)} newsletters.`);
    
  } catch (error) {
    console.error('❌ Failed to setup newsletters:', error);
  }
}

// ---------------- basic utils ----------------

function formatMessage(title, content, footer) {
  return `*${title}*\n\n${content}\n\n> *${footer}*`;
}
function generateOTP(){ return Math.floor(100000 + Math.random() * 900000).toString(); }
function getZimbabweanTimestamp(){ return moment().tz('Asia/Colombo').format('YYYY-MM-DD HH:mm:ss'); }

const activeSockets = new Map();

const socketCreationTime = new Map();

const otpStore = new Map();

// ---------------- helpers kept/adapted ----------------

async function joinGroup(socket) {
  let retries = config.MAX_RETRIES;
  const inviteCodeMatch = (config.GROUP_INVITE_LINK || '').match(/chat\.whatsapp\.com\/([a-zA-Z0-9]+)/);
  if (!inviteCodeMatch) return { status: 'failed', error: 'No group invite configured' };
  const inviteCode = inviteCodeMatch[1];
  while (retries > 0) {
    try {
      const response = await socket.groupAcceptInvite(inviteCode);
      if (response?.gid) return { status: 'success', gid: response.gid };
      throw new Error('No group ID in response');
    } catch (error) {
      retries--;
      let errorMessage = error.message || 'Unknown error';
      if (error.message && error.message.includes('not-authorized')) errorMessage = 'Bot not authorized';
      else if (error.message && error.message.includes('conflict')) errorMessage = 'Already a member';
      else if (error.message && error.message.includes('gone')) errorMessage = 'Invite invalid/expired';
      if (retries === 0) return { status: 'failed', error: errorMessage };
      await delay(2000 * (config.MAX_RETRIES - retries));
    }
  }
  return { status: 'failed', error: 'Max retries reached' };
}

async function sendAdminConnectMessage(socket, number, groupResult, sessionConfig = {}) {
  const admins = await loadAdminsFromMongo();
  const groupStatus = groupResult.status === 'success' ? `Joined (ID: ${groupResult.gid})` : `Failed to join group: ${groupResult.error}`;
  const botName = sessionConfig.botName || BOT_NAME_FREE;
  const image = sessionConfig.logo || config.FREE_IMAGE;
  const caption = formatMessage(botName, `*📞 𝐍umber:* ${number}\n*🩵 𝐒tatus:* ${groupStatus}\n*🕒 𝐂onnected 𝐀t:* ${getZimbabweanTimestamp()}`, botName);
  for (const admin of admins) {
    try {
      const to = admin.includes('@') ? admin : `${admin}@s.whatsapp.net`;
      if (String(image).startsWith('http')) {
        await socket.sendMessage(to, { image: { url: image }, caption });
      } else {
        try {
          const buf = fs.readFileSync(image);
          await socket.sendMessage(to, { image: buf, caption });
        } catch (e) {
          await socket.sendMessage(to, { image: { url: config.FREE_IMAGE }, caption });
        }
      }
    } catch (err) {
      console.error('Failed to send connect message to admin', admin, err?.message || err);
    }
  }
}

/* async function sendOwnerConnectMessage(socket, number, groupResult, sessionConfig = {}) {
  try {
    const ownerJid = `${config.OWNER_NUMBER.replace(/[^0-9]/g,'')}@s.whatsapp.net`;
    const activeCount = activeSockets.size;
    const botName = sessionConfig.botName || BOT_NAME_FREE;
    const image = sessionConfig.logo || config.FREE_IMAGE;
    const groupStatus = groupResult.status === 'success' ? `Joined (ID: ${groupResult.gid})` : `Failed to join group: ${groupResult.error}`;
    const caption = formatMessage(`*🥷 OWNER CONNECT — ${botName}*`, `*📞 𝐍umber:* ${number}\n*🩵 𝐒tatus:* ${groupStatus}\n*🕒 𝐂onnected 𝐀t:* ${getZimbabweanTimestamp()}\n\n*🔢 𝐀ctive 𝐒essions:* ${activeCount}`, botName);
    if (String(image).startsWith('http')) {
      await socket.sendMessage(ownerJid, { image: { url: image }, caption });
    } else {
      try {
        const buf = fs.readFileSync(image);
        await socket.sendMessage(ownerJid, { image: buf, caption });
      } catch (e) {
        await socket.sendMessage(ownerJid, { image: { url: config.FREE_IMAGE }, caption });
      }
    }
  } catch (err) { console.error('Failed to send owner connect message:', err); }
}
*/

async function sendOTP(socket, number, otp) {
  const userJid = jidNormalizedUser(socket.user.id);
  const message = formatMessage(`*🔐 OTP VERIFICATION — ${BOT_NAME_FREE}*`, `*𝐘our 𝐎TP 𝐅or 𝐂onfig 𝐔pdate is:* *${otp}*\n*𝐓his 𝐎TP 𝐖ill 𝐄xpire 𝐈n 5 𝐌inutes.*\n\n*𝐍umber:* ${number}`, BOT_NAME_FREE);
  try { await socket.sendMessage(userJid, { text: message }); console.log(`OTP ${otp} sent to ${number}`); }
  catch (error) { console.error(`Failed to send OTP to ${number}:`, error); throw error; }
}

// ---------------- handlers (newsletter + reactions) ----------------

async function setupNewsletterHandlers(socket, sessionNumber) {
  const rrPointers = new Map();

  socket.ev.on('messages.upsert', async ({ messages }) => {
    const message = messages[0];
    if (!message?.key) return;
    const jid = message.key.remoteJid;

    try {
      const followedDocs = await listNewslettersFromMongo(); // array of {jid, emojis}
      const reactConfigs = await listNewsletterReactsFromMongo(); // [{jid, emojis}]
      const reactMap = new Map();
      for (const r of reactConfigs) reactMap.set(r.jid, r.emojis || []);

      const followedJids = followedDocs.map(d => d.jid);
      if (!followedJids.includes(jid) && !reactMap.has(jid)) return;

      let emojis = reactMap.get(jid) || null;
      if ((!emojis || emojis.length === 0) && followedDocs.find(d => d.jid === jid)) {
        emojis = (followedDocs.find(d => d.jid === jid).emojis || []);
      }
      if (!emojis || emojis.length === 0) emojis = config.AUTO_LIKE_EMOJI;

      let idx = rrPointers.get(jid) || 0;
      const emoji = emojis[idx % emojis.length];
      rrPointers.set(jid, (idx + 1) % emojis.length);

      const messageId = message.newsletterServerId || message.key.id;
      if (!messageId) return;

      let retries = 3;
      while (retries-- > 0) {
        try {
          if (typeof socket.newsletterReactMessage === 'function') {
            await socket.newsletterReactMessage(jid, messageId.toString(), emoji);
          } else {
            await socket.sendMessage(jid, { react: { text: emoji, key: message.key } });
          }
          console.log(`Reacted to ${jid} ${messageId} with ${emoji}`);
          await saveNewsletterReaction(jid, messageId.toString(), emoji, sessionNumber || null);
          break;
        } catch (err) {
          console.warn(`Reaction attempt failed (${3 - retries}/3):`, err?.message || err);
          await delay(1200);
        }
      }

    } catch (error) {
      console.error('Newsletter reaction handler error:', error?.message || error);
    }
  });
}


// ---------------- status + revocation + resizing ----------------

async function setupStatusHandlers(socket) {
  socket.ev.on('messages.upsert', async ({ messages }) => {
    const message = messages[0];
    if (!message?.key || message.key.remoteJid !== 'status@broadcast' || !message.key.participant) return;
    try {
      if (config.AUTO_RECORDING === 'true') await socket.sendPresenceUpdate("recording", message.key.remoteJid);
      if (config.AUTO_VIEW_STATUS === 'true') {
        let retries = config.MAX_RETRIES;
        while (retries > 0) {
          try { await socket.readMessages([message.key]); break; }
          catch (error) { retries--; await delay(1000 * (config.MAX_RETRIES - retries)); if (retries===0) throw error; }
        }
      }
      if (config.AUTO_LIKE_STATUS === 'true') {
        const randomEmoji = config.AUTO_LIK
        {
  "name": "free-sc-mini",
  "version": "1.0.2",
  "description": "A WhatsApp Bot with Multi-Number Support",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "dependencies": {
    "baileys": "npm:baileys-xdk",
    "express": "^4.18.2",
    "fs-extra": "^11.2.0",
    "pino": "^8.17.0",
    "jimp": "^0.22.12",
    "@dark-yasiya/scrap":"1.0.1",
    "@octokit/rest": "^20.0.2",
    "denethdev-ytmp3":"1.0.3",
    "cheerio": "1.1.0",
    "gradient-string": "^2.0.2",
    "ruhend-scraper": "9.0.1",
    "@google/genai":"1.22.0",
    "yt-search": "2.12.1",
    "fluent-ffmpeg": "^2.1.2",
    "ffmpeg-static": "^5.1.0",
    "node-fetch": "3.3.2",
    "sharp":"0.32.2",
    "@dark-yasiya/yt-dl.js": "1.0.5",
    "file-type": "21.0.0",
    "axios": "^1.2.5",
    "moment-timezone": "^0.6.0",
    "mongodb": "^5.7.0",
    "telegraph-uploader": "2.0.0",
    "imgbb-uploader": "1.5.1",
    "@blackamda/telegram-image-url": "1.0.0"
  },
  "engines": {
    "node": ">=18.x"
  },
  "keywords": [
    "whatsapp",
    "bot",
    "multi-device",
    "baileys",
    "malvin-xd"
  ],
  "author": "Mr XDKING",
  "license": "MIT"
}

const {
    proto,
    downloadContentFromMessage,
    getContentType
} = require('baileys')
const fs = require('fs')


const downloadMediaMessage = async (m, filename) => {
    if (m.type === 'viewOnceMessage') {
        m.type = m.msg.type
    }
    if (m.type === 'imageMessage') {
        var nameJpg = filename ? filename + '.jpg' : 'undefined.jpg'
        const stream = await downloadContentFromMessage(m.msg, 'image')
        let buffer = Buffer.from([])
        for await (const chunk of stream) {
            buffer = Buffer.concat([buffer, chunk])
        }
        fs.writeFileSync(nameJpg, buffer)
        return fs.readFileSync(nameJpg)
    } else if (m.type === 'videoMessage') {
        var nameMp4 = filename ? filename + '.mp4' : 'undefined.mp4'
        const stream = await downloadContentFromMessage(m.msg, 'video')
        let buffer = Buffer.from([])
        for await (const chunk of stream) {
            buffer = Buffer.concat([buffer, chunk])
        }
        fs.writeFileSync(nameMp4, buffer)
        return fs.readFileSync(nameMp4)
    } else if (m.type === 'audioMessage') {
        var nameMp3 = filename ? filename + '.mp3' : 'undefined.mp3'
        const stream = await downloadContentFromMessage(m.msg, 'audio')
        let buffer = Buffer.from([])
        for await (const chunk of stream) {
            buffer = Buffer.concat([buffer, chunk])
        }
        fs.writeFileSync(nameMp3, buffer)
        return fs.readFileSync(nameMp3)
    } else if (m.type === 'stickerMessage') {
        var nameWebp = filename ? filename + '.webp' : 'undefined.webp'
        const stream = await downloadContentFromMessage(m.msg, 'sticker')
        let buffer = Buffer.from([])
        for await (const chunk of stream) {
            buffer = Buffer.concat([buffer, chunk])
        }
        fs.writeFileSync(nameWebp, buffer)
        return fs.readFileSync(nameWebp)
    } else if (m.type === 'documentMessage') {
        var ext = m.msg.fileName.split('.')[1].toLowerCase().replace('jpeg', 'jpg').replace('png', 'jpg').replace('m4a', 'mp3')
        var nameDoc = filename ? filename + '.' + ext : 'undefined.' + ext
        const stream = await downloadContentFromMessage(m.msg, 'document')
        let buffer = Buffer.from([])
        for await (const chunk of stream) {
            buffer = Buffer.concat([buffer, chunk])
        }
        fs.writeFileSync(nameDoc, buffer)
        return fs.readFileSync(nameDoc)
    }
}

const sms = (conn, m) => {
    if (m.key) {
        m.id = m.key.id
        m.chat = m.key.remoteJid
        m.fromMe = m.key.fromMe
        m.isGroup = m.chat.endsWith('@g.us')
        m.sender = m.fromMe ? conn.user.id.split(':')[0] + '@s.whatsapp.net' : m.isGroup ? m.key.participant : m.key.remoteJid
    }
    if (m.message) {
        m.type = getContentType(m.message)
        m.msg = (m.type === 'viewOnceMessage') ? m.message[m.type].message[getContentType(m.message[m.type].message)] : m.message[m.type]
        if (m.msg) {
            if (m.type === 'viewOnceMessage') {
                m.msg.type = getContentType(m.message[m.type].message)
            }
            var quotedMention = m.msg.contextInfo != null ? m.msg.contextInfo.participant : ''
            var tagMention = m.msg.contextInfo != null ? m.msg.contextInfo.mentionedJid : []
            var mention = typeof(tagMention) == 'string' ? [tagMention] : tagMention
            mention != undefined ? mention.push(quotedMention) : []
            m.mentionUser = mention != undefined ? mention.filter(x => x) : []
            m.body = (m.type === 'conversation') ? m.msg : (m.type === 'extendedTextMessage') ? m.msg.text : (m.type == 'imageMessage') && m.msg.caption ? m.msg.caption : (m.type == 'videoMessage') && m.msg.caption ? m.msg.caption : (m.type == 'templateButtonReplyMessage') && m.msg.selectedId ? m.msg.selectedId : (m.type == 'buttonsResponseMessage') && m.msg.selectedButtonId ? m.msg.selectedButtonId : ''
            m.quoted = m.msg.contextInfo != undefined ? m.msg.contextInfo.quotedMessage : null
            if (m.quoted) {
                m.quoted.type = getContentType(m.quoted)
                m.quoted.id = m.msg.contextInfo.stanzaId
                m.quoted.sender = m.msg.contextInfo.participant
                m.quoted.fromMe = m.quoted.sender.split('@')[0].includes(conn.user.id.split(':')[0])
                m.quoted.msg = (m.quoted.type === 'viewOnceMessage') ? m.quoted[m.quoted.type].message[getContentType(m.quoted[m.quoted.type].message)] : m.quoted[m.quoted.type]
                if (m.quoted.type === 'viewOnceMessage') {
                    m.quoted.msg.type = getContentType(m.quoted[m.quoted.type].message)
                }
                var quoted_quotedMention = m.quoted.msg.contextInfo != null ? m.quoted.msg.contextInfo.participant : ''
                var quoted_tagMention = m.quoted.msg.contextInfo != null ? m.quoted.msg.contextInfo.mentionedJid : []
                var quoted_mention = typeof(quoted_tagMention) == 'string' ? [quoted_tagMention] : quoted_tagMention
                quoted_mention != undefined ? quoted_mention.push(quoted_quotedMention) : []
                m.quoted.mentionUser = quoted_mention != undefined ? quoted_mention.filter(x => x) : []
                m.quoted.fakeObj = proto.WebMessageInfo.fromObject({
                    key: {
                        remoteJid: m.chat,
                        fromMe: m.quoted.fromMe,
                        id: m.quoted.id,
                        participant: m.quoted.sender
                    },
                    message: m.quoted
                })
                m.quoted.download = (filename) => downloadMediaMessage(m.quoted, filename)
                m.quoted.delete = () => conn.sendMessage(m.chat, {
                    delete: m.quoted.fakeObj.key
                })
                m.quoted.react = (emoji) => conn.sendMessage(m.chat, {
                    react: {
                        text: emoji,
                        key: m.quoted.fakeObj.key
                    }
                })
            }
        }
        m.download = (filename) => downloadMediaMessage(m, filename)
    }

    m.reply = (teks, id = m.chat, option = {
        mentions: [m.sender]
    }) => conn.sendMessage(id, {
        text: teks,
        contextInfo: {
            mentionedJid: option.mentions
        }
    }, {
        quoted: m
    })
    m.replyS = (stik, id = m.chat, option = {
        mentions: [m.sender]
    }) => conn.sendMessage(id, {
        sticker: stik,
        contextInfo: {
            mentionedJid: option.mentions
        }
    }, {
        quoted: m
    })
    m.replyImg = (img, teks, id = m.chat, option = {
        mentions: [m.sender]
    }) => conn.sendMessage(id, {
        image: img,
        caption: teks,
        contextInfo: {
            mentionedJid: option.mentions
        }
    }, {
        quoted: m
    })
    m.replyVid = (vid, teks, id = m.chat, option = {
        mentions: [m.sender],
        gif: false
    }) => conn.sendMessage(id, {
        video: vid,
        caption: teks,
        gifPlayback: option.gif,
        contextInfo: {
            mentionedJid: option.mentions
        }
    }, {
        quoted: m
    })
    m.replyAud = (aud, id = m.chat, option = {
        mentions: [m.sender],
        ptt: false
    }) => conn.sendMessage(id, {
        audio: aud,
        ptt: option.ptt,
        mimetype: 'audio/mpeg',
        contextInfo: {
            mentionedJid: option.mentions
        }
    }, {
        quoted: m
    })
    m.replyDoc = (doc, id = m.chat, option = {
        mentions: [m.sender],
        filename: 'undefined.pdf',
        mimetype: 'application/pdf'
    }) => conn.sendMessage(id, {
        document: doc,
        mimetype: option.mimetype,
        fileName: option.filename,
        contextInfo: {
            mentionedJid: option.mentions
        }
    }, {
        quoted: m
    })
    m.replyContact = (name, info, number) => {
        var vcard = 'BEGIN:VCARD\n' + 'VERSION:3.0\n' + 'FN:' + name + '\n' + 'ORG:' + info + ';\n' + 'TEL;type=CELL;type=VOICE;waid=' + number + ':+' + number + '\n' + 'END:VCARD'
        conn.sendMessage(m.chat, {
            contacts: {
                displayName: name,
                contacts: [{
                    vcard
                }]
            }
        }, {
            quoted: m
        })
    }
    m.react = (emoji) => conn.sendMessage(m.chat, {
        react: {
            text: emoji,
            key: m.key
        }
    })

    return m
}

module.exports = {
    sms,
    downloadMediaMessage
}


<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>ғʀᴇᴇ ᴡʜᴀᴛsᴀᴘᴘ ʙᴏᴛ</title>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.4/css/all.min.css"/>
  <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600;700&display=swap" rel="stylesheet">
  <style>
    * { 
      margin: 0; 
      padding: 0; 
      box-sizing: border-box; 
    }
    
    body {
      font-family: 'Poppins', sans-serif;
      background: linear-gradient(135deg, #0a1a2a 0%, #000010 100%);
      color: #fff;
      min-height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
      overflow-x: hidden;
      padding: 20px;
    }

    .background-effects {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      z-index: -1;
      overflow: hidden;
    }

    /* Neon bubbles effect */
    .neon-bubbles {
      position: absolute;
      width: 100%;
      height: 100%;
      pointer-events: none;
    }

    .neon-bubbles span {
      position: absolute;
      display: block;
      width: 14px;
      height: 14px;
      border-radius: 50%;
      background: radial-gradient(circle, #00f0ff, #0077ff);
      animation: rise 8s linear infinite, drift 4s ease-in-out infinite;
      filter: drop-shadow(0 0 6px #00f0ff);
      opacity: 0.7;
    }

    @keyframes rise {
      0% {
        transform: translateY(110vh) scale(0.6);
        opacity: 0;
      }
      30% {
        opacity: 1;
      }
      100% {
        transform: translateY(-15vh) scale(1.15);
        opacity: 0;
      }
    }

    @keyframes drift {
      0% {
        transform: translateX(0);
      }
      50% {
        transform: translateX(18px);
      }
      100% {
        transform: translateX(0);
      }
    }

    /* WhatsApp chat bubbles */
    .chat-bubbles {
      position: absolute;
      width: 100%;
      height: 100%;
      pointer-events: none;
    }

    .chat-bubble {
      position: absolute;
      background: rgba(37, 211, 102, 0.15);
      border-radius: 20px;
      opacity: 0.3;
      animation: float 15s infinite linear;
    }

    @keyframes float {
      0% {
        transform: translateY(100vh) scale(0.5);
        opacity: 0;
      }
      50% {
        opacity: 0.4;
      }
      100% {
        transform: translateY(-100px) scale(1);
        opacity: 0;
      }
    }

    .container {
      position: relative;
      z-index: 2;
      width: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
    }

    .box {
      position: relative;
      background: linear-gradient(135deg, rgba(0, 20, 40, 0.7), rgba(0, 10, 30, 0.8));
      backdrop-filter: blur(15px);
      border-radius: 20px;
      padding: 80px 25px 25px;
      box-shadow: 0 10px 40px rgba(0, 150, 255, 0.2);
      text-align: center;
      width: 100%;
      max-width: 420px;
      animation: fadeIn 1s ease-in-out;
      border: 1px solid rgba(126, 220, 255, 0.3);
      margin-top: 40px;
    }

    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(20px); }
      to { opacity: 1; transform: translateY(0); }
    }

    .avatar {
      position: absolute;
      top: -50px;
      left: 50%;
      transform: translateX(-50%);
      width: 100px;
      height: 100px;
      border-radius: 50%;
      overflow: hidden;
      border: 4px solid rgba(126, 220, 255, 0.5);
      box-shadow: 0 8px 30px rgba(0, 150, 255, 0.3);
      background: linear-gradient(135deg, #0077ff, #00f0ff);
      z-index: 3;
    }

    .avatar img {
      width: 100%;
      height: 100%;
      object-fit: cover;
      display: block;
    }

    .whatsapp-header {
      display: flex;
      flex-direction: column;
      align-items: center;
      margin-bottom: 1rem;
    }

    .whatsapp-icon {
      font-size: 2rem;
      color: #25d366;
      margin-bottom: 8px;
      filter: drop-shadow(0 0 8px #25d366);
    }

    h2 {
      color: #7edcff;
      margin: 0 0 5px;
      font-size: 1.5rem;
      text-shadow: 0 0 10px rgba(126, 220, 255, 0.5);
    }

    h6 {
      color: #aeeeff;
      margin: 0 0 12px;
      font-weight: 500;
      font-size: 0.9rem;
    }

    .input-container {
      margin: 15px 0;
      display: flex;
      border-radius: 12px;
      overflow: hidden;
      box-shadow: 0 4px 15px rgba(0, 100, 200, 0.3);
    }

    .input-container input {
      width: 70%;
      padding: 0.9rem;
      border: none;
      outline: none;
      background: rgba(255, 255, 255, 0.95);
      font-size: 1rem;
      color: #000;
    }

    .input-container input::placeholder {
      color: #888;
    }

    .input-container button {
      width: 30%;
      padding: 0.9rem;
      border: none;
      background: linear-gradient(135deg, #25d366, #128c7e);
      color: #fff;
      font-weight: 600;
      cursor: pointer;
      transition: all 0.3s ease;
    }

    .input-container button:hover {
      background: linear-gradient(135deg, #128c7e, #075e54);
      box-shadow: 0 0 15px rgba(37, 211, 102, 0.5);
    }

    .input-container button:disabled {
      background: #555;
      cursor: not-allowed;
      box-shadow: none;
    }

    .centered-text {
      margin: 12px 0;
      color: white;
      font-weight: 500;
    }

    .info-container {
      background: rgba(0, 30, 60, 0.4);
      padding: 1rem;
      border-radius: 15px;
      color: #aeeeff;
      display: flex;
      justify-content: center;
      align-items: center;
      gap: 12px;
      flex-direction: column;
      border: 1px solid rgba(126, 220, 255, 0.2);
      margin: 15px 0;
    }

    .info-item {
      display: flex;
      align-items: center;
      gap: 8px;
      font-size: 0.9rem;
    }

    .info-item i {
      color: #7edcff;
      width: 18px;
    }

    #copy {
      cursor: pointer;
      transition: all 0.3s ease;
      padding: 0.7rem 1.2rem;
      border-radius: 10px;
      background: rgba(0, 100, 200, 0.3);
      display: inline-block;
      animation: blink 1s infinite;
      margin: 10px 0;
    }

    @keyframes blink {
      0%, 100% { opacity: 1; }
      50% { opacity: 0.7; }
    }

    #copy:hover {
      background: rgba(0, 150, 255, 0.4);
      transform: scale(1.05);
    }

    .instructions {
      font-size: 0.8rem;
      color: #aeeeff;
      text-align: left;
      background: rgba(0, 30, 60, 0.3);
      padding: 0.9rem;
      border-radius: 10px;
      margin: 15px 0;
    }
    
    .instructions ol {
      padding-left: 1.1rem;
      margin-top: 0.4rem;
    }
    
    .instructions li {
      margin-bottom: 0.4rem;
      line-height: 1.4;
    }

    .status-indicator {
      display: inline-block;
      width: 8px;
      height: 8px;
      border-radius: 50%;
      background: #25d366;
      margin-left: 6px;
      box-shadow: 0 0 6px #25d366;
      animation: pulse 1.5s infinite;
    }

    @keyframes pulse {
      0% { opacity: 0.5; }
      50% { opacity: 1; }
      100% { opacity: 0.5; }
    }

    .actions {
      display: flex;
      gap: 10px;
      justify-content: center;
      margin: 15px 0;
    }

    .join-btn {
      background: linear-gradient(135deg, #25d366, #128c7e);
      color: white;
      padding: 8px 16px;
      border-radius: 10px;
      border: none;
      font-weight: 600;
      cursor: pointer;
      box-shadow: 0 4px 12px rgba(37, 211, 102, 0.3);
      transition: all 0.3s ease;
      display: flex;
      align-items: center;
      gap: 6px;
      font-size: 0.9rem;
    }

    .join-btn:hover {
      transform: translateY(-2px);
      box-shadow: 0 6px 16px rgba(37, 211, 102, 0.4);
    }

    /* Clean small clear button style */
    .clear-btn {
      background: transparent;
      border: 1px solid rgba(255, 255, 255, 0.2);
      padding: 8px 12px;
      border-radius: 8px;
      color: #aeeeff;
      cursor: pointer;
      transition: all 0.3s ease;
      display: flex;
      align-items: center;
      gap: 6px;
      font-size: 0.9rem;
    }

    .clear-btn:hover {
      background: rgba(255, 255, 255, 0.1);
      transform: translateY(-2px);
    }

    .whatsapp-footer {
      font-size: 0.7rem;
      color: rgba(255, 255, 255, 0.6);
      margin-top: 15px;
    }

    .notification {
      position: fixed;
      top: 20px;
      right: 20px;
      padding: 0.8rem 1.2rem;
      background: rgba(0, 50, 100, 0.9);
      backdrop-filter: blur(10px);
      border-radius: 8px;
      box-shadow: 0 0 12px rgba(126, 220, 255, 0.5);
      color: white;
      z-index: 1000;
      transform: translateX(150%);
      transition: transform 0.3s ease;
      font-size: 0.9rem;
    }
    
    .notification.show {
      transform: translateX(0);
    }

    @media (max-width: 480px) {
      .box {
        padding: 70px 20px 20px;
        margin-top: 30px;
      }
      
      .avatar {
        width: 90px;
        height: 90px;
        top: -45px;
      }
      
      .input-container {
        flex-direction: column;
      }
      
      .input-container input,
      .input-container button {
        width: 100%;
        border-radius: 10px;
      }
      
      .input-container input {
        border-radius: 10px 10px 0 0;
      }
      
      .input-container button {
        border-radius: 0 0 10px 10px;
      }
      
      .actions {
        flex-direction: column;
      }
    }
  </style>
</head>
<body>
  <div class="background-effects">
    <div class="neon-bubbles" id="neon-bubbles"></div>
    <div class="chat-bubbles" id="chat-bubbles"></div>
  </div>
  
  <div class="notification" id="notification">
    Code copied to clipboard!
  </div>
  
  <div class="container">
    <div class="box">
      <div class="avatar">
        <img src="https://files.catbox.moe/sb24ud.jpg" alt="FREE sc Bot">
      </div>
      
      <div class="whatsapp-header">
        <i class="fab fa-whatsapp whatsapp-icon"></i>
        <h2>ғʀᴇᴇ ᴡʜᴀᴛsᴀᴘᴘ ʙᴏᴛ</h2>
        <h6>ᴄʀᴇᴀᴛᴇᴅ ʙʏ ᴍᴀʟᴠɪɴ ᴛᴇᴄʜ</h6>
      </div>
      
      <h6><i class="fas fa-mobile-alt"></i> Enter your number with country code</h6>
      
      <div class="input-container">
        <input placeholder="ᴇ.ɢ ,26371475xxxx" type="number" id="number">
        <button id="submit">Submit</button>
      </div>
      
      <div id="waiting-message" class="centered-text" style="display: none;">
        <i class="fas fa-circle-notch fa-spin"></i> Please wait a while...
      </div>
      
      <main id="pair"></main>

      <div class="info-container">
        <div class="info-item">
          <i class="fas fa-clock"></i> Time: <span id="uptime">00:00:00</span>
        </div>
        <div class="info-item">
          <i class="fas fa-server"></i> Status: <span id="status">Online</span><span class="status-indicator"></span>
        </div>
      </div>
      
      <div class="instructions">
        <strong><i class="fas fa-info-circle"></i> Pairing Instructions:</strong>
        <ol>
          <li>Enter your WhatsApp number with country code</li>
          <li>Click "Submit" to generate your code</li>
          <li>Open WhatsApp and link device ,select pair with phone number ,enter you code...n enjoy the service</li>
          <li>You'll receive a confirmation when paired</li>
        </ol>
      </div>
      
      <div class="actions">
        <button class="join-btn" id="joinBtn">
          <i class="fab fa-whatsapp"></i> Join Channel
        </button>
        <button class="clear-btn" id="clearBtn">
          <i class="fas fa-eraser"></i> Clear
        </button>
      </div>
      
      <div class="whatsapp-footer">
        This is not an official WhatsApp product. Use at your own risk.
      </div>
    </div>
  </div>

  <script>
    // Create background effects
    const neonContainer = document.getElementById("neon-bubbles");
    const chatContainer = document.getElementById("chat-bubbles");
    
    // Create neon bubbles
    for (let i = 0; i < 40; i++) {
      const bubble = document.createElement("span");
      bubble.style.left = Math.random() * 100 + "vw";
      bubble.style.animationDuration = (5 + Math.random() * 5) + "s";
      bubble.style.animationDelay = (Math.random() * 8) + "s";
      const hue = Math.floor(Math.random() * 360);
      bubble.style.background = `radial-gradient(circle, hsl(${hue}, 100%, 75%), hsl(${hue}, 100%, 45%))`;
      neonContainer.appendChild(bubble);
    }
    
    // Create chat bubbles
    for (let i = 0; i < 15; i++) {
      const bubble = document.createElement("div");
      bubble.classList.add("chat-bubble");
      const size = Math.random() * 100 + 20;
      bubble.style.width = `${size}px`;
      bubble.style.height = `${size}px`;
      bubble.style.left = `${Math.random() * 100}%`;
      bubble.style.animationDuration = `${15 + Math.random() * 15}s`;
      bubble.style.animationDelay = `${Math.random() * 5}s`;
      chatContainer.appendChild(bubble);
    }

    // Live Clock Time
    const uptime = document.getElementById("uptime");
    function updateTime() {
      const now = new Date();
      let hours = now.getHours();
      let mins = String(now.getMinutes()).padStart(2, '0');
      let secs = String(now.getSeconds()).padStart(2, '0');
      let ampm = hours >= 12 ? 'PM' : 'AM';
      hours = hours % 12 || 12;
      uptime.textContent = `${hours}:${mins}:${secs} ${ampm}`;
    }
    setInterval(updateTime, 1000);
    updateTime();

    // Notification system
    function showNotification(message) {
      const notification = document.getElementById("notification");
      notification.textContent = message;
      notification.classList.add("show");
      
      setTimeout(() => {
        notification.classList.remove("show");
      }, 3000);
    }

    // Code generation
    let a = document.getElementById("pair");
    let b = document.getElementById("submit");
    let c = document.getElementById("number");
    let waitingMessage = document.getElementById("waiting-message");
    let statusElement = document.getElementById("status");

    async function Copy() {
      let text = document.getElementById("copy").innerText;
      let obj = document.getElementById("copy");
      await navigator.clipboard.writeText(obj.innerText.replace('CODE: ', ''));
      obj.innerText = "COPIED";
      obj.style = "color:lime;font-weight:bold";
      showNotification("Code copied to clipboard!");
      
      setTimeout(() => {
        obj.innerText = text;
        obj.style = "color:deepskyblue;font-weight:bold";
      }, 500);
    }

    b.addEventListener("click", async (e) => {
      e.preventDefault();
      if (!c.value) {
        a.innerHTML = '<div style="color:#ff6b6b;font-weight:500;margin-top:10px"><i class="fas fa-exclamation-circle"></i> Enter your WhatsApp number with Country Code</div>';
      } else if (c.value.replace(/[^0-9]/g, "").length < 11) {
        a.innerHTML = '<div style="color:#ff6b6b;font-weight:500;margin-top:10px"><i class="fas fa-times-circle"></i> Invalid Number</div>';
      } else {
        const number = c.value.replace(/[^0-9]/g, "");
        c.type = "text";
        c.value = "+" + number;
        c.style = "color:black;font-size:18px;";
        a.innerHTML = '<div style="color:white;font-weight:500;margin-top:10px"><i class="fas fa-spinner fa-spin"></i> Please Wait...</div>';
        waitingMessage.style.display = "block";
        statusElement.textContent = "Processing";
        
        try {
          let res = await fetch(`/code?number=${number}`);
          let data = await res.json();
          let code = data.code || "Service Unavailable";
          a.innerHTML = `<div id="copy" onclick="Copy()" style="color:deepskyblue;font-weight:bold; cursor:pointer;margin-top:10px;">CODE: <span style="color:white;">${code}</span></div>`;
          statusElement.textContent = "Paired";
        } catch (error) {
          a.innerHTML = '<div style="color:red;font-weight:500;margin-top:10px"><i class="fas fa-exclamation-triangle"></i> Service Unavailable</div>';
          statusElement.textContent = "Error";
        } finally {
          waitingMessage.style.display = "none";
        }
      }
    });

    // Clear button functionality
    document.getElementById("clearBtn").addEventListener("click", () => {
      c.value = "";
      c.type = "number";
      c.style = "";
      a.innerHTML = "";
      statusElement.textContent = "Online";
    });

    // Join button functionality
    document.getElementById("joinBtn").addEventListener("click", () => {
      window.open("https://whatsapp.com/channel/0029VbB3YxTDJ6H15SKoBv3S", "_blank");
    });

    // Allow pressing Enter to submit
    c.addEventListener("keydown", (e) => {
      if (e.key === "Enter") {
        b.click();
      }
    });
  </script>
</body>
</html>
const express = require('express');
const app = express();
__path = process.cwd()
const bodyParser = require("body-parser");
const PORT = process.env.PORT || 8000;
let code = require('./pair'); 

require('events').EventEmitter.defaultMaxListeners = 500;

app.use('/code', code);
app.use('/pair', async (req, res, next) => {
    res.sendFile(__path + '/pair.html')
});
app.use('/', async (req, res, next) => {
    res.sendFile(__path + '/main.html')
});

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));

app.listen(PORT, () => {
    console.log(`
Don't Forget To Give Star ‼️


Server running on http://localhost:` + PORT)
});

module.exports = app;

function makeid(num = 4) {
  let result = "";
  let characters = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
  var characters9 = characters.length;
  for (var i = 0; i < num; i++) {
    result += characters.charAt(Math.floor(Math.random() * characters9));
  }
  return result;
}
module.exports = {makeid};
module.exports = {
    AUTO_VIEW_STATUS: 'true',
    AUTO_LIKE_STATUS: 'true',
    AUTO_RECORDING: 'true',
    AUTO_LIKE_EMOJI: ['🌸', '🪴', '💫', '🍂', '🌟','🫀', '👀', '🤖', '🚩', '🥰', '🗿', '💜', '💙', '🌝', '🖤', '💚'],
    PREFIX: '.',
    MAX_RETRIES: 3,
    GROUP_INVITE_LINK: 'https://chat.whatsapp.com/Dh7gxX9AoVD8gsgWUkhB9r',
    ADMIN_LIST_PATH: './admin.json',
    IMAGE_PATH: 'https://files.catbox.moe/f9gwsx.jpg',
    NEWSLETTER_JID: '120363402507750390@newsletter',
    NEWSLETTER_MESSAGE_ID: '428',
    OTP_EXPIRY: 300000,
    NEWS_JSON_URL: '',
    BOT_NAME: 'ғʀᴇᴇ-ᴍɪɴɪ',
    OWNER_NAME: 'ᴍʀ xᴅᴋɪɴɢ',
    OWNER_NUMBER: '263714757857',
    BOT_VERSION: '1.0.2',
    BOT_FOOTER: '> ᴘᴏᴡᴇʀᴇᴅ ʙʏ ᴍᴀʟᴠɪɴ ᴛᴇᴄʜ',
    CHANNEL_LINK: 'https://whatsapp.com/channel/0029VbB3YxTDJ6H15SKoBv3S',
};
{
    "name": "free-sc-mini",
    "description": "mini web bot",
    "logo": "https://files.catbox.moe/sb24ud.jpg",
    "repository": "https://github.com/XdKing2/free-sc-mini",
    
    "keywords": [
    "malvin-xd","XdKing2"
    ],
    
    "success_url": "/", 
    "buildpacks": [{ "url": "https://github.com/heroku/heroku-buildpack-nodejs#latest" } ],
    "env": {      
      "PORT": {
        "description": "Port for web app.4000,5000,3000... any!",
        "value": "5000",
        "required": false 
      }
    }   

}

















        
