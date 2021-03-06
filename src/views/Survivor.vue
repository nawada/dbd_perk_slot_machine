<template>
    <div>
        <div>
            <perkslot0 @reRollRequested="randomize"
                       ref="perkslot0"
                       type="Surv"
                       :elementLength="elementLength"
                       :colorized="color"
                       :lang="lang"
                       :perkData="Object.keys(perksSHD.frames)"
            />
            <perkslot1 @reRollRequested="randomize"
                       ref="perkslot1"
                       type="Surv"
                       :elementLength="elementLength"
                       :colorized="color"
                       :lang="lang"
                       :perkData="Object.keys(perksSHD.frames)"
            />
            <perkslot2 @reRollRequested="randomize"
                       ref="perkslot2"
                       type="Surv"
                       :elementLength="elementLength"
                       :colorized="color"
                       :lang="lang"
                       :perkData="Object.keys(perksSHD.frames)"
            />
            <perkslot3 @reRollRequested="randomize"
                       ref="perkslot3"
                       type="Surv"
                       :elementLength="elementLength"
                       :colorized="color"
                       :lang="lang"
                       :perkData="Object.keys(perksSHD.frames)"
            />
        </div>
        <div v-if="hintVisible" class="hint-text">
            <img src="/img/icon_shortinfo.png" slot="icon" alt="Survivor" class="info-icon">
            <span v-html="$t('snippets.startPerkRoll')"></span>
        </div>
    </div>
</template>

<script>
import PixiPerkSlot from '../components/PixiPerkSlot'
import rand from '@/lib/randomize'
import vp from '@/lib/viewport'

export default {
  name: 'Survivor',
  components: {
    'perkslot0': PixiPerkSlot,
    'perkslot1': PixiPerkSlot,
    'perkslot2': PixiPerkSlot,
    'perkslot3': PixiPerkSlot
  },
  props: {
    lang: {
      type: String,
      required: true
    },
    color: {
      type: Boolean,
      default: false,
      required: false
    },
    sids: {
      type: Array,
      default: function () {
        return []
      },
      required: false
    },
    kids: {
      type: Array,
      default: function () {
        return []
      },
      required: false
    },
    perksSHD: {
      type: Object,
      required: true
    }
  },
  data: function () {
    let survivorsRaw = Object.keys(this.perksSHD.frames)
    let survivors = []
    // make sure array keys match the ids in file name. TODO maybe make sure no key is reassigned because of naming issues
    for (let i = 0, sLen = survivorsRaw.length; i < sLen; i++) {
      let perkFileName = survivorsRaw[i]
      let key = Number(perkFileName.substr(0, 2))
      survivors[key] = {
        'index': key,
        'name': perkFileName
      }
    }
    return {
      perkData: survivors,
      hintVisible: true,
      elementLength: vp.getElementLength(),
      lastRoll: []
    }
  },
  methods: {
    randomize: function (el, ev) {
      // if shift is pressed, only randomize the perk that was clicked
      if (el && ev.data.originalEvent.shiftKey && (this.sids.length > 4 || this.sids.length === 0)) {
        let randomSingle = rand.getRandomData(1, this.sids, this.perkData, this.lastRoll)
        let counter = 0
        // while the perk is in the last roll, get a new random perk, give up after 10 tries
        while (this.lastRoll.map((e, i) => e.name === randomSingle[0].name ? i : '').filter(String).length > 0) {
          if (counter > 10) return
          randomSingle = rand.getRandomData(1, this.sids, this.perkData, this.lastRoll)
          counter++
        }
        // update the lastroll with the new perk, using the component tag (e.g. 'perkslot0') to get the index
        let replaceIndex = parseInt(el.$options._componentTag.slice(-1))
        this.lastRoll[replaceIndex] = randomSingle[0]
        // actually roll the slot to the new perk
        el.rollWheel(randomSingle[0], this.$t(`perks.survivor.desc.${randomSingle[0].name}`))
        return
      }
      if (!this.lastRoll) this.lastRoll = [this.perkData[0], this.perkData[0], this.perkData[0], this.perkData[0]]
      let random = rand.getRandomData(4, this.sids, this.perkData, this.lastRoll)
      this.hintVisible = false

      this.lastRoll = random
      this.$refs.perkslot0.rollWheel(random[0], this.$t(`perks.survivor.desc.${random[0].name}`))
      this.$refs.perkslot1.rollWheel(random[1], this.$t(`perks.survivor.desc.${random[1].name}`))
      this.$refs.perkslot2.rollWheel(random[2], this.$t(`perks.survivor.desc.${random[2].name}`))
      this.$refs.perkslot3.rollWheel(random[3], this.$t(`perks.survivor.desc.${random[3].name}`))
    }
  },
  mounted: function () {
    if (this.$route.query.streammode === '1') {
      const hints = document.getElementsByClassName('hint-text')
      if (hints.length > 0) {
        hints[0].setAttribute('style', 'display:none;')
      }
      const containers = document.getElementsByClassName('container')
      if (containers.length > 1) {
        containers[1].setAttribute('style', 'padding:0;max-width:none;')
      }
      const headers = document.getElementsByClassName('header')
      if (headers.length > 0) {
        headers[0].setAttribute('style', 'display:none;')
      }
      const corners = document.getElementsByClassName('github-corner')
      if (corners.length > 0) {
        corners[0].setAttribute('style', 'display:none;')
      }
    } else {
      document.getElementsByTagName('body')[0].removeAttribute('style')
    }
    const obs = this.$route.query.obs
    const as = parseInt(this.$route.query.autostart)
    if (!isNaN(as) && as > 0 && obs !== '1') {
      window.setTimeout(() => {
        this.randomize()
      }, as)
    }
    if (obs === '1' && window.obsstudio) {
      const delayedRand = (visible) => {
        if (!visible) return
        window.setTimeout(() => {
          this.randomize()
        }, !isNaN(as) && as > 0 ? as : 1000)
      }
      if (window.obsstudio.linuxbrowser || Number(window.obsstudio.pluginVersion.slice(0, 4)) > 1.29) {
        window.obsstudio.onActiveChange = delayedRand
      } else {
        window.obsstudio.onVisibilityChange = delayedRand
      }
    }
    window.addEventListener('orientationchange', function () {
      let o = window.orientation
      if (o === 90 || o === -90 || o === 0) {
        window.location.reload()
      }
    })
  }
}
</script>

<style lang="scss">
    .hint-text {
        margin: 30px 30%;
        border: 1px solid rgba(255, 255, 255, 0.2);
        background-color: rgba(0, 0, 0, 0.2);
        padding: 10px 20px;
        align-items: center;
        justify-content: center;
        display: flex;

        @media screen and (max-width: 650px) {
            margin: 0 10%;
        }
    }

    .info-icon {
        width: 30px;
        margin: 0 10px;
        height: auto;
    }
</style>
