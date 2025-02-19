<script setup lang="ts">
import { useDevtoolsClient } from '../logic/client'

const { position: _position } = useFrameState()

const frameState = computed(() => ({ position: _position.value }))

const client = useDevtoolsClient()
const showPopupUnsupported = ref(false)
const copy = useCopy()

const dockButton = [
  {
    position: 'popup',
    icon: 'i-carbon-launch',
  },
]

function toggle(position: string) {
  if (position === 'popup') {
    // @ts-expect-error missing type
    if (!window.parent.documentPictureInPicture?.requestWindow) {
      showPopupUnsupported.value = true
      return
    }
  }
  else {
    _position.value = position
  }

  client.value?.panel?.togglePosition(position)
}
</script>

<template>
  <div flex="~ gap-1" text-lg>
    <button
      v-for="item in dockButton" :key="item.position"
      :class="[frameState.position === item.position ? 'text-primary' : 'op50', item.icon]"
      @click="toggle(item.position)"
    />
  </div>

  <!-- popup mode not supported message -->
  <VDialog
    v-model="showPopupUnsupported" class="popup-dialog z-2000 max-w-150 p6 pt-2"
    @close="showPopupUnsupported = false"
  >
    <h1 text-3xl>
      Popup is not Supported
    </h1>
    <p>
      To popup the DevTools, it requires the <a
        href="https://developer.chrome.com/docs/web-platform/document-picture-in-picture/" target="_blank" font-bold
        underline
      >Document
        Picture-in-Picture API</a> which is currently in experimental state.
    </p>
    <p>
      As June 2023, the API is only available in Chrome 111 and above, under a flag
      <code>#document-picture-in-picture-api</code>.
    </p>
    <p>
      Your current browser does not seem to support the API, or the flag is not enabled yet.
      You can try enabling the flag by visiting
      <VButton n="xs primary" title="Click to Copy" @click="copy('chrome://flags/#document-picture-in-picture-api')">
        chrome://flags/#document-picture-in-picture-api
      </VButton>
      and restart the browser.
    </p>
    <div>
      <VButton @click="showPopupUnsupported = false">
        Close
      </VButton>
    </div>
  </VDialog>
</template>
