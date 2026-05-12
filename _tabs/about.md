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
#cat-container {
  position: fixed;
  bottom: 20px;
  right: 20px;
  z-index: 9999;
  cursor: pointer;
  font-size: 10px;
  user-select: none;
}
.cat-body {
  width: 80px;
  height: 60px;
  background: linear-gradient(135deg, #ff9a56 0%, #ff6b35 100%);
  border-radius: 50% 50% 45% 45%;
  position: relative;
  animation: cat-bounce 2s ease-in-out infinite;
}
.cat-head {
  width: 50px;
  height: 45px;
  background: linear-gradient(135deg, #ff9a56 0%, #ff6b35 100%);
  border-radius: 50%;
  position: absolute;
  top: -25px;
  left: 15px;
}
.cat-ear {
  width: 0;
  height: 0;
  border-left: 8px solid transparent;
  border-right: 8px solid transparent;
  border-bottom: 15px solid #ff6b35;
  position: absolute;
  top: -8px;
}
.cat-ear.left { left: 5px; transform: rotate(-15deg); }
.cat-ear.right { right: 5px; transform: rotate(15deg); }
.cat-eye {
  width: 8px;
  height: 10px;
  background: #333;
  border-radius: 50%;
  position: absolute;
  top: 15px;
  animation: cat-blink 4s ease-in-out infinite;
}
.cat-eye.left { left: 12px; }
.cat-eye.right { right: 12px; }
.cat-eye::after {
  content: '';
  width: 3px;
  height: 3px;
  background: white;
  border-radius: 50%;
  position: absolute;
  top: 2px;
  left: 2px;
}
.cat-nose {
  width: 6px;
  height: 4px;
  background: #ff4757;
  border-radius: 50%;
  position: absolute;
  top: 25px;
  left: 22px;
}
.cat-mouth {
  position: absolute;
  top: 28px;
  left: 22px;
  width: 6px;
  height: 6px;
  border-left: 1px solid #333;
  border-right: 1px solid #333;
  border-bottom: 1px solid #333;
  border-radius: 0 0 50% 50%;
}
.cat-whisker {
  position: absolute;
  width: 15px;
  height: 1px;
  background: #333;
  top: 25px;
}
.cat-whisker.l1 { left: -12px; transform: rotate(-10deg); }
.cat-whisker.l2 { left: -14px; top: 28px; transform: rotate(0deg); }
.cat-whisker.l3 { left: -12px; top: 31px; transform: rotate(10deg); }
.cat-whisker.r1 { right: -12px; transform: rotate(10deg); }
.cat-whisker.r2 { right: -14px; top: 28px; transform: rotate(0deg); }
.cat-whisker.r3 { right: -12px; top: 31px; transform: rotate(-10deg); }
.cat-tail {
  width: 40px;
  height: 8px;
  background: #ff6b35;
  border-radius: 4px;
  position: absolute;
  bottom: 20px;
  right: -25px;
  transform-origin: left center;
  animation: cat-tail-wag 1s ease-in-out infinite;
}
.cat-paw {
  width: 15px;
  height: 12px;
  background: #ff9a56;
  border-radius: 50%;
  position: absolute;
  bottom: -5px;
}
.cat-paw.front.left { left: 10px; }
.cat-paw.front.right { right: 10px; }
@keyframes cat-bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-5px); }
}
@keyframes cat-blink {
  0%, 45%, 55%, 100% { transform: scaleY(1); }
  50% { transform: scaleY(0.1); }
}
@keyframes cat-tail-wag {
  0%, 100% { transform: rotate(-10deg); }
  50% { transform: rotate(10deg); }
}
#cat-speech {
  position: absolute;
  top: -40px;
  left: 50%;
  transform: translateX(-50%);
  background: white;
  padding: 5px 10px;
  border-radius: 10px;
  font-size: 12px;
  white-space: nowrap;
  opacity: 0;
  transition: opacity 0.3s;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}
#cat-speech::after {
  content: '';
  position: absolute;
  bottom: -8px;
  left: 50%;
  transform: translateX(-50%);
  border: 4px solid transparent;
  border-top-color: white;
}
#cat-speech.show { opacity: 1; }
</style>

<div id="cat-container">
  <div id="cat-speech">喵~ 你好呀！🐱</div>
  <div class="cat-body">
    <div class="cat-head">
      <div class="cat-ear left"></div>
      <div class="cat-ear right"></div>
      <div class="cat-eye left"></div>
      <div class="cat-eye right"></div>
      <div class="cat-nose"></div>
      <div class="cat-mouth"></div>
      <div class="cat-whisker l1"></div>
      <div class="cat-whisker l2"></div>
      <div class="cat-whisker l3"></div>
      <div class="cat-whisker r1"></div>
      <div class="cat-whisker r2"></div>
      <div class="cat-whisker r3"></div>
    </div>
    <div class="cat-paw front left"></div>
    <div class="cat-paw front right"></div>
    <div class="cat-tail"></div>
  </div>
</div>

<script>
const cat = document.getElementById('cat-container');
const speech = document.getElementById('cat-speech');
const messages = [
  '喵~ 你好呀！',
  '摸摸我！🐱',
  '今天也要加油哦~',
  '一起学习吧！',
  '好无聊啊...',
  '给我看看你的代码？',
  '喵喵喵~',
  '你好呀！我是博客小猫',
  '点我有惊喜哦~'
];

cat.addEventListener('click', () => {
  const msg = messages[Math.floor(Math.random() * messages.length)];
  speech.textContent = msg;
  speech.classList.add('show');
  setTimeout(() => speech.classList.remove('show'), 2000);
});

cat.addEventListener('mouseenter', () => {
  speech.textContent = '喵~ 摸摸我！';
  speech.classList.add('show');
});
cat.addEventListener('mouseleave', () => {
  speech.classList.remove('show');
});
</script>