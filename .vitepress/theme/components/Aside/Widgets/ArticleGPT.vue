<template>
  <div v-if="showGPT" class="article-gpt s-card">
    <div class="title">
      <span class="name" @click="router.go('/posts/2024/0218')">
        <i class="iconfont icon-robot"></i>
        文章摘要
        <i class="iconfont icon-up"></i>
      </span>
      <span :class="['logo', { loading }]" @click="showOther">
        {{ theme?.articleGPT?.name || 'ArticleGPT' }}
      </span>
    </div>
    <div class="content s-card">
      <span class="text">{{ abstractData === "" ? "加载中..." : abstractData }}</span>
      <span v-if="loading" class="point">|</span>
    </div>
    <div class="meta">
      <span class="tip">此内容根据文章生成，并经过人工审核，仅用于文章内容的解释与总结</span>
      <a
        href="https://eqnxweimkr5.feishu.cn/share/base/form/shrcnCXCPmxCKKJYI3RKUfefJre"
        class="report"
        target="_blank"
      >
        投诉
      </a>
    </div>
  </div>
</template>

<script setup>
const { frontmatter, theme, page } = useData();
const router = useRouter();

const loading = ref(true);
const waitTimeOut = ref(null);
const abstractData = ref("");
const showIndex = ref(0);
const showType = ref(false);

const showGPT = computed(() => {
  return frontmatter.value?.articleGPT !== undefined && frontmatter.value?.articleGPT !== false;
});

const getArticleContent = () => {
  if (typeof frontmatter.value?.articleGPT === "string") {
    return frontmatter.value.articleGPT;
  }
  
  if (page.value?.content) {
    let content = page.value.content;
    content = content.replace(/```[\s\S]*?```/g, "");
    content = content.replace(/`[^`]+`/g, "");
    content = content.replace(/[#*>\-+]/g, "");
    content = content.replace(/\s+/g, " ").trim();
    return content.slice(0, 3000);
  }
  
  return "";
};

const typeWriter = (text = null) => {
  try {
    const data = text || abstractData.value;
    if (!data) return false;
    if (showIndex.value < data.length) {
      abstractData.value += data.charAt(showIndex.value++);
      const delay = Math.random() * (150 - 30) + 30;
      setTimeout(() => {
        typeWriter(text);
      }, delay);
    } else {
      loading.value = false;
    }
  } catch (error) {
    loading.value = false;
    abstractData.value = "摘要生成失败";
    $message.error("摘要生成失败，请重试");
    console.error("摘要生成失败：", error);
  }
};

const fetchAbstract = async () => {
  const config = theme.value?.articleGPT;
  if (!config?.enable || !config?.api) {
    abstractData.value = "AI 摘要功能未启用或未配置 API";
    loading.value = false;
    return;
  }

  const content = getArticleContent();
  if (!content) {
    abstractData.value = "未能获取文章内容";
    loading.value = false;
    return;
  }

  try {
    const response = await fetch(config.api, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        ...(config.apiKey ? { Authorization: `Bearer ${config.apiKey}` } : {}),
      },
      body: JSON.stringify({
        content: content,
        model: config.model || "gpt-3.5-turbo",
      }),
    });

    if (!response.ok) {
      throw new Error(`API 请求失败: ${response.status}`);
    }

    const result = await response.json();
    const text = result.choices?.[0]?.message?.content || result.content || result.text || result.summary || "";

    if (text) {
      showIndex.value = 0;
      abstractData.value = "";
      setTimeout(() => {
        typeWriter(text);
      }, 100);
    } else {
      throw new Error("API 返回内容为空");
    }
  } catch (error) {
    loading.value = false;
    abstractData.value = "摘要生成失败";
    $message.error("摘要生成失败，请重试");
    console.error("摘要生成失败：", error);
  }
};

const initAbstract = () => {
  waitTimeOut.value = setTimeout(() => {
    fetchAbstract();
  }, Math.random() * (3800 - 2500) + 2500);
};

const showOther = () => {
  if (loading.value) return false;
  const text = `本文档使用 AI 技术自动生成摘要，API 配置地址在主题配置文件的 articleGPT.api 字段中修改。`;
  showIndex.value = 0;
  loading.value = true;
  abstractData.value = "";
  if (!showType.value) {
    showType.value = true;
    typeWriter(text);
  } else {
    typeWriter();
    showType.value = false;
  }
};

onMounted(() => {
  if (showGPT.value) initAbstract();
});

onBeforeUnmount(() => {
  clearTimeout(waitTimeOut.value);
});
</script>

<style lang="scss" scoped>
.article-gpt {
  margin-top: 1.2rem;
  background-color: var(--main-card-second-background);
  user-select: none;
  cursor: auto;
  .title {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 0.8rem;
    padding: 0 8px;
    .name {
      display: flex;
      align-items: center;
      color: var(--main-color);
      font-weight: bold;
      cursor: pointer;
      .icon-robot {
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 18px;
        font-weight: normal;
        width: 26px;
        height: 26px;
        color: var(--main-card-background);
        background-color: var(--main-color);
        border-radius: 50%;
        margin-right: 8px;
      }
      .icon-up {
        font-weight: normal;
        font-size: 12px;
        margin-left: 6px;
        opacity: 0.6;
        color: var(--main-color);
        transform: rotate(90deg);
      }
    }
    .logo {
      padding: 4px 10px;
      font-size: 12px;
      color: var(--main-card-background);
      background-color: var(--main-color);
      border-radius: 25px;
      font-weight: bold;
      cursor: pointer;
      &.loading {
        animation: loading 1s infinite;
        cursor: not-allowed;
      }
    }
  }
  .content {
    cursor: auto;
    .point {
      color: var(--main-color);
      font-weight: bold;
      margin-left: 4px;
      animation: loading 0.8s infinite;
    }
  }
  .meta {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
    margin-top: 1rem;
    padding: 0 8px;
    font-size: 12px;

    .tip {
      opacity: 0.6;
    }
    .report {
      white-space: nowrap;
      margin-left: 12px;
      opacity: 0.8;
    }
  }
}
</style>