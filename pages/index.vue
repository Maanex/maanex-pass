<template>
  <div class="pass-root bg-gray-800">
    <div class="pass-inner my-48">
      <input ref="area" class="pass-clipboard">
      <h1 class="mb-5 text-xl font-black text-green-400 font-mono ">
        pass.maanex.me
      </h1>
      <input
        v-model="masterpw"
        class="w-full py-3 px-5 rounded bg-gray-600 shadow-md text-gray-50 mb-6 tracking-widest"
        type="password"
        placeholder="Master Password"
        @keydown.enter.prevent="poweruser()"
      >
      <div
        v-for="item of generated"
        :key="item.key"
        class="w-full py-3 px-3 rounded bg-gray-700 text-gray-100 mb-1 flex cursor-pointer transition-colors hover:bg-gray-600"
        @click="copyToClipboard(item.password || '')+flashMe(item.key)"
        @click.middle.prevent="removeMe(item)"
      >
        <img
          class="rounded-full"
          :src="`https://logo.clearbit.com/${item.logo ? item.logo.toLowerCase() : ''}?size=32`"
          :alt="item.name"
          draggable="false"
        >
        <div class="mx-3 flex items-center flex-grow">
          <span
            class="font-semibold mr-3"
            v-text="item.name"
          />
          <span
            class="text-gray-500 text-sm"
            v-text="item.key"
          />
        </div>
        <div
          class="w-72 h-full bg-gray-900 rounded-sm flex items-center px-3 transition-colors font-mono"
          :flashme="flashes.includes(item.key)"
          v-text="censorPassword(item.password)"
        />
      </div>
      <div
        class="w-full py-3 px-3 rounded border-2 border-dashed border-gray-700 text-gray-100 mb-1 flex cursor-pointer transition-colors text-gray-500 hover:bg-gray-700 hover:text-gray-200"
        @click="addNew()"
      >
        <span class="font-semibold mr-3 text-inherit">
          Add
        </span>
      </div>
      <div class="pass-spacer" />
      <div class="pass-footer text-gray-700 w-100 text-center mt-20">
        <a href="https://maanex.me/" target="_blank" rel="norefferer" class="w-min transition-colors hover:text-gray-500">Made by Maanex</a>
        using
        <a href="https://nuxtjs.org/" target="_blank" rel="norefferer" class="w-min transition-colors hover:text-gray-500">Nuxt</a>
        &amp;
        <a href="https://tailwindcss.com/" target="_blank" rel="norefferer" class="w-min transition-colors hover:text-gray-500">Tailwind</a>
        &bull;
        <a href="https://clearbit.com" target="_blank" rel="norefferer" class="w-min transition-colors hover:text-gray-500">Logos provided by Clearbit</a>
      </div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'

const rand = (seed, range) => Math.abs(Math.floor(Math.sin(seed) * range))

export default Vue.extend({
  data () {
    return {
      masterpw: '',
      flashes: [],
      generated: []
    }
  },
  head: {
    title: 'mpass',
    link: [{ rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }]
  },
  watch: {
    masterpw (val) {
      this.generated.forEach(g => (g.password = this.generatePassword(g.key)))
    }
  },
  mounted () {
    const def = '[{"name": "Discord","logo": "discord.com","key": "discord1"},{"name": "Spotify","logo": "spotify.com","key": "spotify"},{"name": "Twitter","logo": "twitter.com","key": "twitter"}]'
    this.generated = JSON.parse(localStorage.getItem('services') || def)
  },
  methods: {
    censorPassword (password) {
      if (!password) {
        return '•'.repeat(rand(Math.random(), 4) + 12)
      }
      return password[0] + password[1] + '•'.repeat(rand(password.length + password.charCodeAt(4), 4) + 12)
    },
    copyToClipboard (text) {
      const area = this.$refs.area
      area.value = text
      area.focus()
      area.select()
      document.execCommand('copy')
    },
    flashMe (key) {
      this.flashes.push(key)
      setTimeout(() => {
        this.flashes.splice(0, 1)
      }, 200)
    },
    removeMe (item) {
      if (!confirm(`Remove ${item.name} from your list? You can add it again any time. Continue?`)) {
        return
      }

      this.generated.splice(this.generated.findIndex(e => e.key === item.key), 1)
      Vue.nextTick(() => this.saveLocalstore())
    },
    generatePassword (key) {
      const lowercase = 'abcdefghijklmnopqrstuvwxyz'
      const uppercase = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
      const numbers = '0123456789'
      const specials = '-+!?&_'
      const charset = lowercase + lowercase + lowercase + uppercase + uppercase + uppercase + numbers + numbers + numbers + specials

      let cmaster = 1
      let ckey = 0

      this.masterpw.split('').forEach(a => (cmaster *= a.charCodeAt()))
      key.split('').forEach(a => (ckey += a.charCodeAt() * cmaster * cmaster))

      let ra = ckey
      const length = rand(ra, 2) + 14
      for (let i = 0; i < 10; i++) { ra = rand(ra, Number.MAX_SAFE_INTEGER) }

      let res = ''
      for (let i = 0; i < length; i++) { res += charset[rand((ra += rand(ra, Number.MAX_SAFE_INTEGER)), charset.length - 1)] }

      ra = rand(ra, Number.MAX_SAFE_INTEGER)
      const splitAt = rand(ra, res.length - 3) + 1

      const insert = [
        lowercase[rand(ra, lowercase.length)],
        uppercase[rand(ra, uppercase.length)],
        numbers[rand(ra, numbers.length)],
        specials[rand(ra, specials.length)]
      ].sort((a, b) => rand(ra, 10) - 5).join('')

      res = res.substring(0, splitAt) + insert + res.substring(splitAt)

      return `${res}`
    },
    addNew (iname, ikey, ilogo) {
      if (!iname && !confirm('Add a new service? You can delete services by middle-clicking. Services are stored in your local storage. Continue?')) {
        return
      }

      const name = iname || window.prompt('SERVICE NAME\nThis is a display name, purely for aesthetics') + ''
      const key = ikey || window.prompt('SERVICE KEY\nThe key should be all lowercase. This is used as a seed for the password generator.') + ''
      const logo = ilogo || window.prompt('SERVICE DOMAIN\nUsed to display the logo image. Example: twitter.com, steampowered.com, discord.com') + ''
      this.generated.push({
        name,
        key,
        logo,
        password: ''
      })
      Vue.nextTick(() => {
        Vue.set(this, 'generated', this.generated.sort((a, b) => a.name.localeCompare(b.name)))
        Vue.nextTick(() => this.saveLocalstore())
      })
    },
    saveLocalstore () {
      const payload = JSON.parse(JSON.stringify(this.generated))
      payload.forEach(i => (delete i.password))
      localStorage.setItem('services', JSON.stringify(payload))
    },
    poweruser () {
      const [cmd, ...args] = this.masterpw.split(' ')
      if (cmd === 'add') {
        const name = args.join(' ')[0].toUpperCase() + args.join(' ').substr(1)
        this.addNew(name, args.join(''), args.join('').toLowerCase() + '.com')
        this.masterpw = ''
      }
    }
  }
})
</script>

<style lang="postcss">
.pass-root {
  width: 100vw;
  height: 100vh;
  position: absolute;
  top: 0;
  left: 0;
  margin: 0;
  padding: 0;
  overflow-x: hidden;
  display: flex;
  justify-content: center;
}

.pass-inner {
  width: 90vw;
  max-width: 600px;
  display: flex;
  flex-direction: column;
}

.pass-inner > * {
  flex-grow: 0;
  flex-shrink: 0;
}

.pass-spacer {
  flex-grow: 1;
  flex-shrink: 1;
}

.pass-footer * {
  color: inherit;
}

.pass-clipboard {
  opacity: 0;
  height: 0;
  pointer-events: none;
}

[flashme] { background-color: #34d399 !important; }
</style>
