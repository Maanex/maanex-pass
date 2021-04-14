<template>
  <div class="pass-root bg-gray-800">
    <div class="pass-inner my-48">
      <input ref="area" class="pass-clipboard">
      <h1 class="mb-5 text-xl font-black text-green-400 font-mono flex items-center">
        <img src="/icon.png" alt="" class="h-4 w-4 mr-2">
        mpass
        {{ pwmod ? `— ${pwmod[0]}${pwmod.length%10}` : '' }}
      </h1>
      <input
        v-model="masterpw"
        class="w-full py-3 px-5 rounded bg-gray-600 shadow-md text-gray-50 mb-6 tracking-widest"
        type="password"
        placeholder="Master Password"
        @keydown.enter.prevent="poweruser()"
      >
      <div
        v-for="(item, index) of generated"
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
          v-text="censorPassword(item.password, index)"
        />
      </div>
      <div
        class="w-full py-3 px-3 rounded border-2 border-dashed border-gray-700 text-gray-100 mb-1 flex cursor-pointer transition-colors text-gray-500 hover:bg-gray-700 hover:text-gray-200"
        @click="addNew()"
      >
        <span class="font-semibold mr-3 text-inherit w-full text-center">
          + Add
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

export default Vue.extend({
  data () {
    return {
      masterpw: '',
      pwmod: '',
      pwmodnum: 0,
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
      this.generated.forEach(g => (g.password = this.generatePassword(g.key, val)))
    },
    pwmod (val) {
      this.pwmodnum = val.split('').reduce((p, c, i) => p + c.charCodeAt(0) * (10 ** i), 0) / (8 ** val.length)
    }
  },
  mounted () {
    const def = '[{"name": "Discord","logo": "discord.com","key": "discord1"},{"name": "Spotify","logo": "spotify.com","key": "spotify"},{"name": "Twitter","logo": "twitter.com","key": "twitter"}]'
    this.generated = JSON.parse(localStorage.getItem('services') || def)
    this.pwmod = localStorage.getItem('pwmod') || ''
  },
  methods: {
    rand (seed, range) {
      return Math.abs(Math.floor(Math.sin(seed + this.pwmodnum) * range))
    },
    censorPassword (password, index) {
      if (!password) {
        return '•'.repeat(this.rand(Math.random(), 4) + 12)
      }
      if (index <= 1) {
        return password[0] + password[1] + '•'.repeat(this.rand(password.length + password.charCodeAt(4), 4) + 12)
      }
      if (index <= 3) {
        return password[0] + '•'.repeat(this.rand(password.length + password.charCodeAt(4), 4) + 13)
      }
      return '•'.repeat(this.rand(password.length + password.charCodeAt(4), 4) + 14)
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
    generatePassword (key, password) {
      const lowercase = 'abcdefghijklmnopqrstuvwxyz'
      const uppercase = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
      const numbers = '0123456789'
      const specials = '-+!?&_'
      const charset = lowercase + lowercase + lowercase + uppercase + uppercase + uppercase + numbers + numbers + numbers + specials

      if (!password) { password = this.masterpw }
      key += `${password}     `[2] + `${password}     `[3]

      let cmaster = 1
      let ckey = 0

      password.split('').forEach(a => (cmaster = cmaster * a.charCodeAt() + 42))
      key.split('').forEach(a => (ckey += a.charCodeAt() * cmaster * cmaster + ckey * 0.1))

      let ra = ckey
      const length = this.rand(ra, 2) + 14
      for (let i = 0; i < 10; i++) { ra = this.rand(ra, Number.MAX_SAFE_INTEGER) }

      let res = ''
      for (let i = 0; i < length; i++) { res += charset[this.rand((ra += this.rand(ra, Number.MAX_SAFE_INTEGER)), charset.length - 1)] }

      ra = this.rand(ra, Number.MAX_SAFE_INTEGER)
      const splitAt = this.rand(ra, res.length - 3) + 1

      const insert = [
        lowercase[this.rand(ra, lowercase.length)],
        uppercase[this.rand(ra, uppercase.length)],
        numbers[this.rand(ra, numbers.length)],
        specials[this.rand(ra, specials.length)]
      ].sort((a, b) => this.rand(ra, 10) - 5).join('')

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
      const [cmd, ...args] = this.masterpw.toLowerCase().split(' ')
      if (cmd === 'add') {
        const name = args.join(' ')[0].toUpperCase() + args.join(' ').substr(1)
        this.addNew(name, args.join(''), args.join('').toLowerCase() + '.com')
        this.masterpw = ''
      } else
      if (cmd === 'copy') {
        const key = args[0]
        const pass = args.slice(1).join(' ')
        this.copyToClipboard(this.generatePassword(key, pass))
        this.masterpw = ''
      } else
      if (cmd === 'mod') {
        this.pwmod = args.join(' ')
        localStorage.setItem('pwmod', args.join(' '))
        this.masterpw = ''
      } else
      if (cmd === 'help') {
        alert([
          '▦ add <name>',
          'quickly add a shortcut to name.com',
          '',
          '▦ copy <name> <password>',
          'quickly copy the password for name to your clipboard. useful if someone is watching and you don\'t want to risk anything.',
          '',
          '▦ mod <text>',
          'add a seed modifier. this modifier is stored in your browser and will change up your results even more. you can take something like your name to get different passwords even if someone picks the same masterpassword as you.'
        ].join('\n'))
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
