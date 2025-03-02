<template>
  <div class="flex flex-col flex-1">
    <div
      class="sticky z-10 flex items-center justify-between pl-4 border-b bg-primary border-dividerLight top-lowerSecondaryStickyFold"
    >
      <label class="font-semibold text-secondaryLight">
        {{ t("response.body") }}
      </label>
      <div class="flex">
        <ButtonSecondary
          v-if="response.body"
          v-tippy="{ theme: 'tooltip' }"
          :title="t('action.download_file')"
          :icon="downloadIcon === 'download' ? IconDownload : IconCheck"
          @click="downloadResponse"
        />
      </div>
    </div>
    <img
      class="flex max-w-full border-b border-dividerLight"
      :src="imageSource"
      loading="lazy"
      :alt="imageSource"
    />
  </div>
</template>

<script lang="ts">
import IconDownload from "~icons/lucide/download"
import IconCheck from "~icons/lucide/check"
import { useI18n } from "@composables/i18n"
import { useToast } from "@composables/toast"
import { defineComponent } from "vue"

export default defineComponent({
  props: {
    response: { type: Object, default: () => ({}) },
  },
  setup() {
    return {
      t: useI18n(),
      toast: useToast(),
      IconDownload,
      IconCheck,
    }
  },
  data() {
    return {
      imageSource: "",
      downloadIcon: "download",
    }
  },
  computed: {
    responseType() {
      return (
        this.response.headers.find(
          (h) => h.key.toLowerCase() === "content-type"
        ).value || ""
      )
        .split(";")[0]
        .toLowerCase()
    },
  },
  watch: {
    response: {
      immediate: true,
      handler() {
        this.imageSource = ""

        const buf = this.response.body
        const bytes = new Uint8Array(buf)
        const blob = new Blob([bytes.buffer])

        const reader = new FileReader()
        reader.onload = ({ target }) => {
          this.imageSource = target.result
        }
        reader.readAsDataURL(blob)
      },
    },
  },
  mounted() {
    this.imageSource = ""

    const buf = this.response.body
    const bytes = new Uint8Array(buf)
    const blob = new Blob([bytes.buffer])

    const reader = new FileReader()
    reader.onload = ({ target }) => {
      this.imageSource = target.result
    }
    reader.readAsDataURL(blob)
  },
  methods: {
    downloadResponse() {
      const dataToWrite = this.response.body
      const file = new Blob([dataToWrite], { type: this.responseType })
      const a = document.createElement("a")
      const url = URL.createObjectURL(file)
      a.href = url
      // TODO get uri from meta
      a.download = `${url.split("/").pop().split("#")[0].split("?")[0]}`
      document.body.appendChild(a)
      a.click()
      this.downloadIcon = "check"
      this.toast.success(this.t("state.download_started"))
      setTimeout(() => {
        document.body.removeChild(a)
        URL.revokeObjectURL(url)
        this.downloadIcon = "download"
      }, 1000)
    },
  },
})
</script>
