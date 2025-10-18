---
title: 致谢名单
aside: false
comment: true
---

<h1 class="centered-bold">如果对你有帮助，可以考虑请我喝杯咖啡</h1>

<div class="payment-container">
  <div class="payment-item">
    <p class="payment-label">微信 / Wechat</p>
    <img class="payment-image" src="" />
  </div>
  <div class="payment-item">
    <p class="payment-label">支付宝 / Alipay</p>
    <img class="payment-image" src="" />
  </div>
</div>

<style>
  .centered-bold {
    text-align: center; /* 居中 */
    font-weight: bold;  /* 加粗 */
  }

  /* 默认布局：并排排列 */
  .payment-container {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 80px;
    margin-top: 40px;
    flex-direction: row;
  }

  /* 每个支付项的样式 */
  .payment-item {
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;
  }

  .payment-label {
    font-weight: bold;
    margin: 0;
    line-height: 1.5;
  }

  .payment-image {
    width: 192px !important; /* 使用 !important 确保图片宽度 */
    border: 1px solid lightgrey;
  }

  /* 屏幕宽度小于768px时，调整为上下排列 */
  @media (max-width: 768px) {
    .payment-container {
      flex-direction: column;
      gap: 20px; /* 调整上下间距 */
    }
  }
</style>

<br><br>

<h1 class="centered-bold">感谢所有热心的朋友！</h1>

<div align="center"><strong>以下排名不分先后</strong></div>

| 昵称          | 捐助金额      | 日期            |  备注  |昵称         | 捐助金额      | 日期            |备注|
| :-----------: | :----------: | :-------------: |:-----------: | :-----------: | :----------: | :-------------: |:-------------: |
| 海阔蓝         | 0             | 2025-10-18     |技术支持           | VitePress           | 0         | 2015-10-17 |开源项目&技术支持|
| 洛谷：tsh_qwq  | 0            |2025-10-18       |技术支持
