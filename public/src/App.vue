<template>
  <div id="app" class="container mt-5">
    <div class="card">
      <div class="card-header">
        <b>Nginx Debian Repository for n.wtf</b>
      </div>
      <div class="card-body">
        <p class="card-text">
          Execute the following to add repository to your system.<br /><br />
          <code v-html="code"></code>
        </p>
      </div>
      <div class="card-body">
        <div v-for="(item) in releases" :key="item.release" class="d-inline-block me-2 mb-2">
          <a class="btn" @click="loadRelease(item)" :class="active_release===item.release ? 'btn-active' : 'btn-inactive'" :href="'#'+item.release">
            {{capitalize(item.distro)}} {{item.ver}} ({{item.release}})
          </a>
        </div>
      </div>
      <div class="card-footer">
        <div class="row text-center">
          <div class="col-sm">
            <a class="btn btn-link" href="https://mirror-cdn.xtom.com/sb/nginx/">xTom Mirror</a>
          </div>
          <div class="col-sm">
            <a class="btn btn-link" href="https://n.wtf/">N.WTF</a>
          </div>
          <div class="col-sm">
            <a class="btn btn-link" href="https://debian-repo.xxx.sb/nginx/">Testing Repo</a>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      active_release: 'bullseye',
      releases: [],
      code: ''
    }
  },
  mounted: function () {
    var hash = window.location.hash
    if (hash.length > 1) {
      this.active_release = hash.substr(1, hash.length)
    }
    this.loadData()
  },
  methods: {
    async loadData() {
      var response = await this.$http.get('releases.json')
      var active_release = this.active_release
      this.releases = response.data
      if (this.active_release !== '') {
        var release = this.releases.find(function(c) {return c.release === active_release})
        if (release)
          this.loadRelease(release)
        else
          this.loadRelease(this.releases[0])
      }
      return false
    },
    loadRelease(release) {
      this.active_release = release.release
      let url = window.location.origin + window.location.pathname
      if (release.key === 'public-rsa.key') {
        this.code = 'curl -sS https://n.wtf/' + release.key + ' | apt-key add -<br />'
                  + 'echo "deb ' + url + ' ' + release.release + ' main" > /etc/apt/sources.list.d/n.wtf.list'
      }
      else {
        this.code = 'curl -sS https://n.wtf/' + release.key + ' | gpg --dearmor > /usr/share/keyrings/n.wtf.gpg<br />'
                  + 'echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/n.wtf.gpg] ' + url + ' $(lsb_release -sc) main" > /etc/apt/sources.list.d/n.wtf.list'
      }
    },
    capitalize(value) {
      if (!value) return ''
      value = value.toString()
      return value.charAt(0).toUpperCase() + value.slice(1)
    }
  }
}
</script>

<style>
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
:root {
  --main-dark-1: #22272e;
  --main-dark-2: #2d333b;
  --main-dark-3: #313a46;
  --main-dark-4: #343d48;
  --main-dark-5: #404c5a;
  --main-dark-6: #4b5b6d;
  --main-dark-7: #294d7f;
  --main-dark-8: #2361b6;
  --main-white-1: #fff;
  --main-white-2: #adbac7;
}
body{
  background-color: var(--main-dark-1) !important;
  color: var(--main-white-2) !important;
}
#app .card{
  background-color: var(--main-dark-2);
}
code{
  color: #ff5fa8;
}
#app .btn-inactive{
  color: var(--main-white-2);
  background-color: var(--main-dark-3);
  border-color: var(--main-dark-5)
}
#app .btn-inactive:hover{
  color: var(--main-white-1);
  background-color: var(--main-dark-6);
  border-color: var(--main-dark-5)
}
#app .btn-active{
  color: var(--main-white-1);
  background-color: var(--main-dark-7);
  border-color: var(--main-dark-8)
}
#app .card-header{
  background-color: var(--main-dark-3);
  border-color: var(--main-dark-5);
  font-size: large;
}
#app .card-footer{
  color: var(--main-white-1);
  background-color: var(--main-dark-3);
  border-color: var(--main-dark-5);
}
#app a.btn-link{
  color: #65a6ff;
}
</style>
