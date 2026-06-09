<script setup lang="ts">
import { ref, reactive, computed } from 'vue'

interface Comment {
  id: number
  author: string
  text: string
  timestamp: Date
}

interface Post {
  id: number
  author: string
  title: string
  content: string
  image: string | null
  likes: number
  liked: boolean
  comments: Comment[]
  showComments: boolean
  newComment: string
  timestamp: Date
}

interface Community {
  id: number
  name: string
  description: string
  emoji: string
  posts: Post[]
  newPostTitle: string
  newPostContent: string
  newPostImage: string | null
  showForm: boolean
  _fileInput?: HTMLInputElement | null
}

interface XpToast {
  id: number
  amount: number
  reason: string
  celebrate?: boolean
}

// ── Level thresholds ──────────────────────────────────────────────
const LEVELS = [
  { level: 1,  xp: 0,    title: 'Newcomer' },
  { level: 2,  xp: 100,  title: 'Explorer' },
  { level: 3,  xp: 250,  title: 'Contributor' },
  { level: 4,  xp: 500,  title: 'Thinker' },
  { level: 5,  xp: 900,  title: 'Innovator' },
  { level: 6,  xp: 1400, title: 'Mentor' },
  { level: 7,  xp: 2000, title: 'Visionary' },
]

const XP_POST    = 30
const XP_IMAGE   = 10
const XP_COMMENT = 10

// ── Encouraging messages shown when a user shares something ───────
const POST_CHEERS = [
  '🎉 Awesome, thanks for sharing!',
  '✨ Great post — the community will love this!',
  '🙌 Way to go! Your voice matters here.',
  '🌟 Nicely done! Keep the ideas coming.',
  '💜 Thank you for contributing!',
  '🚀 Love it — you just made the forum better!',
]
const COMMENT_CHEERS = [
  '💬 Thanks for joining the conversation!',
  '🙌 Great input!',
  '✨ Nice one — keep it up!',
  '💜 Love seeing you engage!',
]
function pickCheer(list: string[]): string {
  return list[Math.floor(Math.random() * list.length)]
}

// ── Profile ───────────────────────────────────────────────────────
const profile = reactive({ name: 'You', xp: 0, postsCount: 0, commentsCount: 0 })

const levelInfo = computed(() => {
  let current = LEVELS[0]
  let next = LEVELS[1]
  for (let i = LEVELS.length - 1; i >= 0; i--) {
    if (profile.xp >= LEVELS[i].xp) { current = LEVELS[i]; next = LEVELS[i + 1] ?? null; break }
  }
  const fromPrev = profile.xp - current.xp
  const toNext   = next ? next.xp - current.xp : 1
  return { current, next, progress: next ? Math.min(100, Math.round((fromPrev / toNext) * 100)) : 100 }
})

// ── XP toasts ─────────────────────────────────────────────────────
const toasts = ref<XpToast[]>([])
let toastId = 0

function awardXp(amount: number, reason: string, celebrate = false) {
  profile.xp += amount
  const id = toastId++
  toasts.value.push({ id, amount, reason, celebrate })
  setTimeout(() => { toasts.value = toasts.value.filter(t => t.id !== id) }, celebrate ? 3200 : 2200)
}

// ── Data ──────────────────────────────────────────────────────────
const nextId = ref(200)

const communities = reactive<Community[]>([
  {
    id: 1, name: 'General', description: 'Open discussions for everyone', emoji: '💬',
    showForm: false, newPostTitle: '', newPostContent: '', newPostImage: null,
    posts: [{
      id: 1, author: 'Athena', title: "Welcome to Athena's Playground!",
      content: "So glad you're all here. This is a place to learn, share, and grow. Feel free to share your thoughts!",
      image: null, likes: 12, liked: false,
      comments: [{ id: 11, author: 'Luna', text: 'Excited to be here!', timestamp: new Date('2026-06-07') }],
      showComments: false, newComment: '', timestamp: new Date('2026-06-01'),
    }],
  },
  {
    id: 2, name: 'Design Thinking', description: 'Creative problem-solving and design processes', emoji: '🎨',
    showForm: false, newPostTitle: '', newPostContent: '', newPostImage: null,
    posts: [{
      id: 2, author: 'Maya', title: 'Empathy in the design process',
      content: 'Empathy mapping is one of my favourite tools. Has anyone tried it and wants to share their experience?',
      image: null, likes: 8, liked: false, comments: [],
      showComments: false, newComment: '', timestamp: new Date('2026-06-05'),
    }],
  },
  {
    id: 3, name: 'Tech & Tools', description: 'Technology, tools, and digital resources', emoji: '🛠️',
    showForm: false, newPostTitle: '', newPostContent: '', newPostImage: null,
    posts: [{
      id: 3, author: 'Kai', title: 'Figma vs. other prototyping tools',
      content: "I'm a big Figma fan — are there other tools you'd recommend?",
      image: null, likes: 5, liked: false, comments: [],
      showComments: false, newComment: '', timestamp: new Date('2026-06-06'),
    }],
  },
  {
    id: 4, name: 'Inspiration', description: 'Share inspirations, resources, and ideas', emoji: '✨',
    showForm: false, newPostTitle: '', newPostContent: '', newPostImage: null, posts: [],
  },
])

// ── Actions ───────────────────────────────────────────────────────
function handleImageUpload(community: Community, event: Event) {
  const file = (event.target as HTMLInputElement).files?.[0]
  if (!file) return
  const reader = new FileReader()
  reader.onload = (e) => { community.newPostImage = e.target?.result as string }
  reader.readAsDataURL(file)
}

function clearImage(community: Community, inputRef: HTMLInputElement | null | undefined) {
  community.newPostImage = null
  if (inputRef) inputRef.value = ''
}

function addPost(community: Community) {
  if (!community.newPostTitle.trim() || !community.newPostContent.trim()) return
  const hasImage = !!community.newPostImage
  community.posts.unshift({
    id: nextId.value++, author: 'You',
    title: community.newPostTitle, content: community.newPostContent,
    image: community.newPostImage, likes: 0, liked: false,
    comments: [], showComments: false, newComment: '', timestamp: new Date(),
  })
  community.newPostTitle = ''
  community.newPostContent = ''
  community.newPostImage = null
  community.showForm = false
  profile.postsCount++
  awardXp(XP_POST + (hasImage ? XP_IMAGE : 0), pickCheer(POST_CHEERS), true)
}

function toggleLike(post: Post) {
  post.liked ? post.likes-- : post.likes++
  post.liked = !post.liked
}

function addComment(post: Post) {
  if (!post.newComment.trim()) return
  post.comments.push({
    id: nextId.value++, author: 'You',
    text: post.newComment, timestamp: new Date(),
  })
  post.newComment = ''
  profile.commentsCount++
  awardXp(XP_COMMENT, pickCheer(COMMENT_CHEERS), true)
}

function formatDate(date: Date): string {
  return date.toLocaleDateString('en-GB', { day: 'numeric', month: 'short' })
}
</script>

<template>
  <!-- XP toast notifications -->
  <Teleport to="body">
    <div class="toast-stack">
      <TransitionGroup name="toast">
        <div v-for="toast in toasts" :key="toast.id" class="xp-toast" :class="{ celebrate: toast.celebrate }">
          <span v-if="toast.celebrate" class="xp-toast-cheer">{{ toast.reason }}</span>
          <span class="xp-toast-amount">+{{ toast.amount }} XP</span>
          <span v-if="!toast.celebrate" class="xp-toast-reason">{{ toast.reason }}</span>
        </div>
      </TransitionGroup>
    </div>
  </Teleport>

  <section id="forum-header">
    <h1>Community Forum</h1>
    <p>Exchange ideas, ask questions, and share what you know.</p>

    <!-- Profile / XP card -->
    <div class="profile-card">
      <div class="profile-left">
        <div class="profile-avatar">{{ profile.name[0] }}</div>
        <div class="profile-info">
          <div class="profile-name-row">
            <span class="profile-name">{{ profile.name }}</span>
            <span class="level-badge">Lvl {{ levelInfo.current.level }}</span>
            <span class="level-title">{{ levelInfo.current.title }}</span>
          </div>
          <div class="xp-bar-wrap">
            <div class="xp-bar-track">
              <div class="xp-bar-fill" :style="{ width: levelInfo.progress + '%' }"></div>
            </div>
            <span class="xp-label">
              {{ profile.xp }} XP
              <template v-if="levelInfo.next"> · {{ levelInfo.next.xp - profile.xp }} to Lvl {{ levelInfo.next.level }}</template>
              <template v-else> · Max level!</template>
            </span>
          </div>
        </div>
      </div>
      <div class="profile-stats">
        <div class="stat">
          <span class="stat-value">{{ profile.postsCount }}</span>
          <span class="stat-label">Posts</span>
        </div>
        <div class="stat">
          <span class="stat-value">{{ profile.commentsCount }}</span>
          <span class="stat-label">Comments</span>
        </div>
      </div>
    </div>

    <!-- XP guide -->
    <div class="xp-guide">
      <span>Earn XP: <strong>+{{ XP_POST }} post</strong> · <strong>+{{ XP_IMAGE }} with image</strong> · <strong>+{{ XP_COMMENT }} comment</strong></span>
    </div>
  </section>

  <div class="ticks"></div>

  <section id="forum">
    <div v-for="community in communities" :key="community.id" class="community">
      <div class="community-head">
        <div class="community-meta">
          <span class="community-emoji">{{ community.emoji }}</span>
          <div>
            <h2>{{ community.name }}</h2>
            <p class="community-desc">{{ community.description }}</p>
          </div>
        </div>
        <button class="btn-new" @click="community.showForm = !community.showForm">
          {{ community.showForm ? '✕ Cancel' : '+ New Post' }}
        </button>
      </div>

      <form v-if="community.showForm" class="post-form" @submit.prevent="addPost(community)">
        <input
          v-model="community.newPostTitle"
          class="form-input"
          placeholder="Post title…"
          maxlength="120"
          required
        />
        <textarea
          v-model="community.newPostContent"
          class="form-textarea"
          placeholder="What's on your mind?"
          rows="3"
          required
        />

        <label class="upload-label">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <rect x="3" y="3" width="18" height="18" rx="2"/><circle cx="8.5" cy="8.5" r="1.5"/>
            <polyline points="21 15 16 10 5 21"/>
          </svg>
          Add image
          <input
            type="file"
            accept="image/*"
            class="upload-input"
            :ref="el => community._fileInput = el as HTMLInputElement"
            @change="handleImageUpload(community, $event)"
          />
        </label>

        <div v-if="community.newPostImage" class="image-preview-wrap">
          <img :src="community.newPostImage" class="image-preview" alt="Preview" />
          <button type="button" class="image-remove" @click="clearImage(community, community._fileInput)" aria-label="Remove image">✕</button>
        </div>

        <div class="form-footer">
          <span class="xp-hint">
            +{{ XP_POST + (community.newPostImage ? XP_IMAGE : 0) }} XP for this post
          </span>
          <button type="submit" class="btn-submit">Publish</button>
        </div>
      </form>

      <div v-if="community.posts.length === 0 && !community.showForm" class="empty-state">
        No posts yet — be the first!
      </div>

      <ul class="post-list">
        <li v-for="post in community.posts" :key="post.id" class="post-card">
          <div class="post-author-row">
            <span class="avatar">{{ post.author[0] }}</span>
            <span class="post-author">{{ post.author }}</span>
            <span class="post-date">{{ formatDate(post.timestamp) }}</span>
          </div>

          <h3 class="post-title">{{ post.title }}</h3>
          <p class="post-content">{{ post.content }}</p>

          <img v-if="post.image" :src="post.image" class="post-image" alt="" />

          <div class="post-actions">
            <button
              class="action-btn"
              :class="{ liked: post.liked }"
              @click="toggleLike(post)"
              :aria-label="`${post.likes} likes`"
            >
              <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"/>
              </svg>
              {{ post.likes }}
            </button>

            <button
              class="action-btn"
              @click="post.showComments = !post.showComments"
              :aria-label="`${post.comments.length} comments`"
            >
              <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/>
              </svg>
              {{ post.comments.length }} {{ post.showComments ? 'hide' : 'comments' }}
            </button>
          </div>

          <div v-if="post.showComments" class="comments-section">
            <div v-if="post.comments.length === 0" class="comments-empty">No comments yet.</div>
            <ul class="comment-list">
              <li v-for="comment in post.comments" :key="comment.id" class="comment">
                <span class="avatar avatar-sm">{{ comment.author[0] }}</span>
                <div class="comment-body">
                  <span class="comment-author">{{ comment.author }}</span>
                  <span class="comment-date">{{ formatDate(comment.timestamp) }}</span>
                  <p class="comment-text">{{ comment.text }}</p>
                </div>
              </li>
            </ul>

            <form class="comment-form" @submit.prevent="addComment(post)">
              <input
                v-model="post.newComment"
                class="form-input"
                placeholder="Write a comment…"
                maxlength="500"
              />
              <button type="submit" class="btn-submit btn-sm">Send</button>
            </form>
          </div>
        </li>
      </ul>
    </div>
  </section>

  <div class="ticks"></div>
  <section id="spacer"></section>
</template>

<style scoped>
/* ── Header ───────────────────────────────────────────────────── */
#forum-header {
  padding: 64px 32px 40px;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 24px;
}

/* ── Profile card ─────────────────────────────────────────────── */
.profile-card {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 24px;
  width: 100%;
  max-width: 600px;
  background: var(--code-bg);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 20px 24px;
  box-sizing: border-box;
  text-align: left;
}

.profile-left {
  display: flex;
  align-items: center;
  gap: 14px;
  flex: 1;
  min-width: 0;
}

.profile-avatar {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  background: var(--accent-bg);
  border: 2px solid var(--accent-border);
  color: var(--accent);
  font-size: 20px;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.profile-info {
  flex: 1;
  min-width: 0;
}

.profile-name-row {
  display: flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
  margin-bottom: 8px;
}

.profile-name {
  font-size: 15px;
  font-weight: 600;
  color: var(--text-h);
}

.level-badge {
  font-size: 11px;
  font-weight: 700;
  background: var(--accent);
  color: #fff;
  border-radius: 20px;
  padding: 2px 8px;
  letter-spacing: 0.5px;
}

.level-title {
  font-size: 12px;
  color: var(--text);
}

.xp-bar-wrap {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.xp-bar-track {
  height: 6px;
  background: var(--border);
  border-radius: 99px;
  overflow: hidden;
  width: 100%;
}

.xp-bar-fill {
  height: 100%;
  background: var(--accent);
  border-radius: 99px;
  transition: width 0.6s cubic-bezier(0.4, 0, 0.2, 1);
}

.xp-label {
  font-size: 11px;
  color: var(--text);
}

.profile-stats {
  display: flex;
  gap: 20px;
  flex-shrink: 0;
}

.stat {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
}

.stat-value {
  font-size: 20px;
  font-weight: 700;
  color: var(--text-h);
  font-family: var(--mono);
}

.stat-label {
  font-size: 11px;
  color: var(--text);
}

.xp-guide {
  font-size: 13px;
  color: var(--text);
}

.xp-guide strong {
  color: var(--accent);
}

/* ── XP toasts ────────────────────────────────────────────────── */
.toast-stack {
  position: fixed;
  top: 24px;
  right: 24px;
  z-index: 9999;
  display: flex;
  flex-direction: column;
  gap: 8px;
  pointer-events: none;
}

.xp-toast {
  display: flex;
  align-items: center;
  gap: 8px;
  background: var(--accent);
  color: #fff;
  border-radius: 10px;
  padding: 10px 16px;
  font-size: 14px;
  box-shadow: 0 4px 20px rgba(0,0,0,0.2);
}

.xp-toast-amount {
  font-weight: 700;
  font-family: var(--mono);
  font-size: 15px;
}

.xp-toast-reason {
  opacity: 0.85;
  font-size: 13px;
}

/* ── Celebration toast (shown when sharing a post/comment) ─────── */
.xp-toast.celebrate {
  flex-direction: column;
  align-items: flex-start;
  gap: 4px;
  padding: 14px 18px;
  background: linear-gradient(135deg, var(--accent), color-mix(in srgb, var(--accent) 70%, #ff4db8));
  box-shadow: 0 6px 26px rgba(0,0,0,0.28);
}

.xp-toast-cheer {
  font-weight: 600;
  font-size: 14px;
  line-height: 1.3;
}

.xp-toast.celebrate .xp-toast-amount {
  font-size: 13px;
  opacity: 0.9;
}

.toast-enter-active { transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1); }
.toast-leave-active { transition: all 0.4s ease; }
.toast-enter-from  { opacity: 0; transform: translateY(-12px) scale(0.9); }
.toast-leave-to    { opacity: 0; transform: translateX(20px); }

/* ── XP hint in form ──────────────────────────────────────────── */
.form-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.xp-hint {
  font-size: 12px;
  color: var(--accent);
  font-weight: 600;
}

/* ── Forum grid ───────────────────────────────────────────────── */
#forum {
  display: grid;
  grid-template-columns: 1fr 1fr;
  border-top: 1px solid var(--border);
}

.community {
  padding: 32px;
  border-bottom: 1px solid var(--border);
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.community:nth-child(odd) {
  border-right: 1px solid var(--border);
}

.community-head {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 16px;
}

.community-meta {
  display: flex;
  align-items: flex-start;
  gap: 12px;
}

.community-emoji {
  font-size: 28px;
  line-height: 1;
  margin-top: 2px;
}

.community-meta h2 { margin: 0 0 4px; }

.community-desc {
  font-size: 14px;
  color: var(--text);
  margin: 0;
}

/* ── Buttons ──────────────────────────────────────────────────── */
.btn-new {
  white-space: nowrap;
  background: var(--accent-bg);
  color: var(--accent);
  border: 1px solid var(--accent-border);
  border-radius: 8px;
  padding: 6px 14px;
  font-size: 14px;
  font-family: var(--sans);
  cursor: pointer;
  transition: box-shadow 0.2s;
  flex-shrink: 0;
}

.btn-new:hover { box-shadow: var(--shadow); }

.btn-submit {
  background: var(--accent);
  color: #fff;
  border: none;
  border-radius: 8px;
  padding: 8px 18px;
  font-size: 14px;
  font-family: var(--sans);
  cursor: pointer;
  transition: opacity 0.2s;
  white-space: nowrap;
}

.btn-submit:hover { opacity: 0.85; }

.btn-sm {
  padding: 6px 14px;
  font-size: 13px;
}

/* ── Post form ────────────────────────────────────────────────── */
.post-form {
  display: flex;
  flex-direction: column;
  gap: 10px;
  background: var(--code-bg);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 16px;
}

.comment-form {
  display: flex;
  gap: 10px;
  align-items: center;
  padding: 8px 0 0;
  border-top: 1px solid var(--border);
}

.form-input {
  background: var(--bg);
  border: 1px solid var(--border);
  border-radius: 8px;
  padding: 8px 12px;
  font-size: 15px;
  font-family: var(--sans);
  color: var(--text-h);
  width: 100%;
  box-sizing: border-box;
  transition: border-color 0.2s;
}

.form-input:focus {
  outline: none;
  border-color: var(--accent-border);
}

.form-textarea {
  background: var(--bg);
  border: 1px solid var(--border);
  border-radius: 8px;
  padding: 8px 12px;
  font-size: 15px;
  font-family: var(--sans);
  color: var(--text-h);
  resize: vertical;
  width: 100%;
  box-sizing: border-box;
  transition: border-color 0.2s;
}

.form-textarea:focus {
  outline: none;
  border-color: var(--accent-border);
}

/* ── Image upload ─────────────────────────────────────────────── */
.upload-label {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 13px;
  color: var(--text);
  border: 1px dashed var(--border);
  border-radius: 8px;
  padding: 8px 14px;
  cursor: pointer;
  transition: border-color 0.2s, color 0.2s;
  align-self: flex-start;
}

.upload-label:hover {
  border-color: var(--accent-border);
  color: var(--accent);
}

.upload-input { display: none; }

.image-preview-wrap {
  position: relative;
  align-self: flex-start;
}

.image-preview {
  display: block;
  max-height: 180px;
  max-width: 100%;
  border-radius: 8px;
  border: 1px solid var(--border);
  object-fit: cover;
}

.image-remove {
  position: absolute;
  top: 6px;
  right: 6px;
  background: rgba(0,0,0,0.55);
  color: #fff;
  border: none;
  border-radius: 50%;
  width: 22px;
  height: 22px;
  font-size: 11px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* ── Posts ────────────────────────────────────────────────────── */
.empty-state {
  font-size: 14px;
  color: var(--text);
  text-align: center;
  padding: 24px 0;
  border: 1px dashed var(--border);
  border-radius: 10px;
}

.post-list {
  list-style: none;
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.post-card {
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 18px;
  background: var(--bg);
  transition: box-shadow 0.2s;
}

.post-card:hover { box-shadow: var(--shadow); }

.post-author-row {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 10px;
}

.avatar {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background: var(--accent-bg);
  border: 1px solid var(--accent-border);
  color: var(--accent);
  font-size: 13px;
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.avatar-sm {
  width: 24px;
  height: 24px;
  font-size: 11px;
}

.post-author {
  font-size: 14px;
  font-weight: 600;
  color: var(--text-h);
}

.post-date {
  font-size: 12px;
  color: var(--text);
  margin-left: auto;
}

.post-title {
  font-size: 16px;
  font-weight: 600;
  color: var(--text-h);
  margin: 0 0 8px;
  font-family: var(--heading);
}

.post-content {
  font-size: 14px;
  color: var(--text);
  line-height: 1.6;
  margin: 0 0 14px;
}

.post-image {
  display: block;
  width: 100%;
  max-height: 260px;
  object-fit: cover;
  border-radius: 8px;
  border: 1px solid var(--border);
  margin-bottom: 14px;
}

.post-actions {
  display: flex;
  gap: 10px;
}

.action-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  background: none;
  border: 1px solid var(--border);
  border-radius: 20px;
  padding: 5px 12px;
  font-size: 13px;
  color: var(--text);
  font-family: var(--sans);
  cursor: pointer;
  transition: all 0.2s;
}

.action-btn:hover {
  border-color: var(--accent-border);
  color: var(--accent);
}

.action-btn.liked {
  border-color: var(--accent-border);
  color: var(--accent);
  background: var(--accent-bg);
}

.action-btn.liked svg {
  fill: var(--accent);
  stroke: var(--accent);
}

/* ── Comments ─────────────────────────────────────────────────── */
.comments-section {
  margin-top: 14px;
  border-top: 1px solid var(--border);
  padding-top: 14px;
}

.comments-empty {
  font-size: 13px;
  color: var(--text);
  padding: 8px 0;
}

.comment-list {
  list-style: none;
  margin: 0 0 12px;
  padding: 0;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.comment {
  display: flex;
  gap: 8px;
  align-items: flex-start;
}

.comment-body {
  display: flex;
  flex-wrap: wrap;
  align-items: baseline;
  gap: 4px 8px;
}

.comment-author {
  font-size: 13px;
  font-weight: 600;
  color: var(--text-h);
}

.comment-date {
  font-size: 11px;
  color: var(--text);
}

.comment-text {
  font-size: 13px;
  color: var(--text);
  line-height: 1.5;
  margin: 0;
  width: 100%;
}

/* ── Responsive ───────────────────────────────────────────────── */
@media (max-width: 1024px) {
  #forum { grid-template-columns: 1fr; }
  .community:nth-child(odd) { border-right: none; }
  .community { padding: 24px 20px; }
  #forum-header { padding: 40px 20px 24px; }
  .profile-card { flex-direction: column; align-items: flex-start; }
  .profile-stats { align-self: flex-end; }
}
</style>
