import fetch from "node-fetch";
import fs from "fs";
import { v4 as uuid } from "uuid";

// ====== Token и Admin ID ======
const TOKEN = "8555630882:AAG4stfUf6hpDxwKQL7jteQYmomC4Xpql44";
const ADMIN = "1999635628";
const API = `https://api.telegram.org/bot${TOKEN}`;
const DB_FILE = "./data/users.json";
const VPN_LINK = "https://hirbilon.net/open?sub_url=https://g3.hirbilon.net:443/yessub/p5ln8k1qld9nf0sa";

function loadDB() {
  try { return JSON.parse(fs.readFileSync(DB_FILE)); }
  catch { return {}; }
}

function saveDB(db) {
  fs.writeFileSync(DB_FILE, JSON.stringify(db, null, 2));
}

async function tg(method, body) {
  await fetch(`${API}/${method}`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(body)
  });
}

// Inline keyboard menu
const menuInline = {
  inline_keyboard: [
    [{ text: "🚀 Получить VPN", callback_data: "get_vpn" }],
    [
      { text: "👤 Профиль", callback_data: "profile" },
      { text: "💎 Тарифы", callback_data: "plans" }
    ],
    [{ text: "👥 Рефералы", callback_data: "ref" }],
    [{ text: "🆘 Поддержка", callback_data: "support" }]
  ]
};

export default async function handler(req, res) {
  if (req.method !== "POST") return res.status(200).send("ok");
  const msg = req.body.message || req.body.callback_query;
  if (!msg) return res.status(200).send("no message");

  const chatId = msg.chat ? msg.chat.id.toString() : msg.from.id.toString();
  const text = msg.data || msg.text || "";

  let db = loadDB();
  if (!db[chatId]) {
    db[chatId] = {
      id: chatId,
      plan: "trial",
      expires: Date.now() + 86400000, // 1 день trial
      ref: uuid()
    };
    saveDB(db);
  }
  const user = db[chatId];

  // START
  if (text === "/start") {
    return tg("sendMessage", {
      chat_id: chatId,
      text: `🔥 Добро пожаловать в VPN бот!\n\n⚡ Скорость: 1Gbps\n🌍 Европа / США\n🔐 Безопасное соединение`,
      reply_markup: menuInline
    });
  }

  // CALLBACK / кнопки
  if (text === "get_vpn" || text === "🚀 Получить VPN") {
    if (Date.now() > user.expires) {
      return tg("sendMessage", { chat_id: chatId, text: "❌ Подписка истекла." });
    }
    return tg("sendMessage", { chat_id: chatId, text: `✅ Ваш VPN готов:\n\n${VPN_LINK}` });
  }

  if (text === "profile" || text === "👤 Профиль") {
    return tg("sendMessage", {
      chat_id: chatId,
      text: `👤 ID: ${chatId}\n💎 Тариф: ${user.plan}\n⏳ До: ${new Date(user.expires).toLocaleDateString()}`
    });
  }

  // 💎 Тарифы + Sberbank оплата
  if (text === "plans" || text === "💎 Тарифы") {
    return tg("sendMessage", {
      chat_id: chatId,
      text: `💎 Тарифы / Купить VPN:

1 месяц — 5$
3 месяца — 12$
1 год — 40$

Для оплаты нажмите на кнопку ниже 👇`,
      reply_markup: {
        inline_keyboard: [
          [
            { text: "💳 Оплатить через Сбербанк", url: "https://www.sberbank.ru/ru/choise_bank?requisiteNumber=79990402614&bankCode=100000000111" }
          ]
        ]
      }
    });
  }

  if (text === "ref" || text === "👥 Рефералы") {
    return tg("sendMessage", {
      chat_id: chatId,
      text: `👥 Пригласите друзей:\nhttps://t.me/YOUR_BOT?start=${user.ref}`
    });
  }

  if (text === "support" || text === "🆘 Поддержка") {
    return tg("sendMessage", { chat_id: chatId, text: "Напишите администратору @admin" });
  }

  // ADMIN BROADCAST
  if (chatId === ADMIN && text.startsWith("/send ")) {
    const msgText = text.replace("/send ", "");
    for (const id of Object.keys(db)) {
      await tg("sendMessage", { chat_id: id, text: `📢 ${msgText}` });
    }
  }

  saveDB(db);
  res.status(200).send("ok");
}
