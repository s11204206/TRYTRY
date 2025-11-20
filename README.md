# TRYTRY
<input type="text" id="message" placeholder="輸入訊息…" />
<button onclick="sendMessage()">送出</button>

<div id="reply" style="margin-top:10px; font-weight:bold;"></div>

<script>
async function sendMessage() {
  const userMessage = document.getElementById("message").value.trim();
  if (!userMessage) {
    alert("請輸入訊息！");
    return;
  }

  // 顯示載入中提示
  const replyDiv = document.getElementById("reply");
  replyDiv.innerText = "回覆中…";

  try {
    const response = await fetch("https://bullying-bot.onrender.com/reply", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ message: userMessage })
    });

    if (!response.ok) throw new Error("伺服器錯誤");

    const data = await response.json();
    replyDiv.innerText = data.reply;
  } catch (err) {
    replyDiv.innerText = "無法取得回覆，請稍後再試。";
    console.error(err);
  }

  // 清空輸入框
  document.getElementById("message").value = "";
}
</script>
