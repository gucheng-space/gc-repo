// nuxt.config.ts
export default defineNuxtConfig({
modules: [
'@unocss/nuxt',
'@nuxt/content',
],

css: ['~/assets/css/main.css'],

content: {
highlight: {
theme: {
default: 'github-light',
dark: 'github-dark',
},
},
markdown: {
toc: {
depth: 3,
searchDepth: 3,
},
},
},
})

// ===================================
// assets/css/main.css
// ===================================
:root {
/_ CSS 变量作为后备方案 _/
--bg-primary: #fafaf8;
--bg-secondary: #f5f5f0;
--text-primary: #2a2823;
--text-secondary: #5a564a;
--link-default: #267658;
--link-hover: #1f5e48;
--code-bg: #f5f5f0;
}

.dark {
--bg-primary: #1a1916;
--bg-secondary: #252218;
--text-primary: #f5f5f0;
--text-secondary: #d4d2c4;
--link-default: #58b089;
--link-hover: #8bcdad;
--code-bg: #2a2823;
}

body {
background-color: var(--bg-primary);
color: var(--text-primary);
font-family: "Noto Serif SC", serif;
transition: background-color 0.3s, color 0.3s;
}

/_ 自定义滚动条 _/
::-webkit-scrollbar {
width: 8px;
height: 8px;
}

::-webkit-scrollbar-track {
background: var(--bg-secondary);
}

::-webkit-scrollbar-thumb {
background: rgba(149, 145, 125, 0.5);
border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
background: rgba(149, 145, 125, 0.7);
}

/_ ===================================
// components/ArticleCard.vue
// =================================== _/
<template>

  <article class="card-base p-6 hover:shadow-lg transition-shadow duration-300">
    <NuxtLink :to="`/blog/${article._path}`" class="block">
      <!-- 标题 -->
      <h2 class="text-2xl font-display text-primary-600 dark:text-primary-400 mb-3 hover:text-primary-700 transition-colors">
        {{ article.title }}
      </h2>
      
      <!-- 元信息 -->
      <div class="flex items-center gap-4 text-sm text-secondary mb-4">
        <time class="flex items-center gap-1">
          <span class="i-carbon-calendar w-4 h-4" />
          {{ formatDate(article.date) }}
        </time>
        <span class="flex items-center gap-1" v-if="article.readingTime">
          <span class="i-carbon-time w-4 h-4" />
          {{ article.readingTime }}
        </span>
      </div>
      
      <!-- 摘要 -->
      <p class="text-secondary leading-relaxed mb-4">
        {{ article.description }}
      </p>
      
      <!-- 标签 -->
      <div class="flex flex-wrap gap-2" v-if="article.tags">
        <span 
          v-for="tag in article.tags" 
          :key="tag"
          class="px-3 py-1 text-xs bg-secondary-100 dark:bg-secondary-900 text-secondary-700 dark:text-secondary-300 rounded-full"
        >
          #{{ tag }}
        </span>
      </div>
    </NuxtLink>
  </article>
</template>

<script setup lang="ts">
const props = defineProps<{
  article: any
}>()

const formatDate = (date: string) => {
  return new Date(date).toLocaleDateString('zh-CN', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
}
</script>

/_ ===================================
// pages/blog/[...slug].vue
// =================================== _/
<template>

  <div class="min-h-screen bg-neutral-50 dark:bg-neutral-950">
    <!-- 文章容器 -->
    <article class="container-article py-12">
      <!-- 文章头部 -->
      <header class="mb-12">
        <h1 class="text-4xl md:text-5xl font-display text-primary-600 dark:text-primary-400 mb-4">
          {{ article.title }}
        </h1>
        
        <div class="flex flex-wrap items-center gap-4 text-secondary mb-6">
          <time class="flex items-center gap-2">
            <span class="i-carbon-calendar w-5 h-5" />
            {{ formatDate(article.date) }}
          </time>
          <span class="flex items-center gap-2" v-if="article.author">
            <span class="i-carbon-user w-5 h-5" />
            {{ article.author }}
          </span>
        </div>
        
        <!-- 标签 -->
        <div class="flex flex-wrap gap-2" v-if="article.tags">
          <span 
            v-for="tag in article.tags" 
            :key="tag"
            class="px-4 py-2 bg-accent-100 dark:bg-accent-900 text-accent-700 dark:text-accent-300 rounded-lg text-sm font-medium"
          >
            #{{ tag }}
          </span>
        </div>
      </header>
      
      <!-- 文章内容 -->
      <div class="prose prose-neutral dark:prose-invert max-w-none">
        <ContentRenderer :value="article" class="article-content" />
      </div>
      
      <!-- 文章底部 -->
      <footer class="mt-16 pt-8 border-t border-neutral-300 dark:border-neutral-700">
        <div class="flex justify-between items-center">
          <NuxtLink 
            to="/blog" 
            class="btn-ghost flex items-center gap-2"
          >
            <span class="i-carbon-arrow-left w-5 h-5" />
            返回文章列表
          </NuxtLink>
          
          <button 
            @click="scrollToTop" 
            class="btn-ghost flex items-center gap-2"
          >
            <span class="i-carbon-arrow-up w-5 h-5" />
            回到顶部
          </button>
        </div>
      </footer>
    </article>
  </div>
</template>

<script setup lang="ts">
const route = useRoute()
const { data: article } = await useAsyncData('article', () => 
  queryContent(route.path).findOne()
)

if (!article.value) {
  throw createError({
    statusCode: 404,
    message: '文章未找到'
  })
}

const formatDate = (date: string) => {
  return new Date(date).toLocaleDateString('zh-CN', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
}

const scrollToTop = () => {
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

// SEO
useHead({
  title: article.value.title,
  meta: [
    { name: 'description', content: article.value.description },
    { property: 'og:title', content: article.value.title },
    { property: 'og:description', content: article.value.description },
  ]
})
</script>

<style scoped>
/* 自定义文章内容样式 */
.article-content :deep(h2) {
  @apply text-3xl font-display text-primary-600 dark:text-primary-400 mt-12 mb-6;
}

.article-content :deep(h3) {
  @apply text-2xl font-display text-primary-600 dark:text-primary-400 mt-8 mb-4;
}

.article-content :deep(p) {
  @apply text-lg leading-relaxed mb-6 text-primary;
}

.article-content :deep(a) {
  @apply link;
}

.article-content :deep(blockquote) {
  @apply border-l-4 border-primary-400 pl-6 py-2 italic text-secondary bg-neutral-100 dark:bg-neutral-900 rounded-r-md;
}

.article-content :deep(code) {
  @apply font-mono text-sm bg-neutral-200 dark:bg-neutral-800 px-2 py-1 rounded;
}

.article-content :deep(pre) {
  @apply bg-neutral-100 dark:bg-neutral-900 p-6 rounded-lg overflow-x-auto my-6 shadow-inner;
}

.article-content :deep(pre code) {
  @apply bg-transparent p-0;
}

.article-content :deep(img) {
  @apply rounded-lg shadow-paper my-8;
}

.article-content :deep(ul), 
.article-content :deep(ol) {
  @apply my-6 pl-6 space-y-2;
}

.article-content :deep(li) {
  @apply text-lg leading-relaxed;
}

.article-content :deep(hr) {
  @apply my-12 border-neutral-300 dark:border-neutral-700;
}
</style>

/_ ===================================
// content/blog/example-post.md
// 示例 Markdown 文章
// =================================== _/

---

title: 记忆中的那个夏天
description: 有些回忆，像老照片一样泛黄，却始终温暖
date: 2025-09-30
author: 你的名字
tags: [回忆, 夏天, 成长]

---

# 记忆中的那个夏天

有些记忆，像老照片一样泛黄，却始终温暖。

## 蝉鸣与西瓜

那年夏天，蝉鸣声特别响亮...

> 时光啊，你慢些走，让我再看看这些美好的瞬间。

代码示例：

\`\`\`javascript
const memories = {
summer: 'unforgettable',
feeling: 'nostalgic'
}
\`\`\`

## 未来的期待

虽然怀念过去，但我依然对未来充满好奇...
