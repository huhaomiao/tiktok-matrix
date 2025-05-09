<template>
  <dialog ref="buy_liscense_dialog" class="modal">
    <div class="modal-box w-11/12 max-w-5xl overflow-hidden">
      <div class="modal-body">
        <div class="relative isolate px-6 py-6 w-full">
          <div class="absolute inset-x-0 -z-10 transform-gpu overflow-hidden px-36 blur-3xl" aria-hidden="true">
            <div
              class="mx-auto aspect-1155/678 w-[72.1875rem] bg-gradient-to-tr from-secondary to-neutral opacity-30 h-96"
              style="clip-path: polygon(0 0, 100% 0, 100% 100%, 0 100%);"></div>
          </div>
          <div class="flex items-start flex-col w-full gap-3">
            <div class="flex items-center flex-row gap-2 w-full">
              <label class="font-bold w-28">{{ $t('mid') }}: </label>
              <input id="mid" type="text" placeholder="mid" class="input input-md input-bordered ring-1"
                v-model="license.mid" readonly disabled />
              <button @click="copyText(license.mid, $event)" class="btn btn-md btn-primary rounded-l-none">
                {{ $t('copy') }}
              </button>
            </div>
            <div class="flex items-center flex-row gap-2 w-full">
              <label class="font-bold w-28">{{ $t('licenseCode') }}: </label>
              <input type="text" placeholder="xxxx-xxxx-xxxx-xxxx" class="input input-md input-bordered ring-1" v-model="license.license_code" readonly disabled />
              <button @click="copyText(license.license_code, $event)" class="btn btn-md btn-primary rounded-l-none">
                {{ $t('copy') }}
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
    <form method="dialog" class="modal-backdrop">
      <button>close</button>
    </form>
  </dialog>
</template>

<script>
import { writeText } from '@tauri-apps/api/clipboard';
import { message } from '@tauri-apps/api/dialog';

export default {
  name: 'BuyLicense',
  props: {
    license: {
      type: Object,
      required: true
    }
  },
  methods: {
    async show() {
      await this.$emiter('LICENSE', { reload: true })
      this.$refs.buy_liscense_dialog.showModal()
    },
    async copyText(text, event) {
      await writeText(text)
      await this.$emiter('NOTIFY', {
        type: 'success',
        message: this.$t('copied'),
        timeout: 2000
      });
    }
  }
}
</script>
