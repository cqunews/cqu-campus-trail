<script setup lang="ts">
import { computed, nextTick, onMounted, onUnmounted, ref } from 'vue'

type AbilityName = keyof ToySDK.Toy
interface Example {
  name: AbilityName
  title: string
  description: string
  args: unknown[]
  returns: unknown
  caution?: string
}

const examples: Example[] = [
  { name: 'isSupport', title: '检查能力', description: '判断当前环境是否支持某项能力。', args: ['getQrCode'], returns: true },
  { name: 'navigate', title: '站内跳转', description: '点击执行后跳转到 B 站视频页。', args: [{ type: 'video', id: 'BV1Hh411S7Ys', extra: { from: 'toy' } }], returns: null, caution: '会离开当前页面。' },
  { name: 'saveImageToAlbum', title: '保存图片', description: '将网络图片保存到系统相册。', args: [{ url: 'https://i0.hdslb.com/bfs/archive/example.jpg', hintMsg: '保存测试图片' }], returns: { localPath: '/storage/emulated/0/Pictures/example.jpg' }, caution: '会请求相册权限；请替换为有效图片地址。' },
  { name: 'share', title: '拉起分享', description: '分享当前 Toy 内的页面路径。', args: [{ path: 'index.html?from=test' }], returns: null, caution: '会打开系统分享面板；仅 App 内支持。' },
  { name: 'getQrCode', title: '生成二维码', description: '返回当前 Toy 页面二维码的 PNG Data URL 与目标链接。', args: [{ path: 'index.html', size: 240 }], returns: { base64: 'data:image/png;base64,...', url: 'https://www.bilibili.com/toy/<slug>/index.html' } },
  { name: 'closeBrowser', title: '关闭容器', description: '关闭当前 B 站 WebView。', args: [], returns: null, caution: '会关闭当前页面；仅 App 内支持。' },
  { name: 'getUserProfile', title: '获取用户资料', description: '读取头像、昵称和当前 Toy 内的假名标识。', args: [], returns: { avatar: 'https://i0.hdslb.com/bfs/face/example.jpg', nickname: '示例用户', toyOpenId: 'toy_open_id_example' }, caution: '首次调用可能出现用户数据确认弹窗。' },
  { name: 'getAuthorProfile', title: '获取作者资料', description: '读取当前 Toy 作者公开资料。', args: [], returns: { status: 'ok', data: { nickname: '作者昵称', avatar: 'https://i0.hdslb.com/bfs/face/example.jpg', follower: 1000, following: 20, archiveCount: 30 } } },
  { name: 'getAuthorVideos', title: '获取作者视频', description: '按 aid / bvid 查询当前作者参与创作的视频。', args: [{ videos: [{ bvid: 'BV17x411w7KC' }] }], returns: { status: 'ok', items: [{ ref: { bvid: 'BV17x411w7KC' }, status: 'ok', data: { aid: 170001, bvid: 'BV17x411w7KC', title: '示例视频', stat: { view: 100, like: 10 } } }] } },
  { name: 'getAuthorRelation', title: '获取作者关系', description: '读取关注、粉丝勋章等关系状态。', args: [], returns: { status: 'ok', data: { isFollowing: false, isAuthor: false, isOldFan: false, hasFanMedal: false, isCharging: false } } },
  { name: 'getVideoUserActions', title: '获取视频互动状态', description: '读取点赞、投币、收藏状态。', args: [{ aids: [170001] }], returns: { status: 'ok', items: [{ aid: 170001, bvid: 'BV17x411w7KC', status: 'ok', liked: false, coinCount: 0, favorited: false }] } },
  { name: 'getCloudStorage', title: '读取云存储', description: '读取当前用户在当前 Toy 下的数据。', args: [['codex_sdk_demo']], returns: { codex_sdk_demo: 'example' } },
  { name: 'setCloudStorage', title: '写入云存储', description: '按 key 批量写入当前 Toy 云存储。', args: [{ codex_sdk_demo: 'example' }], returns: null, caution: '会在当前用户的 Toy 云存储中写入示例键值。' },
  { name: 'removeCloudStorage', title: '删除云存储', description: '删除指定云存储键。', args: [['codex_sdk_demo']], returns: null, caution: '会删除当前用户云存储中的 codex_sdk_demo。' },
  { name: 'getSharedStorage', title: '读取共享存档', description: '读取指定共享存档数据。', args: ['st_7f3a9c2e8b1d4g5h', ['progress']], returns: { progress: '42' }, caution: '请将示例 storageId 替换为你有权限的存档 ID。' },
  { name: 'setSharedStorage', title: '写入共享存档', description: '向指定共享存档写入键值。', args: ['st_7f3a9c2e8b1d4g5h', { codex_sdk_demo: 'example' }], returns: null, caution: '会修改共享存档；请使用你有权限的 storageId。' },
  { name: 'removeSharedStorage', title: '删除共享存档数据', description: '删除当前玩家在指定共享存档中的键。', args: ['st_7f3a9c2e8b1d4g5h', ['codex_sdk_demo']], returns: null, caution: '会删除共享存档数据；请使用你有权限的 storageId。' },
  { name: 'submitScore', title: '提交排行榜分数', description: '提交总榜分数；服务端只保留历史最高分。', args: [{ board: 1, score: 0 }], returns: { score: 0 }, caution: '会向当前 Toy 榜位 1 提交分数；首次可能出现数据确认。' },
  { name: 'getRankList', title: '读取排行榜', description: '读取指定榜位和周期的排行榜。', args: [{ board: 1, period: 'week', limit: 10 }], returns: [{ rank: 1, score: 999, nickname: '示例用户', avatar: 'https://i0.hdslb.com/bfs/face/example.jpg' }] },
  { name: 'getMyRank', title: '读取我的排名', description: '读取当前用户的上榜状态和分数。', args: [{ board: 1, period: 'week' }], returns: { ranked: true, rank: 12, score: 88 } },
  { name: 'checkInputSafety', title: '检查文本安全', description: '检测提交文本是否可以展示。', args: [{ text: '测试留言', scene: 'comment' }], returns: { passed: true, status: 'ok' } },
  { name: 'requestCamera', title: '申请摄像头', description: '申请摄像头并返回实时 MediaStream。', args: [{ facingMode: 'user' }], returns: { kind: 'MediaStream', active: true, tracks: ['video'] }, caution: '会请求业务授权和系统摄像头权限；点击后才会调用。' },
  { name: 'requestMicrophone', title: '申请麦克风', description: '申请麦克风并返回实时 MediaStream。', args: [], returns: { kind: 'MediaStream', active: true, tracks: ['audio'] }, caution: '会请求业务授权和系统麦克风权限；点击后才会调用。' },
  { name: 'stopMedia', title: '释放媒体流', description: '释放本页最近一次申请的摄像头或麦克风流。', args: ['<上一步返回的 MediaStream>'], returns: null, caution: '仅可释放本页通过 Toy SDK 申请的媒体流。' },
  { name: 'onContainerChange', title: '监听容器状态', description: '订阅状态变化并立即返回当前完整状态；可再次点击取消监听。', args: ['(state) => console.log(state)'], returns: { deviceType: 'phone', viewport: { width: 390, height: 844 }, orientation: 'portrait', immersive: false, safeArea: { top: 0, right: 0, bottom: 0, left: 0 }, changedFields: [] } },
  { name: 'getContainerState', title: '读取容器状态', description: '读取设备类型、尺寸、方向和安全区。', args: [], returns: { deviceType: 'phone', viewport: { width: 390, height: 844 }, orientation: 'portrait', immersive: false, safeArea: { top: 0, right: 0, bottom: 0, left: 0 }, changedFields: [] } },
  { name: 'setContainerMode', title: '切换容器模式', description: '请求切换方向或沉浸模式。', args: [{ orientation: 'portrait', immersive: false }], returns: null, caution: '可能改变 Toy 容器方向或沉浸状态；Promise 完成不代表状态已生效。' },
]

const support = ref<Partial<Record<AbilityName, boolean>>>({})
const inputs = ref<Record<string, string>>(Object.fromEntries(examples.map((item) => [item.name, JSON.stringify(item.args, null, 2)])))
const results = ref<Record<string, string>>({})
const running = ref<Record<string, boolean>>({})
const sdkAvailable = ref(false)
const sharedStream = ref<MediaStream>()
const showCameraPreview = ref(false)
const cameraPreview = ref<HTMLVideoElement>()
const listening = ref(false)
let stopListening: (() => void) | undefined

const supportedCount = computed(() => Object.values(support.value).filter(Boolean).length)

function format(value: unknown) {
  return JSON.stringify(value, null, 2) ?? 'null'
}

function displayResult(value: unknown): unknown {
  if (value instanceof MediaStream) {
    sharedStream.value = value
    return { kind: 'MediaStream', active: value.active, tracks: value.getTracks().map((track) => ({ kind: track.kind, label: track.label, readyState: track.readyState })) }
  }
  return value
}

async function checkAbilities() {
  sdkAvailable.value = typeof window.toy !== 'undefined'
  if (!sdkAvailable.value) return
  const values = await Promise.all(examples.map(async ({ name }) => [name, await window.toy.isSupport(name)] as const))
  support.value = Object.fromEntries(values) as Partial<Record<AbilityName, boolean>>
}

async function invoke(example: Example) {
  if (!sdkAvailable.value) return
  running.value[example.name] = true
  try {
    const args = JSON.parse(inputs.value[example.name] || '[]') as unknown[]
    let result: unknown
    const api = window.toy as unknown as Record<string, (...params: unknown[]) => unknown>
    if (example.name === 'onContainerChange') {
      if (stopListening) {
        stopListening()
        stopListening = undefined
        listening.value = false
        results.value[example.name] = '监听已取消。'
        return
      }
      listening.value = true
      results.value[example.name] = '监听已启动，收到状态后会在此显示。'
      stopListening = window.toy.onContainerChange((state) => {
        results.value[example.name] = format(state)
      })
      return
    }
    if (example.name === 'stopMedia') {
      if (!sharedStream.value) throw new Error('请先成功调用 requestCamera 或 requestMicrophone。')
      await window.toy.stopMedia(sharedStream.value)
      sharedStream.value = undefined
      showCameraPreview.value = false
      if (cameraPreview.value) cameraPreview.value.srcObject = null
      result = null
    } else {
      result = await api[example.name](...args)
    }
    if (example.name === 'requestCamera' && result instanceof MediaStream) {
      sharedStream.value = result
      showCameraPreview.value = true
      await nextTick()
      if (cameraPreview.value) {
        cameraPreview.value.srcObject = result
        await cameraPreview.value.play().catch(() => {})
      }
    }
    if (example.name === 'getQrCode' && result && typeof result === 'object') {
      const qr = result as ToySDK.QrCodeResp
      results.value[example.name] = format({ url: qr.url, base64: qr.base64.slice(0, 48) + '…', length: qr.base64.length })
      return
    }
    results.value[example.name] = format(displayResult(result))
  } catch (cause) {
    const err = cause as Error & { type?: string; code?: number }
    results.value[example.name] = format({ name: err.name, message: err.message, type: err.type, code: err.code })
  } finally {
    running.value[example.name] = false
  }
}

onMounted(checkAbilities)
onUnmounted(() => stopListening?.())
</script>

<template>
  <main class="page">
    <div class="heading">
      <div><h1>Toy 能力测试</h1><p>每项均展示调用参数、返回示例和本次实际结果。</p></div>
      <button :disabled="!sdkAvailable" @click="checkAbilities">重新检测</button>
    </div>
    <div v-if="!sdkAvailable" class="notice">未检测到 Toy SDK，请在 Toy 环境中打开此页面。</div>
    <div v-else class="summary">已支持 {{ supportedCount }} / {{ examples.length }} 项；实际返回以当前 Toy 和账号状态为准。</div>
    <video v-if="showCameraPreview" ref="cameraPreview" class="preview" autoplay playsinline muted />
    <section>
      <details v-for="item in examples" :key="item.name" class="ability">
        <summary>
          <span><b>{{ item.title }}</b><code>toy.{{ item.name }}</code></span>
          <em v-if="support[item.name] === true" class="yes">支持</em>
          <em v-else-if="support[item.name] === false" class="no">不支持</em>
          <em v-else class="pending">待检测</em>
        </summary>
        <div class="content">
          <p>{{ item.description }}</p>
          <p v-if="item.caution" class="caution">{{ item.caution }}</p>
          <label>调用参数（JSON 数组；onContainerChange / stopMedia 使用说明占位）</label>
          <textarea v-model="inputs[item.name]" spellcheck="false" />
          <div class="code-title">预期返回示例</div>
          <pre>{{ format(item.returns) }}</pre>
          <button :disabled="!sdkAvailable || running[item.name]" @click="invoke(item)">
            {{ running[item.name] ? '调用中…' : item.name === 'onContainerChange' && listening ? '取消监听' : '执行示例' }}
          </button>
          <div v-if="results[item.name]" class="actual">
            <div class="code-title">本次实际结果（JSON）</div>
            <pre>{{ results[item.name] }}</pre>
          </div>
        </div>
      </details>
    </section>
  </main>
</template>

<style scoped>
.page { max-width: 820px; margin: 40px auto; padding: 0 20px 48px; }
.heading { display: flex; align-items: center; justify-content: space-between; margin-bottom: 20px; }
h1 { margin: 0 0 6px; font-size: 24px; font-weight: 600; }
p { margin: 0 0 12px; color: #606266; }
.summary { margin: 12px 0; color: #606266; font-size: 14px; }
.notice, .caution { color: #b88230; background: #fdf6ec; border-radius: 4px; padding: 10px 12px; font-size: 13px; }
.caution { margin: 8px 0; }
button { border: 0; border-radius: 4px; background: #409eff; color: white; padding: 8px 14px; cursor: pointer; }
button:disabled { opacity: .55; cursor: default; }
section { border: 1px solid #ebeef5; border-radius: 8px; overflow: hidden; }
.ability { border-bottom: 1px solid #ebeef5; }
.ability:last-child { border-bottom: 0; }
summary { min-height: 58px; padding: 8px 14px; display: flex; align-items: center; justify-content: space-between; cursor: pointer; list-style-position: inside; }
summary span { display: inline-flex; flex-direction: column; gap: 3px; }
summary b { font-size: 14px; font-weight: 600; }
code { color: #909399; font-size: 12px; }
em { font-size: 13px; font-style: normal; }
.yes { color: #67c23a; }.no, .pending { color: #909399; }
.content { padding: 0 16px 16px 34px; }
label, .code-title { display: block; margin: 12px 0 6px; color: #606266; font-size: 13px; }
textarea { width: 100%; min-height: 72px; padding: 8px; resize: vertical; border: 1px solid #dcdfe6; border-radius: 4px; color: #303133; font: 12px/1.5 ui-monospace, SFMono-Regular, Menlo, monospace; }
pre { max-height: 240px; overflow: auto; margin: 0 0 12px; padding: 10px; border-radius: 4px; background: #f5f7fa; color: #303133; font: 12px/1.5 ui-monospace, SFMono-Regular, Menlo, monospace; white-space: pre-wrap; overflow-wrap: anywhere; }
.actual { margin-top: 12px; }
.preview { width: 100%; max-height: 300px; background: #111; object-fit: contain; margin: 0 0 16px; border-radius: 8px; }
@media (max-width: 520px) { .page { margin-top: 24px; }.heading { align-items: flex-start; gap: 10px; } .content { padding-left: 16px; } }
</style>
