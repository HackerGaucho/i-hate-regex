<template>
  <section>
    <div
      class="flex my-1 py-2 px-4 text-2xl rounded bg-gray-200 transition"
      :class="{ 'bg-red-200': regexError }"
    >
      <client-only>
        <div class="w-5/6 flex content-center flex-wrap">
          <div
            ref="codebox"
            class="content-center w-full focus:outline-none regex"
            contenteditable="true"
            @paste.prevent="onPaste"
            @keyup="regexChanged"
          >
            {{ regex.source }}
          </div>
        </div>
      </client-only>
      <div class="w-1/6 flex flex-col content-center text-gray-400">
        <div class="w-full text-right">
          <a
            href="#"
            class="text-3xl text-gray-500 hover:text-green-500"
            @click.prevent="toggleFlagSelect"
          >/{{ myflag }}
          </a>
        </div>
        <div class="w-full relative flex content-start text-right">
          <a
            href="#"
            class="text-base w-full hover:text-blue-400"
            :class="{ 'opacity-0': ifCopy }"
            @click.prevent="copyRegex"
          >copy
          </a>
        </div>
      </div>
    </div>
    <client-only>
      <FlagSelector
        :model-show="flagSelectorShow"
        :selections="myflag.split('')"
        @closeModal="toggleFlagSelect"
        @selected="flagsChange"
      />
    </client-only>
    <input ref="regexCopy" v-model="regex" type="text" class="hidden" />
  </section>
</template>

<script>
// import VueRegexColorize from "~/plugins/vue-regex-colorize";
import 'regex-colorize/themes/sweetest.css'
import FlagSelector from '~/components/post-components/utils/FlagSelector'
import pastePlainText from '~/mixins/pastePlainText.js'
//

if (process.client) {
  const RegexColorize =  require('regex-colorize');
  let rgx = new RegexColorize.default();
  rgx.colorizeAll();
}

export default {
  components: {
    FlagSelector
  },
  mixins: [pastePlainText],
  props: {
    regex: RegExp,
    flag: String
  },
  data() {
    return {
      myRegex: null,
      changeTimer: null,
      debounceTimer: null,
      regexError: false,
      myflag: 'gm',
      ifCopy: false, // Variable to show copied message
      flagSelectorShow: false
    }
  },
  mounted() {
    // setting editable flag to prop inherited value
    this.myflag = this.flag
    setTimeout(() => {
      this.regexChanged()
    }, 500)
  },
  methods: {
    regexChanged() {
      clearTimeout(this.debounceTimer)
      this.debounceTimer = setTimeout(() => {
        let isValid = true
        let reg
        const codebox = this.$refs.codebox
        try {
          reg = new RegExp(codebox.innerText, this.myflag)
          this.regexError = false
        } catch (e) {
          isValid = false
          this.regexError = true
        }
        if (this.changeTimer) {
          clearTimeout(this.changeTimer)
        }
        this.changeTimer = setTimeout(() => {
          rgx.colorizeAll()
        }, 3000)
        if (isValid) {
          this.$emit('regexChanged', {
            regex: reg,
            flag: this.myflag
          })
        }
      }, 700)
    },
    toggleFlagSelect() {
      this.flagSelectorShow = !this.flagSelectorShow
    },
    flagsChange(newFlagArray) {
      this.myflag = newFlagArray.join('')
      this.regexChanged()
    },
    selectElementContents(el) {
      const range = document.createRange()
      range.selectNodeContents(el)
      const sel = window.getSelection()
      sel.removeAllRanges()
      sel.addRange(range)
    },
    copyRegex() {
      const regexToCopy = this.$refs.codebox

      // regexToCopy.setAttribute('type', 'text')    //hidden
      this.selectElementContents(regexToCopy)

      try {
        document.execCommand('copy')
        this.$toast.success('Copied')
      } catch (err) {
        this.$toast.error('Oops, unable to copy')
      }

      /* unselect the range */
      // regexToCopy.setAttribute('type', 'hidden')
      window.getSelection().removeAllRanges()
    },
    onPaste(e) {
      const text = (e.originalEvent || e).clipboardData.getData('text/plain')

      document.execCommand('insertHTML', false, text)
    }
  }
}
</script>

<style>
.transition {
  transition: all 0.5s ease;
}

.fade-enter-active,
.fade-leave-active {
  transition: all 0.3s ease-in-out;
  transform: scale(1.5);
}
.fade-enter /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
  transform: scale(1);
}
.fade-leave-to {
  opacity: 0;
  transform: scale(1.5);
}
</style>
