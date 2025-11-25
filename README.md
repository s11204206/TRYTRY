# TRYTRY
// script.js

// ===== 設定 =====
// 若後端部署在其他 domain，把這裡改成你的完整 URL，例如:
// const BACKEND_URL = 'https://your-backend.vercel.app/api/chat';
const BACKEND_URL = '/api/chat';

// ===== 工具函式 =====
function escapeHtml(str) {
  if (!str && str !== 0) return '';
  return String(str)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#039;');
}

function formatTime(date = new Date()) {
  return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
}

// ===== DOM 元件 =====
const messagesEl = document.getElementById('messages');
const inputEl = document.getElementById('chat-input');
const sendBtn = document.getElementById('send-btn');

const reportForm = document.getElementById('report-form');
const reportResult = document.getElementById('report-result');

// ===== 聊天訊息顯示 =====
function createBubble(text, who = 'bot') {
  const wrapper = document.createElement('div');
  wrapper.className = 'message ' + (who === 'user' ? 'user' : 'bot');

  const content = document.createElement('div');
  content.className = 'message-content';
  content.innerHTML = escapeHtml(text);

  const meta = document.createElement('div');
  meta.className = 'message-meta';
  meta.textContent = formatTime();

  wrapper.appendChild(content);
  wrapper.appendChild(meta);

  return wrapper;
}

function appendMessage(text, who = 'bot') {
  const bubble = createBubble(text, who);
  messagesEl.appendChild(bubble);
  messagesEl.scrollTop = messagesEl.scrollHeight;
}

// ===== 輸入/送出處理 =====
let isSending = false;

function setSendingState(state) {
  isSending = state;
  sendBtn.disabled = state;
  inputEl.disabled = state;
}

function showTypingIndicator() {
  const typing = document.createElement('div');
  typing.className = 'message bot typing-indicator';
  typing.dataset.typing = '1';
  typing.innerHTML = '<div class="dots">●●●</div>';
  messagesEl.appendChild(typing);
  messagesEl.scrollTop = messagesEl.scrollHeight;
  return typing;
}

function removeTypingIndicator() {
  const lastTyping = messagesEl.querySelector('.message.typing-indicator[data-typing="1"]');
  if (lastTyping) lastTyping.remove();
}

async function sendMessage() {
  const text = inputEl.value.trim();
  if (!text || isSending) return;

  appendMessage(text, 'user');
  inputEl.value = '';
  setSendingState(true);

  // 顯示打字中
  const typingNode = showTypingIndicator();

  try {
    const resp = await fetch(BACKEND_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ message: text })
    });

    if (!resp.ok) {
      // 嘗試解析回應 JSON 以提供更好錯誤資訊
      let errText = '伺服器回應錯誤';
      try {
        const errJson = await resp.json();
        if (errJson?.error) errText = errJson.error;
      } catch (e) { /* ignore */ }
      throw new Error(errText);
    }

    const j = await resp.json();
    removeTypingIndicator();
    appendMessage(j.reply || '抱歉，我暫時無法回答。', 'bot');
  } catch (err) {
    removeTypingIndicator();
    appendMessage('發生錯誤，請稍後再試。', 'bot');
    console.error('chat error:', err);
  } finally {
    setSendingState(false);
  }
}

// Enter 鍵送出
inputEl.addEventListener('keydown', (e) => {
  if (e.key === 'Enter' && !e.shiftKey) {
    e.preventDefault();
    sendMessage();
  }
});

sendBtn.addEventListener('click', sendMessage);

// ===== 匿名通報表單 =====
if (reportForm) {
  reportForm.addEventListener('submit', (e) => {
    e.preventDefault();
    const fd = new FormData(reportForm);
    const desc = (fd.get('desc') || '').toString().trim();

    if (!desc) {
      reportResult.textContent = '請提供事件描述以協助後續處理。';
      return;
    }

    // 這裡範例為前端顯示回饋，實際可 POST 到後端儲存或發郵件
    reportResult.textContent = '已收到通報，感謝你提供資訊。我們會盡快處理。';
    reportForm.reset();
  });
}

// ===== 初始樣例訊息 =====
document.addEventListener('DOMContentLoaded', () => {
  // 若沒有任何訊息，顯示歡迎訊息
  if (messagesEl && messagesEl.children.length === 0) {
    appendMessage('你好，我是線上支援聊天機器人。如果你正遭遇霸凌或需要協助，可以簡短描述情況。我會提供建議與資源。', 'bot');
  }
});
