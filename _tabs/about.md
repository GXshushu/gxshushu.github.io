---
title: 关于
icon: fas fa-info
order: 4
---
- ### Hi there 👋, I'm C9xx!

  - 🏫  _MSc in Cybersecurity_.
  - 💻  I work on Java / Python / Markdown.
  - 🧠  I used to take internship as a SDET at Tencent and Monee.
  - 🏖️  I will go to Shenzhen and work as a SDET at Monee.

---

## 📊 GitHub 贡献图

<img src="https://ghchart.rshah.org/gxshushu" alt="C9xx GitHub Contribution Chart">

*数据来源：[ghchart.rshah.org](https://ghchart.rshah.org)*

---

<style>
#site-pet {
  position: fixed;
  right: 20px;
  bottom: 20px;
  z-index: 9999;
  cursor: pointer;
  font-size: 12px;
  text-align: center;
}
#site-pet img {
  width: 80px;
  height: auto;
  animation: pet-bounce 1s ease-in-out infinite;
}
#site-pet .speech {
  background: white;
  padding: 5px 10px;
  border-radius: 10px;
  margin-bottom: 5px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  display: none;
}
#site-pet:hover .speech {
  display: block;
}
@keyframes pet-bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-5px); }
}
</style>

<div id="site-pet">
  <div class="speech">喵~ 摸摸我！🐱</div>
  <img src="https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif" alt="pet">
</div>

<script>
const pet = document.getElementById('site-pet');
const messages = ['喵~ 你好呀！', '今天也要加油哦~', '一起学习吧！', '好无聊啊...', '给我看看代码？'];
pet.addEventListener('click', () => {
  const msg = messages[Math.floor(Math.random() * messages.length)];
  pet.querySelector('.speech').textContent = msg;
  pet.querySelector('.speech').style.display = 'block';
  setTimeout(() => {
    pet.querySelector('.speech').style.display = 'none';
  }, 2000);
});
</script>