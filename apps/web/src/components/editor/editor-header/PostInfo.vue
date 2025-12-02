<script setup lang="ts">
import type { Post, PostAccount } from '@md/shared/types'
import { Info, Send } from 'lucide-vue-next'
import { useEditorStore } from '@/stores/editor'
import { useRenderStore } from '@/stores/render'
import { useUIStore } from '@/stores/ui'

defineOptions({
  inheritAttrs: false,
})

const editorStore = useEditorStore()
const { editor } = storeToRefs(editorStore)

const renderStore = useRenderStore()
const { output } = storeToRefs(renderStore)

const uiStore = useUIStore()
const { isMobile } = storeToRefs(uiStore)

const dialogVisible = ref(false)
const allAccounts = ref<PostAccount[]>([])
const postTaskDialogVisible = ref(false)

const form = ref<Post>({
  title: ``,
  desc: ``,
  thumb: ``,
  content: ``,
  markdown: ``,
  accounts: [] as PostAccount[],
})

// const allowPost = computed(() => extensionInstalled.value && form.value.accounts.some(a => a.checked))

async function prePost() {
  let auto: Post = {
    thumb: ``,
    title: ``,
    desc: ``,
    content: ``,
    markdown: ``,
    accounts: [],
  }
  const accounts = allAccounts.value.filter(a => ![`ipfs`].includes(a.type))
  try {
    auto = {
      thumb: document.querySelector<HTMLImageElement>(`#output img`)?.src ?? ``,
      title: [1, 2, 3, 4, 5, 6]
        .map(h => document.querySelector(`#output h${h}`)!)
        .find(h => h)
        ?.textContent ?? ``,
      desc: document.querySelector(`#output p`)?.textContent?.trim() ?? ``,
      content: output.value,
      markdown: editor.value?.state.doc.toString() ?? ``,
      accounts,
    }
  }
  catch (error) {
    console.log(`error`, error)
  }
  finally {
    form.value = {
      ...auto,
    }

    console.log('form.value', form.value)
  }
}

declare global {
  interface Window {
    syncPost: (data: { thumb: string, title: string, desc: string, content: string }) => void
    $syncer: any
  }
}

function post() {
  form.value.accounts = form.value.accounts.filter(a => a.checked)
  postTaskDialogVisible.value = true
  dialogVisible.value = false
}

function onUpdate(val: boolean) {
  if (!val) {
    dialogVisible.value = false
  }
}

onBeforeMount(() => {
})
</script>

<template>
  <div v-bind="$attrs">
    <Dialog v-model:open="dialogVisible" @update:open="onUpdate">
      <DialogTrigger>
        <Button v-if="!isMobile" variant="outline" class="h-9" @click="prePost">
          <Send class="mr-2 h-4 w-4" />
          发布
        </Button>
      </DialogTrigger>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>发布</DialogTitle>
          <DialogDescription>
            将文章发布到微信公众平台草稿箱
          </DialogDescription>
        </DialogHeader>
        <Alert>
          <Info class="h-4 w-4" />
          <AlertTitle>提示</AlertTitle>
          <AlertDescription>
            此功能由第三方浏览器插件支持，本平台不保证安全性及同步准确度。
          </AlertDescription>
        </Alert>

        <div class="w-full flex items-center gap-4">
          <Label for="thumb" class="w-10 text-end">
            封面
          </Label>
          <Input id="thumb" v-model="form.thumb" placeholder="自动提取第一张图" />
        </div>
        <div class="w-full flex items-center gap-4">
          <Label for="title" class="w-10 text-end">
            标题
          </Label>
          <Input id="title" v-model="form.title" placeholder="自动提取第一个标题" />
        </div>
        <div class="w-full flex items-start gap-4">
          <Label for="desc" class="w-10 text-end">
            描述
          </Label>
          <Textarea id="desc" v-model="form.desc" placeholder="自动提取第一个段落" />
        </div>

        <DialogFooter>
          <Button variant="outline" @click="dialogVisible = false">
            取 消
          </Button>
          <Button @click="post">
            确 定
          </Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>

    <PostTaskDialog v-model:open="postTaskDialogVisible" :post="form" />
  </div>
</template>
