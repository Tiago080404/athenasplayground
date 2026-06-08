<script setup lang="ts">
import { ref, reactive } from 'vue'

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
  showForm: boolean
}

const nextId = ref(200)

const communities = reactive<Community[]>([
  {
    id: 1,
    name: 'General',
    description: 'Allgemeine Diskussionen für alle',
    emoji: '💬',
    showForm: false,
    newPostTitle: '',
    newPostContent: '',
    posts: [
      {
        id: 1,
        author: 'Athena',
        title: 'Willkommen in Athenas Playground!',
        content: 'Schön, dass ihr alle hier seid. Dies ist ein Ort zum Lernen, Teilen und Wachsen. Fühlt euch frei, eure Gedanken zu teilen!',
        likes: 12,
        liked: false,
        comments: [
          { id: 11, author: 'Luna', text: 'Super, freue mich schon!', timestamp: new Date('2026-06-07') },
        ],
        showComments: false,
        newComment: '',
        timestamp: new Date('2026-06-01'),
      },
    ],
  },
  {
    id: 2,
    name: 'Design Thinking',
    description: 'Kreative Problemlösung und Designprozesse',
    emoji: '🎨',
    showForm: false,
    newPostTitle: '',
    newPostContent: '',
    posts: [
      {
        id: 2,
        author: 'Maya',
        title: 'Empathie im Designprozess',
        content: 'Empathie-Mapping ist eines meiner liebsten Tools. Hat jemand Erfahrungen damit gesammelt und möchte sie teilen?',
        likes: 8,
        liked: false,
        comments: [],
        showComments: false,
        newComment: '',
        timestamp: new Date('2026-06-05'),
      },
    ],
  },
  {
    id: 3,
    name: 'Tech & Tools',
    description: 'Technologie, Werkzeuge und digitale Ressourcen',
    emoji: '🛠️',
    showForm: false,
    newPostTitle: '',
    newPostContent: '',
    posts: [
      {
        id: 3,
        author: 'Kai',
        title: 'Figma vs. andere Prototyping-Tools',
        content: 'Ich bin ein großer Figma-Fan — habt ihr andere Tools, die ihr empfehlen würdet?',
        likes: 5,
        liked: false,
        comments: [],
        showComments: false,
        newComment: '',
        timestamp: new Date('2026-06-06'),
      },
    ],
  },
  {
    id: 4,
    name: 'Inspiration',
    description: 'Teile Inspirationen, Ressourcen und Ideen',
    emoji: '✨',
    showForm: false,
    newPostTitle: '',
    newPostContent: '',
    posts: [],
  },
])

function addPost(community: Community) {
  if (!community.newPostTitle.trim() || !community.newPostContent.trim()) return
  community.posts.unshift({
    id: nextId.value++,
    author: 'Du',
    title: community.newPostTitle,
    content: community.newPostContent,
    likes: 0,
    liked: false,
    comments: [],
    showComments: false,
    newComment: '',
    timestamp: new Date(),
  })
  community.newPostTitle = ''
  community.newPostContent = ''
  community.showForm = false
}

function toggleLike(post: Post) {
  post.liked ? post.likes-- : post.likes++
  post.liked = !post.liked
}

function addComment(post: Post) {
  if (!post.newComment.trim()) return
  post.comments.push({
    id: nextId.value++,
    author: 'Du',
    text: post.newComment,
    timestamp: new Date(),
  })
  post.newComment = ''
}

function formatDate(date: Date): string {
  return date.toLocaleDateString('de-DE', { day: 'numeric', month: 'short' })
}
</script>

<template>
  <section id="forum-header">
    <h1>Community Forum</h1>
    <p>Tausch dich aus, stell Fragen und teile deine Ideen.</p>
  </section>

  <div class="ticks"></div>

  <section id="forum">
    <div
      v-for="community in communities"
      :key="community.id"
      class="community"
    >
      <div class="community-head">
        <div class="community-meta">
          <span class="community-emoji">{{ community.emoji }}</span>
          <div>
            <h2>{{ community.name }}</h2>
            <p class="community-desc">{{ community.description }}</p>
          </div>
        </div>
        <button
          class="btn-new"
          @click="community.showForm = !community.showForm"
        >
          {{ community.showForm ? '✕ Abbrechen' : '+ Neuer Post' }}
        </button>
      </div>

      <form
        v-if="community.showForm"
        class="post-form"
        @submit.prevent="addPost(community)"
      >
        <input
          v-model="community.newPostTitle"
          class="form-input"
          placeholder="Titel deines Posts…"
          maxlength="120"
          required
        />
        <textarea
          v-model="community.newPostContent"
          class="form-textarea"
          placeholder="Was möchtest du mitteilen?"
          rows="3"
          required
        />
        <button type="submit" class="btn-submit">Veröffentlichen</button>
      </form>

      <div v-if="community.posts.length === 0 && !community.showForm" class="empty-state">
        Noch keine Posts — sei die Erste!
      </div>

      <ul class="post-list">
        <li v-for="post in community.posts" :key="post.id" class="post-card">
          <div class="post-header">
            <div class="post-author-row">
              <span class="avatar">{{ post.author[0] }}</span>
              <span class="post-author">{{ post.author }}</span>
              <span class="post-date">{{ formatDate(post.timestamp) }}</span>
            </div>
          </div>

          <h3 class="post-title">{{ post.title }}</h3>
          <p class="post-content">{{ post.content }}</p>

          <div class="post-actions">
            <button
              class="action-btn"
              :class="{ liked: post.liked }"
              @click="toggleLike(post)"
              :aria-label="`${post.likes} Likes`"
            >
              <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"/>
              </svg>
              {{ post.likes }}
            </button>

            <button
              class="action-btn"
              @click="post.showComments = !post.showComments"
              :aria-label="`${post.comments.length} Kommentare`"
            >
              <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/>
              </svg>
              {{ post.comments.length }}
              {{ post.showComments ? 'ausblenden' : 'Kommentare' }}
            </button>
          </div>

          <div v-if="post.showComments" class="comments-section">
            <div v-if="post.comments.length === 0" class="comments-empty">
              Noch keine Kommentare.
            </div>
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
                placeholder="Kommentar schreiben…"
                maxlength="500"
              />
              <button type="submit" class="btn-submit btn-sm">Senden</button>
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
#forum-header {
  padding: 64px 32px 40px;
  text-align: center;
}

#forum {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0;
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

.community-meta h2 {
  margin: 0 0 4px;
}

.community-desc {
  font-size: 14px;
  color: var(--text);
  margin: 0;
}

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

.btn-new:hover {
  box-shadow: var(--shadow);
}

.post-form,
.comment-form {
  display: flex;
  flex-direction: column;
  gap: 10px;
  background: var(--code-bg);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 16px;
}

.comment-form {
  flex-direction: row;
  align-items: center;
  background: none;
  border: none;
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

.btn-submit {
  align-self: flex-end;
  background: var(--accent);
  color: #fff;
  border: none;
  border-radius: 8px;
  padding: 8px 18px;
  font-size: 14px;
  font-family: var(--sans);
  cursor: pointer;
  transition: opacity 0.2s;
}

.btn-submit:hover {
  opacity: 0.85;
}

.btn-sm {
  padding: 6px 14px;
  font-size: 13px;
  white-space: nowrap;
}

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

.post-card:hover {
  box-shadow: var(--shadow);
}

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
  flex-shrink: 0;
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

@media (max-width: 1024px) {
  #forum {
    grid-template-columns: 1fr;
  }

  .community:nth-child(odd) {
    border-right: none;
  }

  .community {
    padding: 24px 20px;
  }

  #forum-header {
    padding: 40px 20px 24px;
  }
}
</style>
