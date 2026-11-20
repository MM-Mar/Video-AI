﻿<template>
  <div class="app-stage">
    <div class="ambient-noise"></div>
    <div class="ambient-glow"></div>

    <header class="navbar">
      <div class="nav-content">
        <div class="brand">
          <span class="brand-mark">VA</span>
          <span class="brand-name">Video&nbsp;AI</span>
          <span class="beta-badge">Beta</span>
        </div>

        <div class="nav-controls">
          <button v-if="!currentUser" class="auth-btn" @click="openAuthModal">
            <span class="btn-icon">
              <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"></path><circle cx="12" cy="7" r="4"></circle></svg>
            </span>
            登录 / 注册
          </button>

          <div v-else class="user-profile">
            <span class="user-name">{{ currentUser.nickname }}</span>
            <button class="logout-btn" @click="logout" title="退出登录" aria-label="退出登录">
              <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"></path><polyline points="16 17 21 12 16 7"></polyline><line x1="21" y1="12" x2="9" y2="12"></line></svg>
            </button>
          </div>

          <div class="status-pill" :class="{ 'is-active': uploading }" role="status" aria-live="polite">
            <div class="status-dot"></div>
            <span class="status-text">{{ systemStatusText }}</span>
          </div>
        </div>
      </div>
    </header>

    <main class="main-container">
      <section class="hero-section">
        <h1 class="slogan-main">让每一段视频<br/>都能被精准理解</h1>
        <p class="slogan-sub">上传视频 · 设定目标 · Agent 自动生成带证据的结构化洞察</p>

        <div class="upload-wrapper">
          <input
              type="file"
              id="file-input"
              @change="handleFileChange"
              accept="video/*"
              hidden
          />

          <div
              class="upload-magnet"
              :class="{ 'processing': uploading, 'is-dragover': isDragOver }"
              @dragenter.prevent="handleDragEnter"
              @dragover.prevent="isDragOver = true"
              @dragleave.prevent="handleDragLeave"
              @drop.prevent="handleDrop"
          >
            <div class="split-container" v-if="!uploading">

              <label for="file-input" class="skew-pane pane-local">
                <div class="pane-content unskew">
                  <div class="magnet-icon">
                    <svg width="42" height="42" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path><polyline points="17 8 12 3 7 8"></polyline><line x1="12" y1="3" x2="12" y2="15"></line></svg>
                  </div>
                  <span class="magnet-title">本地上传</span>
                  <span class="magnet-desc">{{ isDragOver ? '松手上传' : '点击 / 拖拽本地文件' }}</span>
                </div>
              </label>

              <div class="split-gap"></div>

              <div class="skew-pane pane-url">
                <div class="pane-content unskew">
                  <div class="magnet-icon">
                    <svg width="42" height="42" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"></circle><line x1="2" y1="12" x2="22" y2="12"></line><path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1 4-10z"></path></svg>
                  </div>
                  <span class="magnet-title">网络链接</span>
                  <span class="magnet-desc">B站 / YouTube / 抖音</span>

                  <div class="url-input-box" @click.stop>
                    <input
                        v-model="videoUrl"
                        type="text"
                        inputmode="url"
                        autocomplete="off"
                        spellcheck="false"
                        placeholder="粘贴视频链接..."
                        aria-label="视频链接"
                        :disabled="uploading"
                        @keyup.enter="handleUrlUpload"
                    />
                    <button class="url-go-btn" :disabled="uploading" @click="handleUrlUpload" aria-label="解析视频链接">
                      <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"><polyline points="9 18 15 12 9 6"></polyline></svg>
                    </button>
                  </div>
                </div>
              </div>

            </div>

            <div class="magnet-content busy" v-else>
              <div class="quantum-loader"></div>
              <span class="busy-text">{{ uploadProgress.label }}</span>
              <span v-if="uploadProgress.filename" class="busy-file">{{ uploadProgress.filename }}</span>
              <div
                  v-if="uploadProgress.percent !== null"
                  class="upload-progress"
                  role="progressbar"
                  aria-label="视频上传进度"
                  aria-valuemin="0"
                  aria-valuemax="100"
                  :aria-valuenow="uploadProgress.percent"
              >
                <span :style="{ width: `${uploadProgress.percent}%` }"></span>
              </div>
              <span v-if="uploadProgress.detail" class="busy-stat" aria-live="polite">{{ uploadProgress.detail }}</span>
              <span v-if="uploadProgress.warning" class="busy-warning" role="status">{{ uploadProgress.warning }}</span>
              <div v-if="uploadAbort" class="busy-actions">
                <button type="button" @click="cancelUpload">取消上传</button>
              </div>
            </div>

            <div class="border-glow"></div>
          </div>

          <div v-if="resumableFile && !uploading" class="upload-resume" role="status">
            <span>{{ resumeHint }}</span>
            <button type="button" @click="resumeUpload">继续上传</button>
            <button type="button" @click="discardResumableUpload">重新开始</button>
          </div>
        </div>
        <transition name="toast-pop">
          <div
              v-if="message"
              class="notification-bar"
              :class="{ 'error': messageIsError }"
              :role="messageIsError ? 'alert' : 'status'"
              :aria-live="messageIsError ? 'assertive' : 'polite'"
              :title="messageIsError ? '点击关闭这条提示' : null"
              @click="dismissMessage"
          >
            {{ message }}
          </div>
        </transition>
      </section>

      <section v-if="list.length > 0" class="workspace-section">
        <div class="section-header">
          <div class="library-title">
            <h3>视频资料库</h3>
            <div class="count-chip">{{ list.length }} 个视频</div>
          </div>
          <label class="library-search">
            <svg width="17" height="17" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line></svg>
            <input v-model="searchQuery" type="search" placeholder="搜索视频名称" aria-label="搜索视频名称" />
          </label>
        </div>
        <div class="card-grid">
          <div v-for="item in visibleList" :key="item.id" class="project-card">

            <button
                class="delete-btn"
                :disabled="deletingId === item.id"
                :title="deletingId === item.id ? '正在删除…' : '删除视频'"
                :aria-label="`删除 ${item.filename}`"
                @click.stop="deleteItem(item)"
            >
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <polyline points="3 6 5 6 21 6"></polyline><path d="M19 6l-1 14H6L5 6"></path><path d="M8 6V4h8v2"></path>
              </svg>
            </button>
            <div class="card-meta">
              <div class="meta-icon">
                <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><polygon points="23 7 16 12 23 17 23 7"></polygon><rect x="1" y="5" width="15" height="14" rx="2" ry="2"></rect></svg>
              </div>
              <div class="meta-info">
                <div class="filename-mask" :title="item.filename">{{ item.filename }}</div>
                <div class="meta-tags">
                  <span class="time-tag">{{ formatTime(item.uploadTime) }}</span>
                  <span
                      class="status-indicator"
                      :class="cardStatusClass(item)"
                      :title="cardStatusTitle(item)"
                  >
                    {{ cardStatusLabel(item) }}
                  </span>
                </div>
              </div>
            </div>

            <div class="action-dock">
              <button
                  class="dock-item"
                  :disabled="item.status !== 'COMPLETED'"
                  :title="actionTitle(item, '下载音频')"
                  @click="downloadAudio(item)"
              >
                <span class="item-icon">
                  <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M9 18V5l12-2v13"></path><circle cx="6" cy="18" r="3"></circle><circle cx="18" cy="16" r="3"></circle></svg>
                </span>
                <span class="item-label">下载音频</span>
              </button>

              <button
                  class="dock-item"
                  :disabled="item.status !== 'COMPLETED'"
                  :title="actionTitle(item, '提取文字')"
                  @click="transcribe(item.id)"
              >
                <span class="item-icon">
                  <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path><polyline points="14 2 14 8 20 8"></polyline><line x1="16" y1="13" x2="8" y2="13"></line><line x1="16" y1="17" x2="8" y2="17"></line><polyline points="10 9 9 9 8 9"></polyline></svg>
                </span>
                <span class="item-label">提取文字</span>
              </button>

              <button
                  class="dock-item ai-core"
                  :disabled="item.status !== 'COMPLETED'"
                  :title="actionTitle(item, '打开 Video Agent')"
                  @click="openAgent(item)"
              >
                <span class="item-icon">
                  <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><rect x="4" y="4" width="16" height="16" rx="2" ry="2"></rect><rect x="9" y="9" width="6" height="6"></rect><line x1="9" y1="1" x2="9" y2="4"></line><line x1="15" y1="1" x2="15" y2="4"></line><line x1="9" y1="20" x2="9" y2="23"></line><line x1="15" y1="20" x2="15" y2="23"></line><line x1="20" y1="9" x2="23" y2="9"></line><line x1="20" y1="14" x2="23" y2="14"></line><line x1="1" y1="9" x2="4" y2="9"></line><line x1="1" y1="14" x2="4" y2="14"></line></svg>
                </span>
                <div class="label-group">
                  <span class="item-label">Video Agent</span>
                </div>
                <div class="shimmer"></div>
              </button>
            </div>
          </div>
        </div>
        <div v-if="visibleList.length === 0" class="library-empty">
          <p>没有找到“{{ searchQuery }}”</p>
          <button type="button" @click="searchQuery = ''">清除搜索</button>
        </div>
      </section>

      <div class="sidebar-backdrop" v-if="sidebar.visible" @click="closeSidebar"></div>
      <div
          ref="sidebarPanel"
          class="sidebar-panel"
          :class="{ 'is-open': sidebar.visible }"
          :inert="!sidebar.visible"
          role="dialog"
          aria-modal="true"
          tabindex="-1"
          :aria-label="sidebar.title || '任务详情'"
      >
        <div class="sidebar-header">
          <div class="sidebar-title">
            <span class="icon" v-if="sidebar.type === 'ai'">
              <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M2 12h2"></path><path d="M20 12h2"></path><path d="M12 2v2"></path><path d="M12 20v2"></path><path d="M20.2 6.47l-1.4 1.4"></path><path d="M15.9 5.35l-1.4-1.4"></path><path d="M9 11a3 3 0 1 0 6 0a3 3 0 0 0-6 0"></path></svg>
            </span>
            <span class="icon" v-else>
              <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path><polyline points="14 2 14 8 20 8"></polyline><line x1="16" y1="13" x2="8" y2="13"></line><line x1="16" y1="17" x2="8" y2="17"></line><polyline points="10 9 9 9 8 9"></polyline></svg>
            </span>
            {{ sidebar.title }}
          </div>
          <button class="close-btn" @click="closeSidebar" aria-label="关闭分析面板">×</button>
        </div>
        <div ref="sidebarBody" class="sidebar-body">
          <div v-if="sidebar.type === 'ai'" class="video-evidence">
            <video
                v-if="sidebar.playbackUrl"
                ref="videoPlayer"
                :src="sidebar.playbackUrl"
                controls
                playsinline
                preload="metadata"
                @error="handlePlaybackError"
            ></video>
            <div v-else-if="sidebar.playbackLoading" class="video-evidence-loading">正在载入原视频...</div>
            <div v-else-if="sidebar.playbackError" class="video-evidence-error" role="alert">
              <span>{{ sidebar.playbackError }}</span>
              <button type="button" @click="retryPlayback">重新加载</button>
            </div>
            <p v-if="sidebar.playbackUrl">点击分析结果中的时间戳，可跳转到对应画面</p>
          </div>
          <div v-if="sidebar.type === 'ai' && sidebar.mode === 'compose'" class="agent-composer">
            <p class="agent-caption">选择分析模式（决定产物形态）</p>
            <div class="goal-presets agent-mode-row">
              <button
                  v-for="m in analysisModes"
                  :key="m.value"
                  :class="{ active: sidebar.analysisMode === m.value }"
                  @click="sidebar.analysisMode = m.value"
              >
                <strong>{{ m.title }}</strong>
                <span>{{ m.description }}</span>
              </button>
            </div>
            <p class="agent-caption">告诉 Agent 你希望从视频中得到什么产物</p>
            <p v-if="sidebar.error" class="inline-error" role="alert">{{ sidebar.error }}</p>
            <textarea
                v-model="sidebar.goal"
                maxlength="500"
                placeholder="例如：梳理核心观点，给出带时间戳的证据和可执行建议（Ctrl / ⌘ + Enter 提交）"
                @keydown.ctrl.enter.prevent="submitAgent"
                @keydown.meta.enter.prevent="submitAgent"
            ></textarea>
            <p v-if="sidebar.goal.length > 400" class="field-counter">已输入 {{ sidebar.goal.length }} / 500 字</p>
            <div class="goal-presets">
              <button
                  v-for="preset in goalPresets"
                  :key="preset.title"
                  :class="{ active: sidebar.goal === preset.prompt }"
                  @click="sidebar.goal = preset.prompt"
              >
                <strong>{{ preset.title }}</strong>
                <span>{{ preset.description }}</span>
              </button>
            </div>
            <button class="agent-run-btn" :disabled="!sidebar.goal.trim()" @click="submitAgent">
              {{ sidebar.error ? '重新分析' : '开始分析' }}
            </button>
          </div>

          <div v-else-if="sidebar.loading" class="agent-running">
            <div class="loading-state">
              <div class="quantum-loader small"></div>
              <p aria-live="polite">{{ loadingHeadline }}</p>
              <p v-if="sidebar.streamOffline" class="stream-offline" role="status">
                连接中断，正在自动重连（第 {{ sidebar.streamRetry }} 次）· 任务仍在服务端继续
              </p>
              <p class="loading-hint">可以关闭本面板，任务会在后台继续，完成后会通知你</p>
            </div>
            <div v-if="sidebar.plan?.tasks?.length" class="agent-meta-block">
              <span class="meta-label">任务计划</span>
              <ol><li v-for="task in sidebar.plan.tasks" :key="task">{{ task }}</li></ol>
            </div>
            <div v-if="traceStages.length" class="agent-meta-block">
              <span class="meta-label">已完成阶段</span>
              <div class="stage-list"><span v-for="stage in traceStages" :key="stage[0]">{{ stage[0] }} · {{ stage[1] }}</span></div>
            </div>
          </div>

          <div v-else>
            <div v-if="sidebar.type === 'ai'">
              <div class="result-actions">
                <button type="button" @click="startNewAnalysis">更换产物</button>
                <button type="button" :disabled="!sidebar.content" @click="copyResult">复制结果</button>
                <button type="button" :disabled="!sidebar.content" @click="downloadResult">导出 Markdown</button>
              </div>
              <div class="evidence-search">
                <div class="evidence-search-form">
                  <input
                      v-model="sidebar.evidenceQuery"
                      aria-label="视频证据检索"
                      maxlength="500"
                      placeholder="定位 PPT、字幕、代码或某段讲解"
                      @keyup.enter="searchEvidence"
                  />
                  <button type="button" :disabled="sidebar.evidenceLoading || !sidebar.evidenceQuery.trim()" @click="searchEvidence">
                    {{ sidebar.evidenceLoading ? '检索中' : '定位证据' }}
                  </button>
                </div>
                <p v-if="sidebar.evidenceError" class="evidence-search-error" aria-live="polite">{{ sidebar.evidenceError }}</p>
                <div v-if="sidebar.evidenceResults.length" class="evidence-search-results" aria-live="polite">
                  <button
                      v-for="hit in sidebar.evidenceResults"
                      :key="`${hit.startMs}-${hit.endMs}`"
                      type="button"
                      :title="hit.snippet || '该时间段暂无可展示文本'"
                      @click="seekToEvidence(hit.startMs)"
                  >
                    <strong>{{ formatEvidenceTime(hit.startMs) }}</strong>
                    <small>{{ hit.source || '视频证据' }}</small>
                    <span>{{ hit.snippet || '该时间段暂无可展示文本' }}</span>
                  </button>
                </div>
              </div>
              <div class="markdown-content" v-html="renderedMarkdown" @click="seekEvidence"></div>
              <details v-if="sidebar.plan?.tasks?.length || traceStages.length" class="agent-inspector">
                <summary>分析详情</summary>
                <div class="agent-inspector-content">
                <div v-if="sidebar.plan?.tasks?.length" class="agent-meta-block">
                  <span class="meta-label">Planner 任务</span>
                  <div v-if="sidebar.editingPlan" class="plan-editor">
                    <div v-for="(_, index) in sidebar.planDraft" :key="index" class="plan-editor-row">
                      <input v-model="sidebar.planDraft[index]" maxlength="500" :aria-label="`任务 ${index + 1}`" />
                      <button type="button" title="删除任务" @click="removePlanTask(index)">×</button>
                    </div>
                    <button v-if="sidebar.planDraft.length < 5" type="button" @click="addPlanTask">添加任务</button>
                    <div class="plan-editor-actions">
                      <button type="button" @click="cancelPlanEdit">取消</button>
                      <button type="button" :disabled="sidebar.rerunLoading" @click="rerunWithPlan">
                        {{ sidebar.rerunLoading ? '提交中' : '按新计划重跑' }}
                      </button>
                    </div>
                  </div>
                  <template v-else>
                    <ol><li v-for="task in sidebar.plan.tasks" :key="task">{{ task }}</li></ol>
                    <button type="button" class="plan-edit-trigger" @click="startPlanEdit">调整计划</button>
                  </template>
                </div>
                <div v-if="traceStages.length" class="agent-meta-block">
                  <span class="meta-label">执行轨迹</span>
                  <div class="stage-list"><span v-for="stage in traceStages" :key="stage[0]">{{ stage[0] }} · {{ stage[1] }}</span></div>
                </div>
                <div v-if="sidebar.evaluation && Object.keys(sidebar.evaluation).length" class="quality-row">
                  <span>结构完整 {{ sidebar.evaluation.structuredValid ? '通过' : '待完善' }}</span>
                  <span>证据支持 {{ formatPercent(sidebar.evaluation.evidenceSupportRate) }}</span>
                  <span>Critic {{ sidebar.evaluation.criticPassed ? '通过' : '达到轮次上限' }}</span>
                </div>
                </div>
              </details>
              <div class="follow-up-box">
                <textarea
                    v-model="sidebar.followUp"
                    maxlength="500"
                    placeholder="基于视频继续追问...（Ctrl / ⌘ + Enter 发送）"
                    @keydown.ctrl.enter.prevent="submitFollowUp"
                    @keydown.meta.enter.prevent="submitFollowUp"
                ></textarea>
                <button :disabled="sidebar.followUpLoading || !sidebar.followUp.trim()" @click="submitFollowUp">
                  {{ sidebar.followUpLoading ? '分析中' : '追问' }}
                </button>
              </div>
              <div class="feedback-row">
                <span>这个结果有帮助吗？</span>
                <button :disabled="sidebar.feedbackLoading" :class="{ active: sidebar.feedback === 1 }" :aria-pressed="sidebar.feedback === 1" @click="sendFeedback(1)" title="有帮助">赞</button>
                <button :disabled="sidebar.feedbackLoading" :class="{ active: sidebar.feedback === -1 }" :aria-pressed="sidebar.feedback === -1" @click="sendFeedback(-1)" title="需改进">踩</button>
              </div>
            </div>
            <div v-else class="text-content">
              <p v-if="sidebar.error" class="inline-error" role="alert">{{ sidebar.error }}</p>
              <template v-if="sidebar.content">
                <div class="result-actions">
                  <button type="button" @click="copyResult">复制全文</button>
                  <button type="button" @click="downloadResult">导出文本</button>
                </div>
                <p class="text-meta">{{ transcriptMeta }}</p>
                <pre>{{ sidebar.content }}</pre>
              </template>
              <p v-else-if="!sidebar.error" class="text-meta">这个视频还没有可展示的转写文本。</p>
            </div>
          </div>
        </div>
      </div>

      <div v-if="showAuthModal" class="auth-backdrop" @click.self="closeAuthModal">
        <div
            ref="authPanel"
            class="auth-panel"
            role="dialog"
            aria-modal="true"
            aria-labelledby="auth-title"
            @keydown="trapAuthFocus"
        >
          <div class="auth-header">
            <h2 id="auth-title" class="auth-title">{{ authMode === 'login' ? '用户登录' : '新用户注册' }}</h2>
            <button class="close-btn" @click="closeAuthModal" aria-label="关闭登录窗口">×</button>
          </div>
          <form class="auth-body" @submit.prevent="handleAuth">
            <div class="input-group">
              <label for="auth-username">用户名</label>
              <input id="auth-username" v-model="authForm.username" type="text" placeholder="输入账号" autocomplete="username" autofocus />
            </div>
            <div class="input-group">
              <label for="auth-password">密码</label>
              <input id="auth-password" v-model="authForm.password" type="password" placeholder="输入密码" :autocomplete="authMode === 'login' ? 'current-password' : 'new-password'" />
            </div>
            <div class="input-group" v-if="authMode === 'register'">
              <label for="auth-nickname">昵称</label>
              <input id="auth-nickname" v-model="authForm.nickname" type="text" placeholder="设置一个好听的名字" autocomplete="nickname" />
            </div>
            <div class="auth-action">
              <button type="submit" class="cyber-btn" :disabled="authLoading">
                <span v-if="!authLoading">{{ authMode === 'login' ? '立即登录' : '提交注册' }}</span>
                <span v-else>请求处理中...</span>
              </button>
            </div>
            <div class="auth-toggle">
              <span class="toggle-text">{{ authMode === 'login' ? '没有账号?' : '已有账号?' }}</span>
              <button type="button" class="toggle-link" @click="switchAuthMode()">{{ authMode === 'login' ? '去注册' : '去登录' }}</button>
            </div>
            <p
                v-if="authMessage"
                class="auth-msg"
                :class="{'error': authError}"
                :role="authError ? 'alert' : 'status'"
                aria-live="polite"
            >{{ authMessage }}</p>
          </form>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup>
import { computed, nextTick, ref, watch, onMounted, onUnmounted } from 'vue'
import { apiRequest, clearAuthToken, hasAuthToken, setAuthToken } from './api'
import {
  forgetUploadProgress,
  formatBytes,
  formatDurationText,
  hasUploadProgress,
  uploadVideoInChunks,
  validateVideoFile
} from './chunkUpload'
import { DEMO_ITEM } from './demoData'
import { createTaskStreams } from './taskEvents'
import { useAnalysisWorkspace } from './useAnalysisWorkspace'

// --- 变量定义 ---
const DEMO_MODE = new URLSearchParams(window.location.search).has('demo')
const MESSAGE_TIMEOUT_MS = 4000
const file = ref(null)
const videoUrl = ref('')
const message = ref('')
const messageIsError = ref(false)
const uploading = ref(false)
const uploadProgress = ref({ label: '准备上传', filename: '', percent: null, detail: '', warning: '' })
const uploadAbort = ref(null)
const resumableFile = ref(null)
const resumableChunks = ref({ done: 0, total: 0 })
const list = ref([])
const searchQuery = ref('')
const videoPlayer = ref(null)
const sidebarPanel = ref(null)
const sidebarBody = ref(null)
const authPanel = ref(null)
const deletingId = ref(null)
const isOffline = ref(typeof navigator !== 'undefined' && navigator.onLine === false)
const activeTasks = ref([])
const elapsedSeconds = ref(0)
const visibleList = computed(() => {
  const query = searchQuery.value.trim().toLocaleLowerCase()
  if (!query) return list.value
  return list.value.filter(item => item.filename?.toLocaleLowerCase().includes(query))
})
const isDragOver = ref(false)
const currentUser = ref(null)
const showAuthModal = ref(false)
const authMode = ref('login')
const authLoading = ref(false)
const authMessage = ref('')
const authError = ref(false)
const authForm = ref({ username: '', password: '', nickname: '' })
const taskStreams = createTaskStreams({
  onActiveChange: tasks => { activeTasks.value = tasks }
})
const VIDEO_EXTENSIONS = new Set(['mp4', 'mov', 'mkv', 'avi', 'webm', 'm4v'])
let dragDepth = 0
let messageTimer = null
let elapsedTimer = null
let lastUploadProgress = {}
let focusBeforeAuth = null
let focusBeforeSidebar = null

// --- 状态展示 ---
const activeTaskOf = mediaId => activeTasks.value.find(task => String(task.id) === String(mediaId))

const systemStatusText = computed(() => {
  if (isOffline.value) return '网络已断开'
  if (uploading.value) {
    return uploadProgress.value.percent !== null
      ? `上传 ${uploadProgress.value.percent}%`
      : '处理中'
  }
  if (activeTasks.value.length) return `后台任务 ${activeTasks.value.length}`
  return '系统就绪'
})

const cardStatusClass = item => {
  if (activeTaskOf(item.id)) return 'processing'
  return mediaStatusClass(item.status)
}
const cardStatusLabel = item => {
  const task = activeTaskOf(item.id)
  if (task) return task.type === 'ai' ? 'ANALYZING' : 'TRANSCRIBING'
  return mediaStatusLabel(item.status)
}
const cardStatusTitle = item => {
  const task = activeTaskOf(item.id)
  if (!task) return null
  return task.type === 'ai'
    ? 'AI 分析正在后台执行，完成后会提示你'
    : '文字提取正在后台执行，完成后会提示你'
}
const actionTitle = (item, label) => item.status === 'COMPLETED'
  ? null
  : `视频尚未处理完成，暂时无法${label}`

const elapsedLabel = computed(() => {
  const total = elapsedSeconds.value
  if (total < 1) return ''
  const minutes = String(Math.floor(total / 60)).padStart(2, '0')
  return `${minutes}:${String(total % 60).padStart(2, '0')}`
})

const loadingHeadline = computed(() => {
  const fallback = sidebar.value.type === 'ai'
    ? 'Agent 正在分析视频证据'
    : '正在识别视频语音'
  const headline = sidebar.value.statusMessage || fallback
  // 用“已等待”而不是“已运行”：接管历史任务时计时是从打开面板算起的。
  return elapsedLabel.value ? `${headline} · 已等待 ${elapsedLabel.value}` : headline
})

const transcriptMeta = computed(() => {
  const length = sidebar.value.content?.length || 0
  if (!length) return ''
  return `共 ${length.toLocaleString('zh-CN')} 字`
})

const resumeHint = computed(() => {
  const target = resumableFile.value
  if (!target) return ''
  const { done, total } = resumableChunks.value
  const progress = total ? `已完成 ${Math.round((done / total) * 100)}%` : '已保留上传进度'
  return `${target.name} ${progress}，可继续未完成的上传`
})

// --- 核心业务逻辑 ---

const handleDragEnter = () => {
  dragDepth += 1
  isDragOver.value = true
}

// 拖过子元素也会触发 dragleave，用进出计数避免提示文案反复闪烁。
const handleDragLeave = () => {
  dragDepth = Math.max(0, dragDepth - 1)
  if (!dragDepth) isDragOver.value = false
}

const resetDragState = () => {
  dragDepth = 0
  isDragOver.value = false
}

/** 统一入口：登录、格式、体积三道校验全部在进入上传态之前完成。 */
const startUpload = async (selectedFile, extraFileCount = 0) => {
  if (uploading.value) {
    showMsg('已有上传任务在进行，请等当前任务结束', true)
    return
  }
  if (!currentUser.value) {
    showMsg('⚠️ 权限受限：请先登录系统', true)
    openAuthModal()
    return
  }
  if (!selectedFile) return
  if (!isSupportedVideo(selectedFile)) {
    showMsg(`⚠️ ${selectedFile.name} 不是受支持的视频格式`, true)
    return
  }
  const invalid = validateVideoFile(selectedFile)
  if (invalid) {
    showMsg(`⚠️ ${invalid}`, true)
    return
  }
  if (extraFileCount > 0) {
    showMsg(`一次只处理一个视频，已选择 ${selectedFile.name}，其余 ${extraFileCount} 个已忽略`)
  }
  file.value = selectedFile
  videoUrl.value = ''
  await uploadFile()
}

const handleFileChange = async (e) => {
  const selected = e.target.files
  await startUpload(selected?.[0], Math.max(0, (selected?.length || 0) - 1))
  e.target.value = ''
}

const handleDrop = async (e) => {
  resetDragState()
  const dropped = e.dataTransfer?.files
  if (!dropped?.length) return
  await startUpload(dropped[0], dropped.length - 1)
}

const buildUploadWarning = progress => {
  if (progress.retryingCount) {
    return `网络不稳定，正在重试 ${progress.retryingCount} 个分片（第 ${progress.retryAttempt}/${progress.retryMaxAttempts} 次）`
  }
  if (progress.resumedChunks) {
    return `已续传：跳过 ${progress.resumedChunks} 个此前完成的分片`
  }
  return ''
}

const applyUploadProgress = progress => {
  lastUploadProgress = progress
  const merging = progress.phase === 'merging'
  const detail = [`${formatBytes(progress.uploadedBytes)} / ${formatBytes(progress.totalBytes)}`]
  detail.push(`分片 ${progress.completedChunks}/${progress.totalChunks}`)
  if (!merging && progress.bytesPerSecond) {
    detail.push(`${formatBytes(progress.bytesPerSecond)}/s`)
    const eta = formatDurationText(progress.etaSeconds)
    if (eta) detail.push(`剩余约 ${eta}`)
  }
  uploadProgress.value = {
    label: merging ? '分片已全部送达，正在服务端合并' : '正在安全上传',
    filename: file.value?.name || uploadProgress.value.filename,
    percent: progress.percent,
    detail: detail.join(' · '),
    warning: buildUploadWarning(progress)
  }
}

const rememberResumableUpload = target => {
  if (!target || !hasUploadProgress(target)) {
    resumableFile.value = null
    return
  }
  resumableFile.value = target
  resumableChunks.value = {
    done: lastUploadProgress.completedChunks || 0,
    total: lastUploadProgress.totalChunks || 0
  }
}

const uploadFile = async () => {
  const target = file.value
  if (!target) return
  if (DEMO_MODE) {
    showMsg('演示模式：已模拟完成分片上传')
    return
  }

  const controller = new AbortController()
  uploadAbort.value = controller
  uploading.value = true
  resumableFile.value = null
  lastUploadProgress = {}
  const uploadUserId = currentUser.value?.id
  uploadProgress.value = {
    label: hasUploadProgress(target) ? '正在核对已上传分片' : '准备分片上传',
    filename: target.name,
    percent: 0,
    detail: `0 B / ${formatBytes(target.size)}`,
    warning: ''
  }

  try {
    const uploadedMedia = await uploadVideoInChunks(target, applyUploadProgress, controller.signal)
    if (currentUser.value?.id !== uploadUserId) return
    resumableFile.value = null
    showMsg(`✅ ${target.name} 上传完成`)
    await fetchList({ notify: true })
    openAgent(uploadedMedia)
  } catch (error) {
    if (currentUser.value?.id !== uploadUserId) return
    rememberResumableUpload(target)
    if (error?.aborted) {
      showMsg('上传已取消，进度已保留，可点“继续上传”接着传')
      return
    }
    console.error(error)
    showMsg(
      resumableFile.value
        ? `❌ 上传中断：${error.message}（进度已保留，可继续上传）`
        : `❌ 上传失败：${error.message}`,
      true
    )
  } finally {
    uploading.value = false
    uploadAbort.value = null
    file.value = null
  }
}

const cancelUpload = () => {
  if (!uploadAbort.value) return
  uploadProgress.value = { ...uploadProgress.value, label: '正在取消上传', warning: '' }
  uploadAbort.value.abort()
}

const resumeUpload = async () => {
  const target = resumableFile.value
  if (!target || uploading.value) return
  file.value = target
  await uploadFile()
}

const discardResumableUpload = () => {
  forgetUploadProgress(resumableFile.value)
  resumableFile.value = null
  resumableChunks.value = { done: 0, total: 0 }
  showMsg('已清除保留的上传进度，下次将从头开始')
}

const handleUrlUpload = async () => {
  const normalizedUrl = videoUrl.value.trim()
  if (!normalizedUrl) return
  if (uploading.value) {
    showMsg('已有上传任务在进行，请等当前任务结束', true)
    return
  }
  if (DEMO_MODE) {
    videoUrl.value = ''
    showMsg('演示模式：已模拟完成链接解析')
    return
  }

  if (!currentUser.value) {
    showMsg('⚠️ 权限受限：请先登录系统', true)
    openAuthModal()
    return
  }

  let parsedUrl
  try {
    parsedUrl = new URL(normalizedUrl)
  } catch {
    parsedUrl = null
  }
  if (!parsedUrl || !['http:', 'https:'].includes(parsedUrl.protocol)) {
    showMsg('⚠️ 请输入合法的 http/https 链接', true)
    return
  }

  uploading.value = true
  const uploadUserId = currentUser.value?.id
  uploadProgress.value = {
    label: '正在解析视频链接',
    filename: parsedUrl.hostname,
    percent: null,
    detail: '服务端正在拉取源视频，时长取决于源站速度',
    warning: ''
  }
  messageIsError.value = false
  message.value = '正在解析链接并极速下载 (低码率模式)...'

  const formData = new FormData()
  formData.append('url', normalizedUrl)

  try {
    const res = await apiRequest('/media/upload-url', {
      method: 'POST',
      body: formData
    })
    if (!res.ok) throw new Error(await res.text())
    const uploadedMedia = await res.json()
    if (currentUser.value?.id !== uploadUserId) return

    showMsg('✅ 链接资源已入库')
    videoUrl.value = ''
    await fetchList({ notify: true })
    openAgent(uploadedMedia)
  } catch (error) {
    console.error(error)
    if (currentUser.value?.id !== uploadUserId) return
    let errMsg = error.message
    if (errMsg.includes("Unsupported URL")) errMsg = "不支持该平台链接"
    showMsg('❌ 解析失败: ' + errMsg, true)
  } finally {
    uploading.value = false
  }
}

/** 成功提示自动消失；错误提示保留到用户点掉，避免关键失败原因 4 秒后就没了。 */
const showMsg = (msg, isError = false) => {
  clearTimeout(messageTimer)
  messageTimer = null
  message.value = msg
  messageIsError.value = isError
  if (isError) return
  messageTimer = setTimeout(() => {
    if (message.value !== msg) return
    message.value = ''
    messageIsError.value = false
  }, MESSAGE_TIMEOUT_MS)
}

const dismissMessage = () => {
  if (!messageIsError.value) return
  clearTimeout(messageTimer)
  messageTimer = null
  message.value = ''
  messageIsError.value = false
}

const fetchList = async ({ notify = false } = {}) => {
  if (DEMO_MODE) return list.value
  if (!currentUser.value) {
    list.value = []
    return list.value
  }
  try {
    // 带时间戳绕开浏览器缓存，避免删除/新增之后列表还是旧的。
    const res = await apiRequest(`/media/list?_t=${Date.now()}`)
    if (res.status === 401) return null
    if (!res.ok) throw new Error('加载视频列表失败')
    list.value = await res.json()
  } catch (error) {
    console.error(error)
    if (notify) showMsg('视频资料库加载失败，请稍后刷新', true)
    return null
  }
  return list.value
}

const isSupportedVideo = selectedFile => {
  if (selectedFile.type?.startsWith('video/')) return true
  const extension = selectedFile.name?.split('.').pop()?.toLowerCase()
  return VIDEO_EXTENSIONS.has(extension)
}

const mediaStatusClass = status => ['COMPLETED', 'PROCESSING', 'FAILED'].includes(status)
  ? status.toLowerCase()
  : 'unknown'
const mediaStatusLabel = status => ({
  COMPLETED: 'READY',
  PROCESSING: 'PROCESSING',
  FAILED: 'FAILED'
})[status] || 'PENDING'

const {
  sidebar,
  goalPresets,
  analysisModes,
  traceStages,
  renderedMarkdown,
  transcribe,
  closeSidebar,
  openAgent,
  submitAgent,
  startNewAnalysis,
  showDemoResult,
  startPlanEdit,
  cancelPlanEdit,
  addPlanTask,
  removePlanTask,
  rerunWithPlan,
  submitFollowUp,
  searchEvidence,
  sendFeedback,
  retryPlayback,
  handlePlaybackError,
  resetWorkspace,
  discardMediaWorkspace,
  formatPercent
} = useAnalysisWorkspace({
  demoMode: DEMO_MODE,
  taskStreams,
  showMessage: showMsg,
  refreshMediaList: fetchList,
  findMediaItem: id => list.value.find(item => item.id === id),
  onAnswerAppended: () => scrollToLatestAnswer()
})

/** 追问的答案追加在长文末尾，主动滚过去，否则用户会以为“点了没反应”。 */
const scrollToLatestAnswer = async () => {
  await nextTick()
  const container = sidebarBody.value?.querySelector('.markdown-content')
  if (!container) return
  const headings = container.querySelectorAll('h2, h3')
  const anchor = headings.length ? headings[headings.length - 1] : container.lastElementChild
  anchor?.scrollIntoView({ behavior: 'smooth', block: 'start' })
}

const seekVideo = seconds => {
  if (!Number.isFinite(seconds)) return
  const player = videoPlayer.value
  if (!player) {
    if (sidebar.value.playbackError) {
      showMsg('原视频加载失败，无法跳转，可先点“重新加载”', true)
    } else if (sidebar.value.playbackLoading) {
      showMsg('原视频还在载入，稍等一下再点这个时间戳')
    } else {
      showMsg('这个视频暂时没有可播放的原片，无法跳转', true)
    }
    return
  }
  if (player.readyState === 0) {
    player.addEventListener('loadedmetadata', () => seekVideo(seconds), { once: true })
    return
  }
  const duration = player.duration
  const maxTime = Number.isFinite(duration) ? Math.max(0, duration - 0.1) : Number.MAX_SAFE_INTEGER
  player.currentTime = Math.min(Math.max(0, seconds), maxTime)
  player.play().catch(() => {})
  player.scrollIntoView({ behavior: 'smooth', block: 'nearest' })
}

const seekEvidence = event => {
  const link = event.target.closest('a[href^="#video-t="]')
  if (!link) return
  event.preventDefault()
  seekVideo(Number(link.getAttribute('href').split('=')[1]))
}

const seekToEvidence = timestampMs => seekVideo(Number(timestampMs) / 1000)
const formatEvidenceTime = timestampMs => {
  const seconds = Math.max(0, Math.floor(Number(timestampMs) / 1000))
  const hours = Math.floor(seconds / 3600)
  const minutes = Math.floor((seconds % 3600) / 60)
  const time = `${String(minutes).padStart(2, '0')}:${String(seconds % 60).padStart(2, '0')}`
  return hours ? `${String(hours).padStart(2, '0')}:${time}` : time
}

/** Clipboard API 在非 HTTPS 环境不可用，这里保留一条降级路径，避免“复制失败”变成死路。 */
const copyToClipboard = async text => {
  try {
    await navigator.clipboard.writeText(text)
    return true
  } catch {
    // 继续走下面的降级方案。
  }
  try {
    const scratch = document.createElement('textarea')
    scratch.value = text
    scratch.setAttribute('readonly', '')
    scratch.style.position = 'fixed'
    scratch.style.top = '0'
    scratch.style.opacity = '0'
    document.body.appendChild(scratch)
    scratch.select()
    const copied = document.execCommand('copy')
    document.body.removeChild(scratch)
    return copied
  } catch {
    return false
  }
}

const copyResult = async () => {
  const content = sidebar.value.content
  if (!content) {
    showMsg('还没有可复制的内容', true)
    return
  }
  const label = sidebar.value.type === 'ai' ? '分析结果' : '转写全文'
  if (await copyToClipboard(content)) showMsg(`${label}已复制`)
  else showMsg('复制失败，请手动选中内容后复制', true)
}

const resultFileBaseName = () => {
  const title = sidebar.value.title || ''
  const raw = title.split(' · ').slice(1).join(' · ') || title
  const cleaned = raw.replace(/\.[^/.]+$/, '').replace(/[\\/:*?"<>|]/g, '_').trim()
  return cleaned || (sidebar.value.type === 'ai' ? 'analysis' : 'transcript')
}

const downloadResult = () => {
  const content = sidebar.value.content
  if (!content) {
    showMsg('还没有可导出的内容', true)
    return
  }
  const isMarkdown = sidebar.value.type === 'ai'
  const blob = new Blob([content], {
    type: isMarkdown ? 'text/markdown;charset=utf-8' : 'text/plain;charset=utf-8'
  })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.href = url
  link.download = `${resultFileBaseName()}.${isMarkdown ? 'md' : 'txt'}`
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
  // 立刻 revoke 在部分浏览器会导致下载拿不到内容，延后一拍更稳。
  setTimeout(() => URL.revokeObjectURL(url), 0)
  showMsg(`已导出 ${link.download}`)
}

const deleteItem = async (item) => {
  if (DEMO_MODE) {
    list.value = list.value.filter(i => i.id !== item.id)
    discardMediaWorkspace(item.id)
    showMsg('演示任务已移除')
    return
  }
  if (deletingId.value) return
  const runningTask = activeTaskOf(item.id)
  const warning = runningTask
    ? '\n\n注意：该视频还有任务正在后台执行，删除后这次的结果会丢失。'
    : ''
  if (!confirm(`确认要永久删除 "${item.filename}" 吗？${warning}`)) return
  deletingId.value = item.id
  try {
    const res = await apiRequest(`/media/delete?id=${item.id}`, { method: 'DELETE' })
    const text = await res.text()
    if (res.ok) {
      showMsg(`已删除 ${item.filename}`)
      list.value = list.value.filter(i => i.id !== item.id)
      discardMediaWorkspace(item.id)
    } else {
      showMsg('❌ ' + text, true)
    }
  } catch (e) {
    showMsg('❌ 删除请求失败', true)
  } finally {
    deletingId.value = null
  }
}

const formatTime = (timeStr) => {
  if (!timeStr) return '--'
  const date = new Date(timeStr)
  if (Number.isNaN(date.getTime())) return '--'
  return `${date.getMonth() + 1}/${date.getDate()} ${String(date.getHours()).padStart(2, '0')}:${String(date.getMinutes()).padStart(2, '0')}`
}

const downloadAudio = async (item) => {
  if (DEMO_MODE) {
    showMsg(`演示模式：${item.filename} 音频已准备`)
    return
  }
  let fileName = item.filename || 'audio.mp3';
  fileName = fileName.replace(/\.[^/.]+$/, "") + ".mp3";
  try {
    showMsg('正在转码并下载...')
    const res = await apiRequest(`/analysis/download?id=${item.id}`)
    // 失败时后端返回的是 JSON 信封，api.js 会把 message 解包给 text()，
    // 这里读出来向上抛，避免把“视频不存在 / 无权访问 / 转码失败”统一显示成同一句话。
    if (!res.ok) throw new Error((await res.text()) || '请稍后重试')
    const blob = await res.blob()
    const downloadUrl = window.URL.createObjectURL(blob)
    const link = document.createElement('a')
    link.href = downloadUrl
    link.download = fileName
    document.body.appendChild(link)
    link.click()
    document.body.removeChild(link)
    window.URL.revokeObjectURL(downloadUrl)
    showMsg('✅ 下载完成')
  } catch (e) {
    showMsg('音频下载失败：' + (e?.message || '请稍后重试'), true)
  }
}

const restoreFocus = element => {
  if (element?.isConnected && typeof element.focus === 'function') element.focus()
}

const openAuthModal = () => {
  if (showAuthModal.value) return
  focusBeforeAuth = document.activeElement
  showAuthModal.value = true
  authMessage.value = ''
  authForm.value = { username: '', password: '', nickname: '' }
}
const closeAuthModal = () => {
  showAuthModal.value = false
  restoreFocus(focusBeforeAuth)
  focusBeforeAuth = null
}
const closeActiveOverlay = () => {
  if (showAuthModal.value) closeAuthModal()
  else if (sidebar.value.visible) closeSidebar()
}
const handleKeydown = event => {
  if (event.key === 'Escape') closeActiveOverlay()
}

/** 弹窗内循环 Tab，键盘用户不会一路跳到被遮住的背景里。 */
const trapAuthFocus = event => {
  if (event.key !== 'Tab' || !authPanel.value) return
  const focusable = [...authPanel.value.querySelectorAll('button, input, [tabindex]:not([tabindex="-1"])')]
    .filter(element => !element.disabled && element.offsetParent !== null)
  if (!focusable.length) return
  const first = focusable[0]
  const last = focusable[focusable.length - 1]
  const active = document.activeElement
  if (event.shiftKey && (active === first || !authPanel.value.contains(active))) {
    event.preventDefault()
    last.focus()
  } else if (!event.shiftKey && active === last) {
    event.preventDefault()
    first.focus()
  }
}

const switchAuthMode = ({ keepMessage = false } = {}) => {
  authMode.value = authMode.value === 'login' ? 'register' : 'login'
  if (!keepMessage) authMessage.value = ''
}
const handleAuth = async () => {
  if (!authForm.value.username || !authForm.value.password) {
    authMessage.value = '请输入完整的账号和密码'
    authError.value = true
    return
  }
  authLoading.value = true
  authMessage.value = ''
  const endpoint = authMode.value === 'login' ? '/user/login' : '/user/register'
  try {
    const res = await apiRequest(endpoint, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(authForm.value)
    })
    if (!res.ok) {
      authMessage.value = (await res.text()) || `请求失败（HTTP ${res.status}）`
      authError.value = true
      return
    }
    const data = await res.json().catch(() => null)
    if (!data?.userInfo) {
      authMessage.value = '服务端返回异常，请稍后重试'
      authError.value = true
      return
    }
    if (authMode.value === 'login') {
      currentUser.value = data.userInfo
      localStorage.setItem('user', JSON.stringify(data.userInfo))
      setAuthToken(data.token)
      closeAuthModal()
      showMsg(`欢迎回来，${data.userInfo.nickname}`)
      fetchList({ notify: true })
    } else {
      authMessage.value = '注册成功，账号密码已保留，直接点“立即登录”即可'
      authError.value = false
      setTimeout(() => switchAuthMode({ keepMessage: true }), 900)
    }
  } catch (e) {
    console.error(e)
    authMessage.value = e?.message || '网络连接错误'
    authError.value = true
  } finally {
    authLoading.value = false
  }
}
/** 退出与登录失效走同一套清理，避免两处漏掉不同的字段。 */
const resetSessionState = () => {
  uploadAbort.value?.abort()
  uploadAbort.value = null
  taskStreams.stopAll()
  resetWorkspace()
  currentUser.value = null
  list.value = []
  searchQuery.value = ''
  videoUrl.value = ''
  file.value = null
  resumableFile.value = null
  resumableChunks.value = { done: 0, total: 0 }
  uploading.value = false
  localStorage.removeItem('user')
}

const logout = () => {
  if (hasAuthToken()) {
    apiRequest('/user/logout', { method: 'POST' }).catch(() => {})
  }
  resetSessionState()
  clearAuthToken()
  showMsg('已退出系统')
}

const handleAuthExpired = () => {
  resetSessionState()
  showMsg('登录状态已失效，请重新登录', true)
  openAuthModal()
}

const handleOnline = () => {
  isOffline.value = false
  showMsg('网络已恢复，正在同步最新状态')
  if (currentUser.value) fetchList()
}

const handleOffline = () => {
  isOffline.value = true
  showMsg('网络已断开：上传会自动重试，后台任务会在恢复后继续', true)
}

// 上传中误关标签页会白丢已传分片，这里让浏览器先问一句。
const handleBeforeUnload = event => {
  if (!uploading.value) return
  event.preventDefault()
  event.returnValue = ''
}

// 打开弹层时挂上标记类，锁背景滚动的规则只在窄屏生效（见样式里的说明）。
const overlayOpen = computed(() => sidebar.value.visible || showAuthModal.value)
watch(overlayOpen, open => {
  document.body.classList.toggle('overlay-open', open)
})

watch(() => sidebar.value.visible, async visible => {
  if (visible) {
    focusBeforeSidebar = document.activeElement
    await nextTick()
    sidebarPanel.value?.focus()
    return
  }
  restoreFocus(focusBeforeSidebar)
  focusBeforeSidebar = null
})

// 长任务给一个时间锚点，用户才不会怀疑是不是卡死了。
watch(() => sidebar.value.loading, loading => {
  clearInterval(elapsedTimer)
  elapsedTimer = null
  elapsedSeconds.value = 0
  if (!loading) return
  const startedAt = Date.now()
  elapsedTimer = setInterval(() => {
    elapsedSeconds.value = Math.floor((Date.now() - startedAt) / 1000)
  }, 1000)
})

onMounted(() => {
  window.addEventListener('auth-expired', handleAuthExpired)
  window.addEventListener('keydown', handleKeydown)
  window.addEventListener('online', handleOnline)
  window.addEventListener('offline', handleOffline)
  window.addEventListener('beforeunload', handleBeforeUnload)
  if (DEMO_MODE) {
    currentUser.value = { id: 1, nickname: 'Agent Demo' }
    list.value = [DEMO_ITEM]
    openAgent(DEMO_ITEM)
    showDemoResult()
    return
  }
  const savedUser = localStorage.getItem('user')
  if (savedUser && hasAuthToken()) {
    try {
      currentUser.value = JSON.parse(savedUser)
    } catch(e) {}
  }
  fetchList({ notify: Boolean(currentUser.value) })
})
onUnmounted(() => {
  window.removeEventListener('auth-expired', handleAuthExpired)
  window.removeEventListener('keydown', handleKeydown)
  window.removeEventListener('online', handleOnline)
  window.removeEventListener('offline', handleOffline)
  window.removeEventListener('beforeunload', handleBeforeUnload)
  clearTimeout(messageTimer)
  clearInterval(elapsedTimer)
  uploadAbort.value?.abort()
  document.body.classList.remove('overlay-open')
  taskStreams.stopAll()
})
</script>

<style>
:root {
  --bg-deep: #0e1013;
  --bg-card: #181a1f;
  --bg-elevated: #1d1f26;
  --bg-hover: #23262f;
  --accent-primary: #6366f1;
  --accent-primary-soft: rgba(99, 102, 241, 0.1);
  --accent-cyan: #06b6d4;
  --accent-cyan-soft: rgba(6, 182, 212, 0.1);
  --text-main: #e6e6e6;
  --text-sub: #9ca3af;
  --text-muted: #6b7280;
  --text-inverse: #ffffff;
  --border-subtle: #262830;
  --border-strong: #3a3d47;
  --success: #10b981;
  --success-soft: rgba(16, 185, 129, 0.1);
  --error: #ef4444;
  --error-soft: rgba(239, 68, 68, 0.1);
  --shadow-card: 0 1px 2px rgba(0, 0, 0, 0.3), 0 1px 3px rgba(0, 0, 0, 0.2);
  --shadow-hover: 0 8px 24px rgba(0, 0, 0, 0.4), 0 2px 6px rgba(0, 0, 0, 0.2);
  --shadow-sidebar: -1px 0 0 var(--border-subtle), 0 0 60px rgba(0, 0, 0, 0.5);
  --radius-sm: 6px;
  --radius-md: 10px;
  --radius-lg: 14px;
  --font-sans: 'Inter', 'Noto Sans SC', -apple-system, BlinkMacSystemFont, sans-serif;
  --font-mono: 'JetBrains Mono', 'Cascadia Code', 'Fira Code', monospace;
}

* { box-sizing: border-box; margin: 0; padding: 0; }

html, body, #app {
  margin: 0 !important; padding: 0 !important; width: 100vw !important;
  max-width: 100vw !important; min-height: 100vh !important;
  overflow-x: hidden; background-color: var(--bg-deep);
}

.app-stage { position: relative; z-index: 1; width: 100%; min-height: 100vh; color: var(--text-main); font-family: var(--font-sans); -webkit-font-smoothing: antialiased; }

.ambient-noise { display: none; }
.ambient-glow {
  position: fixed; top: -30%; left: 10%; width: 50vw; height: 50vw;
  background: radial-gradient(circle, rgba(99, 102, 241, 0.06) 0%, rgba(14, 16, 19, 0) 65%);
  pointer-events: none; z-index: -1;
}

.navbar {
  position: sticky; top: 0; z-index: 100; width: 100%; padding: 0;
  background: rgba(14, 16, 19, 0.75); backdrop-filter: blur(16px) saturate(1.5);
  border-bottom: 1px solid var(--border-subtle);
}
.nav-content { max-width: 1400px; margin: 0 auto; padding: 0 2rem; height: 56px; display: flex; justify-content: space-between; align-items: center; }
.brand { display: flex; align-items: center; gap: 10px; }
.brand-mark {
  width: 30px; height: 30px; border-radius: 7px;
  background: linear-gradient(135deg, var(--accent-primary), var(--accent-cyan));
  display: grid; place-items: center;
  font-family: var(--font-sans); font-weight: 700; font-size: 0.75rem;
  color: #fff; letter-spacing: -0.5px;
}
.brand-name { font-size: 1rem; font-weight: 600; letter-spacing: -0.01em; color: var(--text-main); }
.beta-badge {
  font-size: 0.65rem; font-weight: 600; letter-spacing: 0.04em; text-transform: uppercase;
  color: var(--accent-cyan); background: var(--accent-cyan-soft);
  padding: 2px 6px; border-radius: 4px;
}

.nav-controls { display: flex; align-items: center; gap: 14px; }
.auth-btn {
  background: transparent; border: 1px solid var(--border-strong); color: var(--text-main);
  padding: 7px 16px; border-radius: var(--radius-sm); font-family: var(--font-sans);
  font-weight: 500; cursor: pointer; display: flex; align-items: center; gap: 8px;
  transition: all 0.15s ease; font-size: 0.85rem;
}
.auth-btn:hover { background: var(--bg-hover); border-color: var(--border-strong); }
.user-profile { display: flex; align-items: center; gap: 10px; font-family: var(--font-sans); font-size: 0.875rem; color: var(--text-main); }
.user-name { color: var(--text-main); font-weight: 500; }
.logout-btn { background: none; border: none; color: var(--text-muted); cursor: pointer; padding: 4px; display: flex; align-items: center; transition: color 0.15s; }
.logout-btn:hover { color: var(--error); }

.status-pill {
  display: flex; align-items: center; gap: 8px;
  background: var(--bg-card); padding: 5px 11px; border-radius: 999px;
  border: 1px solid var(--border-subtle); font-size: 0.75rem; color: var(--text-sub);
}
.status-dot { width: 6px; height: 6px; background: var(--success); border-radius: 50%; }
.status-pill.is-active .status-dot { animation: pulse-dot 2s infinite; }

.main-container { max-width: 1200px; margin: 0 auto; padding: 5rem 2rem 3rem; }
.hero-section { text-align: center; margin-bottom: 4rem; opacity: 0; animation: fadeUp 0.6s forwards; }
.slogan-main {
  font-size: clamp(1.75rem, 4vw, 2.75rem); font-weight: 700; line-height: 1.25;
  letter-spacing: -0.02em; color: var(--text-main); margin-bottom: 0.8rem;
}
.slogan-main br { display: block; content: ""; }
.slogan-sub { font-size: 0.95rem; color: var(--text-sub); letter-spacing: 0; margin-bottom: 2.5rem; line-height: 1.6; }

.upload-wrapper { max-width: 800px; margin: 0 auto; opacity: 0; animation: fadeUp 0.6s 0.15s forwards; }
.upload-magnet {
  position: relative; height: 280px;
  background: var(--bg-card); border-radius: var(--radius-lg);
  box-shadow: var(--shadow-card); border: 1px solid var(--border-subtle);
  overflow: hidden; transition: border-color 0.2s, box-shadow 0.2s;
}
.upload-magnet:hover { border-color: var(--border-strong); box-shadow: var(--shadow-hover); }

.split-container { display: flex; height: 100%; width: 100%; position: relative; overflow: hidden; }

.skew-pane {
  flex: 1; height: 100%; position: relative; cursor: pointer;
  background: var(--bg-card); transition: background 0.2s;
  display: flex; align-items: center; justify-content: center;
  border-right: 1px solid var(--border-subtle);
}
.skew-pane:last-child { border-right: none; }
.skew-pane:hover { background: var(--bg-elevated); }

.pane-content { display: flex; flex-direction: column; align-items: center; gap: 0; z-index: 2; padding: 2rem; }

.split-gap { display: none; }

.magnet-icon { color: var(--accent-primary); margin-bottom: 1rem; }
.magnet-title {
  font-size: 1.05rem; font-weight: 600; letter-spacing: -0.01em;
  margin-bottom: 6px; color: var(--text-main);
}
.magnet-desc { font-size: 0.8rem; color: var(--text-sub); line-height: 1.5; }

.url-input-box {
  display: flex; align-items: center; margin-top: 14px;
  background: var(--bg-deep); border: 1px solid var(--border-subtle); border-radius: var(--radius-sm);
  transition: border-color 0.15s, background 0.15s;
}
.url-input-box:focus-within { border-color: var(--accent-primary); background: var(--bg-card); }
.skew-pane:hover .url-input-box { border-color: var(--border-strong); }
.skew-pane:hover .url-input-box:focus-within { border-color: var(--accent-primary); }
.url-input-box input {
  background: transparent; border: none; outline: none; color: var(--text-main);
  font-family: var(--font-sans); padding: 8px 12px; width: 200px; font-size: 0.85rem;
}
.url-input-box input::placeholder { color: var(--text-muted); }
.url-go-btn {
  background: transparent; border: none; color: var(--text-muted); cursor: pointer;
  padding: 6px 10px 6px 6px; transition: color 0.15s; display: flex;
}
.url-go-btn:hover { color: var(--accent-primary); }

.magnet-content.busy {
  height: 100%; width: 100%; display: flex; flex-direction: column; align-items: center; justify-content: center;
  background: var(--bg-card); position: relative; z-index: 50;
}
.busy-text { margin-top: 16px; color: var(--text-main); font-size: 0.85rem; font-weight: 500; }
.busy-file { max-width: 80%; margin-top: 6px; color: var(--text-sub); font-size: 0.8rem; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.upload-progress { width: min(320px, 72%); height: 3px; margin-top: 18px; background: var(--border-subtle); border-radius: 2px; overflow: hidden; }
.upload-progress span { display: block; height: 100%; background: var(--accent-primary); transition: width 0.25s ease; border-radius: 2px; }
.busy-stat { max-width: 82%; margin-top: 10px; color: var(--text-sub); font-family: var(--font-mono); font-size: 0.75rem; text-align: center; }
.busy-warning { max-width: 82%; margin-top: 8px; color: var(--error); font-family: var(--font-mono); font-size: 0.75rem; text-align: center; }
.busy-actions { margin-top: 14px; }
.busy-actions button { border: 1px solid var(--border-subtle); border-radius: var(--radius-sm); background: transparent; color: var(--text-sub); padding: 6px 14px; font-size: 0.8rem; cursor: pointer; transition: all 0.15s; }
.busy-actions button:hover { border-color: var(--error); color: var(--error); }

.upload-resume {
  margin-top: 14px; display: flex; flex-wrap: wrap; align-items: center; justify-content: center; gap: 10px;
  padding: 12px 16px; background: var(--bg-card); border: 1px solid var(--border-subtle); border-radius: var(--radius-sm);
  color: var(--text-sub); font-size: 0.82rem; text-align: left;
}
.upload-resume button { border: 1px solid var(--border-subtle); border-radius: var(--radius-sm); background: transparent; color: var(--accent-primary); padding: 5px 12px; cursor: pointer; font-size: 0.8rem; transition: all 0.15s; }
.upload-resume button:hover { border-color: var(--accent-primary); background: var(--accent-primary-soft); }

.notification-bar {
  margin-top: 1.5rem; display: inline-block;
  background: var(--accent-primary); color: #fff;
  padding: 9px 20px; font-weight: 500; border-radius: var(--radius-sm);
  font-size: 0.85rem; transition: opacity 0.15s;
}
.notification-bar.error { background: var(--error); color: #fff; cursor: pointer; }

.quantum-loader {
  width: 36px; height: 36px;
  border: 2px solid var(--border-subtle); border-top-color: var(--accent-primary);
  border-radius: 50%; animation: spin 0.7s linear infinite;
}
.quantum-loader.small { width: 22px; height: 22px; margin: 0 auto; border-width: 2px; }

.workspace-section { opacity: 0; animation: fadeUp 0.6s 0.3s forwards; }
.section-header {
  display: flex; align-items: center; justify-content: space-between; gap: 20px;
  margin-bottom: 1.5rem;
}
.library-title { display: flex; align-items: center; gap: 12px; }
.section-header h3 { font-size: 1.2rem; font-weight: 600; letter-spacing: -0.01em; }
.count-chip { background: var(--bg-card); border: 1px solid var(--border-subtle); padding: 3px 9px; border-radius: 999px; font-size: 0.72rem; font-weight: 500; color: var(--text-sub); }
.library-search {
  width: min(320px, 42vw); display: flex; align-items: center; gap: 8px;
  padding: 7px 12px; border: 1px solid var(--border-subtle); background: var(--bg-card);
  border-radius: var(--radius-sm); color: var(--text-muted); transition: border-color 0.15s;
}
.library-search:focus-within { border-color: var(--accent-primary); color: var(--accent-primary); }
.library-search input { width: 100%; border: 0; outline: 0; background: transparent; color: var(--text-main); font: inherit; font-size: 0.85rem; }
.library-empty { padding: 48px 20px; text-align: center; color: var(--text-muted); border: 1px dashed var(--border-subtle); border-radius: var(--radius-md); font-size: 0.88rem; }
.library-empty button { margin-top: 10px; border: 0; background: transparent; color: var(--accent-primary); cursor: pointer; font-size: 0.85rem; }

.card-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(300px, 1fr)); gap: 16px; }
.project-card {
  background: var(--bg-card); border-radius: var(--radius-md);
  box-shadow: var(--shadow-card); border: 1px solid var(--border-subtle);
  overflow: hidden; transition: transform 0.15s, border-color 0.15s, box-shadow 0.15s;
  position: relative;
}
.project-card:hover { transform: translateY(-2px); border-color: var(--border-strong); box-shadow: var(--shadow-hover); }
.card-meta { display: flex; gap: 1rem; padding: 1.1rem 1.2rem; align-items: center; border-bottom: 1px solid var(--border-subtle); }
.meta-icon {
  width: 44px; height: 44px; background: var(--bg-deep);
  border: 1px solid var(--border-subtle); border-radius: var(--radius-sm);
  display: flex; align-items: center; justify-content: center; color: var(--accent-primary);
}
.filename-mask { font-size: 0.95rem; font-weight: 600; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; max-width: 180px; color: var(--text-main); }
.meta-tags { display: flex; gap: 10px; font-size: 0.78rem; margin-top: 4px; }
.time-tag { color: var(--text-muted); font-family: var(--font-mono); }
.status-indicator { font-weight: 500; padding: 1px 8px; border-radius: 4px; font-size: 0.72rem; }
.status-indicator.completed { color: var(--success); background: var(--success-soft); }
.status-indicator.processing { color: var(--accent-primary); background: var(--accent-primary-soft); animation: pulse-dot 2s infinite; }
.status-indicator.failed { color: var(--error); background: var(--error-soft); }
.status-indicator.unknown { color: var(--text-muted); background: var(--bg-hover); }

.action-dock { display: grid; grid-template-columns: 1fr 1fr 1.5fr; gap: 10px; padding: 10px; }
.dock-item {
  position: relative; border: 1px solid var(--border-subtle); background: var(--bg-card);
  border-radius: var(--radius-sm); padding: 14px 10px;
  display: flex; align-items: center; justify-content: center; gap: 8px;
  cursor: pointer; transition: all 0.15s; color: var(--text-sub); font-size: 0.82rem; font-weight: 500;
  overflow: hidden;
}
.dock-item:hover:not(:disabled) { color: var(--text-main); border-color: var(--border-strong); background: var(--bg-elevated); }
.dock-item:disabled { opacity: 0.35; cursor: not-allowed; }
.dock-item.ai-core { border-color: var(--accent-primary); color: var(--accent-primary); background: var(--accent-primary-soft); }
.dock-item.ai-core .label-group { display: flex; flex-direction: column; align-items: flex-start; gap: 2px; z-index: 1; }
.dock-item.ai-core .item-sub { font-size: 0.7rem; color: var(--accent-primary); opacity: 0.8; font-weight: 400; }
.dock-item.ai-core:hover:not(:disabled) { background: var(--accent-primary); color: #fff; }
.dock-item.ai-core:hover:not(:disabled) .item-sub { color: rgba(255,255,255,0.8); }

.sidebar-backdrop { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); backdrop-filter: blur(3px); z-index: 998; }
.sidebar-panel {
  position: fixed; top: 0; right: -920px; width: 880px; max-width: calc(100vw - 24px); height: 100%;
  background: var(--bg-elevated); border-left: 1px solid var(--border-subtle);
  z-index: 999; transition: right 0.35s cubic-bezier(0.16, 1, 0.3, 1);
  display: flex; flex-direction: column; box-shadow: var(--shadow-sidebar);
}
.sidebar-panel.is-open { right: 0; }
.sidebar-header { padding: 16px 28px; border-bottom: 1px solid var(--border-subtle); display: flex; justify-content: space-between; align-items: center; }
.sidebar-title { font-size: 1.05rem; font-weight: 600; color: var(--text-main); display: flex; align-items: center; gap: 10px; letter-spacing: -0.01em; }
.icon { color: var(--accent-primary); display: flex; align-items: center; }
.close-btn { width: 32px; height: 32px; background: var(--bg-card); border: 1px solid var(--border-subtle); border-radius: var(--radius-sm); color: var(--text-sub); cursor: pointer; transition: all 0.15s; font-size: 1rem; display: grid; place-items: center; }
.close-btn:hover { color: var(--text-main); background: var(--bg-hover); border-color: var(--border-strong); }
.sidebar-body { flex: 1; overflow-y: auto; padding: 24px 28px; }
.loading-state { display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100%; color: var(--text-sub); gap: 16px; }
.markdown-content, .text-content { line-height: 1.75; color: var(--text-main); font-size: 0.9rem; }
.text-content pre { white-space: pre-wrap; font-family: var(--font-mono); background: var(--bg-deep); padding: 14px; border-radius: var(--radius-sm); border: 1px solid var(--border-subtle); color: #c4c4c4; font-size: 0.82rem; }
.text-meta { margin-bottom: 10px; color: var(--text-muted); font-family: var(--font-mono); font-size: 0.75rem; }
.markdown-content h1, .markdown-content h2, .markdown-content h3 { color: var(--text-main); margin-top: 1.3em; margin-bottom: 0.5em; font-weight: 600; letter-spacing: -0.01em; }
.markdown-content h1 { border-bottom: 1px solid var(--border-subtle); padding-bottom: 10px; font-size: 1.3rem; }
.markdown-content h2 { font-size: 1.1rem; }
.markdown-content h3 { font-size: 0.98rem; }
.markdown-content ul { padding-left: 20px; }
.markdown-content li { margin-bottom: 6px; color: var(--text-sub); }
.markdown-content strong { color: var(--accent-primary); font-weight: 600; }
.markdown-content p { margin-bottom: 0.8em; color: var(--text-sub); }
.markdown-content a[href^="#video-t="] {
  display: inline-block; padding: 1px 7px; border: 1px solid var(--accent-primary);
  color: var(--accent-primary); text-decoration: none; border-radius: 4px;
  font-family: var(--font-mono); font-size: 0.78rem; margin: 0 2px;
}
.markdown-content a[href^="#video-t="]:hover { background: var(--accent-primary); color: #fff; }
.result-actions { display: flex; justify-content: flex-end; gap: 6px; margin-bottom: 18px; }
.result-actions button {
  border: 1px solid var(--border-subtle); background: var(--bg-card); color: var(--text-sub);
  padding: 6px 10px; border-radius: var(--radius-sm); cursor: pointer; font-size: 0.8rem; transition: all 0.15s;
}
.result-actions button:hover:not(:disabled) { border-color: var(--border-strong); color: var(--text-main); }
.result-actions button:disabled { opacity: 0.4; cursor: not-allowed; }

.video-evidence { margin: -24px -28px 24px; background: var(--bg-deep); border-bottom: 1px solid var(--border-subtle); }
.video-evidence video { display: block; width: 100%; max-height: 420px; aspect-ratio: 16 / 9; background: #000; object-fit: contain; }
.video-evidence p, .video-evidence-loading { padding: 10px 28px; color: var(--text-muted); font-size: 0.78rem; }
.video-evidence-loading { min-height: 100px; display: grid; place-items: center; }
.video-evidence-error { min-height: 100px; padding: 16px 28px; display: flex; align-items: center; justify-content: center; gap: 10px; color: var(--error); font-size: 0.82rem; }
.video-evidence-error button { border: 1px solid var(--error); background: transparent; color: var(--error); padding: 5px 10px; border-radius: var(--radius-sm); cursor: pointer; font-size: 0.8rem; }

.agent-composer { display: flex; flex-direction: column; gap: 16px; }
.agent-caption { color: var(--text-sub); line-height: 1.65; font-size: 0.88rem; }
.inline-error { padding: 10px 12px; border-left: 2px solid var(--error); background: var(--error-soft); color: var(--error); line-height: 1.5; border-radius: 0 var(--radius-sm) var(--radius-sm) 0; font-size: 0.82rem; }
.agent-composer textarea, .follow-up-box textarea {
  width: 100%; min-height: 120px; resize: vertical;
  background: var(--bg-deep); color: var(--text-main);
  border: 1px solid var(--border-subtle); border-radius: var(--radius-sm);
  padding: 12px 14px; line-height: 1.6; outline: none; font-family: var(--font-sans); font-size: 0.88rem;
  transition: border-color 0.15s;
}
.agent-composer textarea:focus, .follow-up-box textarea:focus { border-color: var(--accent-primary); }
.field-counter { color: var(--text-muted); font-family: var(--font-mono); font-size: 0.72rem; text-align: right; }
.goal-presets { display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 8px; }
.goal-presets button, .feedback-row button {
  border: 1px solid var(--border-subtle); border-radius: var(--radius-sm); background: var(--bg-card);
  color: var(--text-sub); padding: 7px 10px; cursor: pointer; font-size: 0.82rem;
  transition: all 0.15s; font-family: var(--font-sans);
}
.goal-presets button { min-height: 78px; padding: 12px; text-align: left; }
.goal-presets strong, .goal-presets span { display: block; }
.goal-presets strong { margin-bottom: 5px; color: var(--text-main); font-weight: 600; font-size: 0.85rem; }
.goal-presets span { font-size: 0.73rem; line-height: 1.45; color: var(--text-muted); }
.goal-presets button:hover, .goal-presets button.active, .feedback-row button:hover, .feedback-row button.active {
  color: var(--accent-primary); border-color: var(--accent-primary); background: var(--accent-primary-soft);
}
.goal-presets button:hover strong, .goal-presets button.active strong { color: var(--accent-primary); }
.agent-run-btn {
  border: 0; border-radius: var(--radius-sm); padding: 11px 18px;
  background: var(--accent-primary); color: #fff;
  font-weight: 600; cursor: pointer; font-size: 0.88rem;
  transition: all 0.15s;
}
.agent-run-btn:hover:not(:disabled) { background: #5558e3; }
.agent-run-btn:disabled, .follow-up-box button:disabled { opacity: 0.4; cursor: not-allowed; }

.evidence-search { margin: 16px 0 20px; }
.evidence-search-form { display: grid; grid-template-columns: minmax(0, 1fr) auto; gap: 8px; }
.evidence-search-form input {
  min-width: 0; border: 1px solid var(--border-subtle); border-radius: var(--radius-sm);
  background: var(--bg-deep); color: var(--text-main); padding: 9px 12px; outline: none; font-size: 0.85rem;
  transition: border-color 0.15s;
}
.evidence-search-form input:focus { border-color: var(--accent-primary); }
.evidence-search-form button, .evidence-search-results button {
  border: 1px solid var(--border-subtle); border-radius: var(--radius-sm); background: var(--bg-card);
  color: var(--text-sub); padding: 8px 12px; cursor: pointer; font-size: 0.82rem; transition: all 0.15s;
}
.evidence-search-form button:hover, .evidence-search-results button:hover { border-color: var(--border-strong); color: var(--text-main); }
.evidence-search-form button:disabled { opacity: 0.4; cursor: not-allowed; }
.evidence-search-error { margin-top: 6px; color: var(--error); font-size: 0.8rem; }
.evidence-search-results { display: grid; gap: 6px; margin-top: 8px; }
.evidence-search-results button { display: grid; grid-template-columns: 58px 70px minmax(0, 1fr); gap: 10px; text-align: left; padding: 10px 12px; }
.evidence-search-results strong { color: var(--accent-primary); font-weight: 600; font-size: 0.8rem; }
.evidence-search-results small { color: var(--accent-cyan); overflow: hidden; text-overflow: ellipsis; white-space: nowrap; font-size: 0.72rem; }
.evidence-search-results span { overflow: hidden; text-overflow: ellipsis; white-space: nowrap; font-size: 0.8rem; }

.agent-running { display: flex; flex-direction: column; gap: 16px; }
.agent-running .loading-state { min-height: 200px; height: auto; }
.loading-state p { max-width: 34rem; text-align: center; font-size: 0.85rem; }
.stream-offline { color: var(--error); font-size: 0.78rem; }
.loading-hint { color: var(--text-muted); font-size: 0.78rem; opacity: 0.85; }

.agent-inspector { margin-top: 24px; border-top: 1px solid var(--border-subtle); padding-top: 14px; }
.agent-inspector summary { color: var(--text-sub); cursor: pointer; font-weight: 500; padding: 8px 0; font-size: 0.85rem; list-style: none; display: flex; align-items: center; gap: 6px; }
.agent-inspector summary::-webkit-details-marker { display: none; }
.agent-inspector summary:hover { color: var(--text-main); }
.agent-inspector summary::before { content: '›'; color: var(--text-muted); transition: transform 0.15s; }
.agent-inspector[open] summary::before { transform: rotate(90deg); }
.agent-inspector-content { padding-top: 12px; }
.agent-meta-block {
  margin-bottom: 14px; padding: 12px 14px; background: var(--bg-deep);
  border-left: 2px solid var(--accent-primary); border-radius: 0 var(--radius-sm) var(--radius-sm) 0;
}
.meta-label { display: block; color: var(--accent-primary); font-size: 0.72rem; font-weight: 600; margin-bottom: 8px; letter-spacing: 0.04em; text-transform: uppercase; }
.agent-meta-block ol { padding-left: 18px; color: var(--text-sub); }
.agent-meta-block li { margin: 5px 0; font-size: 0.82rem; }
.plan-editor { display: grid; gap: 8px; margin-top: 10px; }
.plan-editor-row { display: grid; grid-template-columns: minmax(0, 1fr) 34px; gap: 8px; }
.plan-editor input { min-width: 0; border: 1px solid var(--border-subtle); background: var(--bg-deep); color: var(--text-main); padding: 8px 10px; border-radius: var(--radius-sm); font-size: 0.85rem; outline: none; transition: border-color 0.15s; }
.plan-editor input:focus { border-color: var(--accent-primary); }
.plan-editor button, .plan-edit-trigger {
  border: 1px solid var(--border-subtle); background: var(--bg-card); color: var(--text-sub);
  padding: 6px 10px; cursor: pointer; border-radius: var(--radius-sm); font-size: 0.82rem; transition: all 0.15s;
}
.plan-editor button:hover, .plan-edit-trigger:hover { color: var(--text-main); border-color: var(--border-strong); }
.plan-editor-actions { display: flex; justify-content: flex-end; gap: 8px; }
.plan-edit-trigger { margin-top: 6px; font-size: 0.8rem; }

.stage-list { display: flex; flex-wrap: wrap; gap: 6px; }
.stage-list span, .quality-row span {
  border: 1px solid var(--border-subtle); border-radius: var(--radius-sm); padding: 4px 8px;
  color: var(--text-sub); font-size: 0.75rem; background: var(--bg-card);
}
.quality-row { display: flex; flex-wrap: wrap; gap: 6px; }

.follow-up-box { display: grid; grid-template-columns: 1fr auto; gap: 10px; margin-top: 20px; }
.follow-up-box textarea { min-height: 72px; }
.follow-up-box button {
  align-self: stretch; min-width: 72px; border: 0; border-radius: var(--radius-sm);
  background: var(--accent-primary); color: #fff; cursor: pointer;
  font-weight: 600; font-size: 0.85rem; transition: all 0.15s;
}
.follow-up-box button:hover:not(:disabled) { background: #5558e3; }

.feedback-row { display: flex; align-items: center; gap: 8px; margin-top: 16px; color: var(--text-sub); font-size: 0.82rem; }

.auth-backdrop { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.6); backdrop-filter: blur(4px); z-index: 2000; display: flex; justify-content: center; align-items: center; }
.auth-panel {
  width: 400px; max-width: 90vw; background: var(--bg-elevated);
  border: 1px solid var(--border-subtle); border-radius: var(--radius-lg);
  box-shadow: 0 20px 60px rgba(0,0,0,0.5);
  display: flex; flex-direction: column; animation: fadeUp 0.3s forwards; overflow: hidden;
}
.auth-header { padding: 18px 22px; border-bottom: 1px solid var(--border-subtle); background: var(--bg-card); display: flex; justify-content: space-between; align-items: center; }
.auth-title { font-size: 0.95rem; color: var(--text-main); font-weight: 600; letter-spacing: -0.01em; }
.auth-body { padding: 26px 22px; }
.input-group { margin-bottom: 18px; }
.input-group label { display: block; color: var(--text-sub); font-size: 0.72rem; margin-bottom: 7px; letter-spacing: 0.04em; text-transform: uppercase; font-weight: 500; }
.input-group input {
  width: 100%; background: var(--bg-deep); border: 1px solid var(--border-subtle);
  padding: 10px 12px; color: var(--text-main); font-family: var(--font-sans); font-size: 0.88rem;
  outline: none; transition: border-color 0.15s; border-radius: var(--radius-sm);
}
.input-group input:focus { border-color: var(--accent-primary); }
.cyber-btn {
  width: 100%; background: var(--accent-primary); color: #fff;
  border: none; padding: 11px; font-weight: 600; font-family: var(--font-sans);
  cursor: pointer; transition: background 0.15s; border-radius: var(--radius-sm);
  margin-bottom: 16px; font-size: 0.88rem;
}
.cyber-btn:hover:not(:disabled) { background: #5558e3; }
.cyber-btn:disabled { opacity: 0.5; cursor: not-allowed; }
.auth-toggle { text-align: center; font-size: 0.82rem; color: var(--text-muted); }
.toggle-link { background: none; border: none; color: var(--accent-primary); cursor: pointer; font-weight: 500; margin-left: 4px; }
.toggle-link:hover { color: #818cf8; }
.auth-msg { margin-top: 12px; text-align: center; font-size: 0.8rem; color: var(--text-sub); }
.auth-msg.error { color: var(--error); }

.delete-btn {
  position: absolute; top: 8px; right: 8px; background: var(--bg-card);
  border: 1px solid var(--border-subtle); border-radius: var(--radius-sm);
  width: 30px; height: 30px; color: var(--text-muted); cursor: pointer;
  opacity: 0; transition: all 0.15s ease; z-index: 10;
  display: grid; place-items: center;
}
.project-card:hover .delete-btn, .delete-btn:focus-visible { opacity: 1; }
.delete-btn:hover:not(:disabled) { color: var(--error); border-color: var(--error); background: var(--error-soft); }
.delete-btn:disabled { cursor: progress; opacity: 0.5; }

.url-input-box input:disabled, .url-go-btn:disabled { opacity: 0.5; cursor: not-allowed; }

@media (max-width: 720px) {
  .navbar { padding: 0; }
  .nav-content { padding: 0 1rem; }
  .brand-mark { width: 26px; height: 26px; font-size: 0.65rem; }
  .brand-name { font-size: 0.9rem; }
  .status-pill { display: none; }
  .auth-btn { padding: 6px 10px; font-size: 0.8rem; }
  .main-container { padding: 2.5rem 1rem 2rem; }
  .hero-section { margin-bottom: 2.5rem; }
  .slogan-sub { margin-bottom: 1.8rem; }
  .upload-magnet { height: auto; min-height: 380px; border-radius: var(--radius-md); }
  .split-container { flex-direction: column; }
  .skew-pane, .pane-local, .pane-url { min-height: 190px; transform: none; margin: 0; padding: 0; }
  .pane-content, .skew-pane:hover .pane-content { transform: none; padding: 1.5rem; }
  .card-grid { grid-template-columns: 1fr; }
  .section-header { align-items: stretch; flex-direction: column; }
  .library-search { width: 100%; }
  .action-dock { grid-template-columns: 1fr; }
  .filename-mask { max-width: 55vw; }
  .sidebar-panel { width: 100%; max-width: 100vw; right: -100vw; }
  .sidebar-header { padding: 14px 18px; }
  .sidebar-title { font-size: 1rem; max-width: calc(100vw - 60px); overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
  .sidebar-body { padding: 18px 16px; }
  .video-evidence { margin: -18px -16px 20px; }
  .video-evidence p, .video-evidence-loading { padding-left: 16px; padding-right: 16px; }
  .goal-presets { grid-template-columns: 1fr; }
  .goal-presets button { min-height: 64px; }
  .result-actions { justify-content: stretch; }
  .result-actions button { flex: 1; padding: 8px 4px; font-size: 0.76rem; }
  .evidence-search-form { grid-template-columns: 1fr; }
  .evidence-search-results button { grid-template-columns: 56px minmax(0, 1fr); }
  .evidence-search-results small { display: none; }
  .follow-up-box { grid-template-columns: 1fr; }
  .follow-up-box button { min-height: 42px; }
  .delete-btn { opacity: 1; }
  .upload-resume { flex-direction: column; align-items: stretch; text-align: center; }
  .upload-resume button { min-height: 38px; }
  .busy-stat, .busy-warning { max-width: 92%; }
  body.overlay-open { overflow: hidden; }
}

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after { animation-duration: 0.01ms !important; animation-iteration-count: 1 !important; scroll-behavior: auto !important; transition-duration: 0.01ms !important; }
}

@keyframes spin { to { transform: rotate(360deg); } }
@keyframes fadeUp { from { opacity: 0; transform: translateY(16px); } to { opacity: 1; transform: translateY(0); } }
@keyframes pulse-dot { 0%, 100% { opacity: 1; } 50% { opacity: 0.4; } }
</style>