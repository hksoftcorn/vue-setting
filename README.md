

# vue cli & youtube pjt

## 1. settings

### 1.1. vscode

```
npm install --global yarn
yarn --version

vue create vue-youtube
	Manually select features
	<<설정 : 아래 이미지 참고>>

cd vue-youtube
yarn serve

yarn add gh-pages -D
```

![image-20210711131509926](README.assets/image-20210711131509926.png)

### 1.2. GitHub

```
git init
git add .
git remote add origin https://github.com/hksoftcorn/vue-youtube.git
git commit -m "first commit"
git push -u origin master
```



### 1.3. vscode

```
// package.json

{
  "name": "vue-youtube",
  "version": "0.1.0",
  "private": true,
  "homepage": "https://hksoftcorn.github.io/vue-youtube",
  "scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build",
    "predeploy": "vue-cli-service build",
    "deploy": "gh-pages -d dist",
    "clean": "gh-pages-clean",
    "test:unit": "vue-cli-service test:unit",
    "lint": "vue-cli-service lint"
  },
  "dependencies": {
    "core-js": "^3.6.5",
    "vue": "^3.0.0"
  },
  "devDependencies": {
    "@vue/cli-plugin-babel": "~4.5.0",
    "@vue/cli-plugin-eslint": "~4.5.0",
    "@vue/cli-plugin-unit-jest": "~4.5.0",
    "@vue/cli-service": "~4.5.0",
    "@vue/compiler-sfc": "^3.0.0",
    "@vue/eslint-config-prettier": "^6.0.0",
    "@vue/test-utils": "^2.0.0-0",
    "babel-eslint": "^10.1.0",
    "eslint": "^6.7.2",
    "eslint-plugin-prettier": "^3.3.1",
    "eslint-plugin-vue": "^7.0.0",
    "gh-pages": "^3.2.3",
    "prettier": "^2.2.1",
    "typescript": "~3.9.3",
    "vue-jest": "^5.0.0-0"
  }
}

```

```
// root 폴더에 
// vue.config.js

module.exports = {
  publicPath: "/vue-youtube/",
  outputDir: "dist"
}
```

```
$ yarn deploy
```



### 1.4. GitHub

```
settings - pages - 생성된 url 확인

repository - about - website 설정
```

```
// Github Actions
<< edit >>
deploy.yml
name: Deployment
start commit 클릭 - 빌드 확인 - 성공
```



### 1.5. vscode

```
$ git pull
```

```
// deploy.yml 
# This is a basic workflow to help you get started with Actions

name: Deployment

# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the master branch
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - name: Checkout source code\
        uses: actions/checkout@master

      - name: Set up Node.js
        uses: actions/setup-node@master
        with:
          node-version: 14.x

      # Runs a single command using the runners shell
      - name: Install dependencies
        run: yarn install

      - name: Test unit
        run: yarn test:unit

      # Runs a set of commands using the runners shell
      - name: Build page
        run: yarn build
        env:
          NODE_ENV: production

      - name: Deploy to gh-pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
```

```
git add .
git commit -m "add deploy work flow"
git push origin master

github action 확인
```



## 2. Youtube-pjt

### 2.1. Youtube API

- event의 데이터가 어디에 있는 지 확인! : event.target.value
- emit 이벤트 이름은 kebab-case

```vue
// src - componenets - SearchBar.vue

<template>
  <div>
    <input type="text" @keyup.enter="onInputKeyword">
  </div>
</template>

<script>
export default {
  name: 'SearchBar',
  methods: {
    onInputKeyword: function (event) {
      // console.log(event.target.value)
      this.$emit('input-search', event.target.value)
    }
  }
}
</script>

<style>

</style>
```

- 데이터가 emit으로 잘 들어오는 지 확인

```vue
// src - App.vue

<template>
  <div id="app">
    <h1>My Youtube Project</h1>
    <SearchBar @input-search="onInputSearch"/>
  </div>
</template>

<script>
import SearchBar from '@/components/SearchBar.vue'

export default {
  name: 'App',
  components: {
    SearchBar,
  },
  data: function () {
    return {
      searchData: '',
    }
  },
  methods: {
    onInputSearch: function (searchData) {
      console.log('searchbar의 이벤트가 감지되었습니다!')
      console.log(searchData)
      this.searchData = searchData
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

```

- Youtube API KEY

```vue
// .env.local
VUE_APP_YOUBUE_API_KEY='AIzaSyBRBRAeobINPv1_D3twK5lI2-hbn0GUG64'

// App.vue
const API_KEY = process.env.VUE_APP_YOUBUE_API_KEY

// youtube data api
// https://developers.google.com/youtube/v3/docs/search
```

- Axios 설치

```
yarn add axios
```

- Youtube로부터 Data 받아오기

```VUE
<template>
  <div id="app">
    <h1>My Youtube Project</h1>
    <SearchBar @input-search="onInputSearch"/>
  </div>
</template>

<script>
import axios from 'axios'
import SearchBar from '@/components/SearchBar.vue'

const API_KEY = process.env.VUE_APP_YOUTUBE_API_KEY
const API_URL = 'https://www.googleapis.com/youtube/v3/search'

export default {
  name: 'App',
  components: {
    SearchBar,
  },
  data: function () {
    return {
      searchData: '',
      videos: [],
    }
  },
  methods: {
    onInputSearch: function (searchData) {
      // console.log('searchbar의 이벤트가 감지되었습니다!')
      // console.log(searchData)
      this.searchData = searchData

      const params = {
        key: API_KEY,
        part: 'snippet',
        type: 'video',
        q: this.searchData,
      }

      axios({
        url: API_URL,
        methods: 'get',
        params,
      })
        .then(response => {
          // console.log(response)
          console.log(response.data.items)
          this.videos = response.data.items
        })
        .catch(error => {
          console.log(error)
        })
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

```

![image-20210711160709542](README.assets/image-20210711160709542.png)



### 2.2. App - VideoList - VideoListItem

```vue
<template>
  <div id="app">
    <h1>My Youtube Project</h1>
    <SearchBar @input-search="onInputSearch"/>
      
    <!-- 왼쪽은 props를 받을 곳의 이름 // 오른쪽은 여기서 보내는 데이터 값 -->
    <VideoList :videos="videos"/>
  </div>
</template>

<script>
import axios from 'axios'
import SearchBar from '@/components/SearchBar.vue'
import VideoList from '@/components/VideoList.vue'


const API_KEY = process.env.VUE_APP_YOUTUBE_API_KEY
const API_URL = 'https://www.googleapis.com/youtube/v3/search'

export default {
  name: 'App',
  components: {
    SearchBar,
    VideoList,
  },
  data: function () {
    return {
      searchData: '',
      videos: [],
    }
  },
  methods: {
    onInputSearch: function (searchData) {
      // console.log('searchbar의 이벤트가 감지되었습니다!')
      // console.log(searchData)
      this.searchData = searchData

      const params = {
        key: API_KEY,
        part: 'snippet',
        type: 'video',
        q: this.searchData,
      }

      axios({
        url: API_URL,
        method: 'get',
        params,
      })
        .then(response => {
          // console.log(response)
          console.log(response.data.items)
          this.videos = response.data.items
        })
        .catch(error => {
          console.log(error)
        })
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

```

```vue
// VideoList.vue

<template>
  <ul>
    <VideoListItem 
      v-for="(video, index) in videos" 
      :key="index"
      :video="video"
    />
  </ul>
</template>

<script>
import VideoListItem from '@/components/VideoListItem.vue'

export default {
  name: 'VideoList',
  components: {
    VideoListItem,
  },
  props: {
    // 상위에서 받은(props) 데이터를 상세하게 적습니다.
    videos: {
      type: Array,
    }
  }
}
</script>

<style>

</style>

```

- 이미지 URL은 미리 계산해둡니다 computued

```vue
// VideoListItem

<template>
  <li style="text-align: left;">
    <img :src="youtubeImgSrc" alt="img">
    {{ video.snippet.title }}
  </li>
</template>

<script>
export default {
  name: 'VideoListItem',
  props: {
    video: {
      type: Object,
    }
  },
  computed: {
    youtubeImgSrc: function () {
      return this.video.snippet.thumbnails.default.url
    }
  }
}
</script>

<style>

</style>
```

#### 👀 unescaped 특수문자 처리

- 기존의 filters를 이용한 방법이 3.0 버전 업데이트로 적용 x ( Disallow using deprecated filters syntax (in Vue.js 3.0.0+)

- computed 방법으로 해결

  ```
  yarn add lodash
  ```

  ```vue
  <template>
    <li @click="selectVideo">
      <img :src="youtubeImgSrc" alt="#">
      {{ stringUnescape(video.snippet.title) }}
    </li>
  </template>
  
  <script>
  import _ from 'lodash'
  
  export default {
    name: 'VideoListItem',
    props: {
      video: {
        type: Object,
      }
    },
    computed: {
      youtubeImgSrc: function () {
        return this.video.snippet.thumbnails.default.url
      },
      stringUnescape() {
        return (rawText) => {
          return _.unescape(rawText)
        }
      }
    },
  }
  </script>
  
  <style>
  
  </style>
  ```



- Data를 emit으로 끌어 올려줍니다.

```vue
// VideoListItem

<template>
  <li @click="selectVideo">
    <img :src="youtubeImgSrc" alt="#">
    {{ stringUnescape(video.snippet.title) }}
  </li>
</template>

<script>
import _ from 'lodash'

export default {
  name: 'VideoListItem',
  props: {
    video: {
      type: Object,
    }
  },
  methods: {
    selectVideo: function () {
      this.$emit('select-video', this.video)
    }
  },
  computed: {
    youtubeImgSrc: function () {
      return this.video.snippet.thumbnails.default.url
    },
    stringUnescape() {
      return (rawText) => {
        return _.unescape(rawText)
      }
    }
  },
  // Disallow using deprecated filters syntax (in Vue.js 3.0.0+)
  // filters: {
  //   stringUnescape: function (rawText) {
  //     return _.unescape(rawText)
  //   }
  // },
}
</script>

<style>

</style>
```

```vue
// VideoList

<template>
  <ul>
    <VideoListItem 
      v-for="(video, index) in videos" 
      :key="index"
      :video="video"
      @select-video="onSelectVideo"
    />
  </ul>
</template>

<script>
import VideoListItem from '@/components/VideoListItem.vue'

export default {
  name: 'VideoList',
  components: {
    VideoListItem,
  },
  props: {
    // 상위에서 받은(props) 데이터를 상세하게 적습니다.
    videos: {
      type: Array,
    }
  },
  methods: {
    onSelectVideo: function (video) {
      console.log(video)
      this.$emit('select-video', video)
    }
  }
}
</script>

<style>

</style>
```

```vue
// app

<template>
  <div id="app">
    <h1>My Youtube Project</h1>
    <SearchBar @input-search="onInputSearch"/>
    <!-- 왼쪽은 props를 받을 곳의 이름 // 오른쪽은 여기서 보내는 데이터 값 -->
    <VideoList 
      :videos="videos"
      @select-video="onSelectVideo"
    />
  </div>
</template>

<script>
import axios from 'axios'
import SearchBar from '@/components/SearchBar.vue'
import VideoList from '@/components/VideoList.vue'


const API_KEY = process.env.VUE_APP_YOUTUBE_API_KEY
const API_URL = 'https://www.googleapis.com/youtube/v3/search'

export default {
  name: 'App',
  components: {
    SearchBar,
    VideoList,
  },
  data: function () {
    return {
      searchData: '',
      videos: [],
      selectVideo: '',
    }
  },
  methods: {
    onSelectVideo: function (video) {
      console.log(video)
      this.selectVideo = video
    },
    onInputSearch: function (searchData) {
      // console.log('searchbar의 이벤트가 감지되었습니다!')
      // console.log(searchData)
      this.searchData = searchData

      const params = {
        key: API_KEY,
        part: 'snippet',
        type: 'video',
        q: this.searchData,
      }

      axios({
        url: API_URL,
        method: 'get',
        params,
      })
        .then(response => {
          // console.log(response)
          console.log(response.data.items)
          this.videos = response.data.items
        })
        .catch(error => {
          console.log(error)
        })
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

```

​	

### 2.3. VideoDetail

​	

```vue
// app

<VideoDetail :video="selectVideo"/>

import VideoDetail from '@/components/VideoDetail.vue'

VideoDetail
```

```vue
// VideoDetail
<template>
  <div style="text-align: center;">
    <iframe :src="videoUrl" frameborder="0"></iframe>
  </div>
</template>

<script>
export default {
  name: 'VideoDetail',
  props: {
    video: {
      // 처음 데이터가 없는 상황일때
      // [] empty String으로 가지고 있다가 -> Data가 들어오면 Object으로 받음
      type: [String, Object],
    }
  },
  computed: {
    videoUrl: function () {
      const videoId = this.video.id.videoId
      return `https://www.youtube.com/embed/${videoId}`
    }
  }
}
</script>

<style>

</style>
```

- 예외처리 : 처음 렌더링에서 video 데이터가 없는 경우
- 검색결과 가장 처음 데이터를 객체에 넣어줍니다.

```vue
<template>
  <div v-if="video" style="text-align: center; margin-top: 2rem;">
    <iframe :src="videoUrl" frameborder="0"></iframe>
    <h2>{{ video.snippet.title | stringUnescape }}</h2>
    <p>{{ video.snippet.description | stringUnescape }}</p>
  </div>
</template>

<script>
import _ from 'lodash'

export default {
  name: 'VideoDetail',
  props: {
    video: {
      // 처음 데이터가 없는 상황일때
      // [] empty String으로 가지고 있다가 -> Data가 들어오면 Object으로 받음
      type: [String, Object],
    }
  },
  computed: {
    videoUrl: function () {
      const videoId = this.video.id.videoId
      return `https://www.youtube.com/embed/${videoId}`
    }
  },
  filters: {
    stringUnescape: function (rawText) {
      return _.unescape(rawText)
    }
  },
}
</script>

<style>

</style>
```



### 2.4. CSS

```vue
// App
<template>
  <div id="app">
    <h1>Vue-Youtube Porject</h1>
    <header>
      <SearchBar
        @input-search="onInputSearch"
      />
    </header>
    <section>
      <VideoDetail 
        :video="selectVideo"
      />
      <VideoList
        :videos="videos"
        @select-video="onSelectVideo"
      />
    </section>
  </div>
</template>

<script>
import axios from 'axios'
import SearchBar from "@/components/SearchBar.vue";
import VideoList from "@/components/VideoList.vue";
import VideoDetail from "@/components/VideoDetail.vue";


const API_KEY = process.env.VUE_APP_YOUBUE_API_KEY
const API_URL = 'https://www.googleapis.com/youtube/v3/search'

export default {
  name: "App",
  components: {
    SearchBar,
    VideoList,
    VideoDetail,
  },

  data: function () {
    return {
      searchData: '',
      videos: [],
      selectVideo: '',
    }
  },
  methods: {
    // VideoListItem에서 emit으로 올라온 "선택된 비디오" 정보입니다.
    onSelectVideo: function (video) {
      this.selectVideo = video
    },

    // Youtube Api를 통한 검색 결과 반환
    onInputSearch: function (searchData) {
      this.searchData = searchData

      const params = {
        key: API_KEY,
        part: 'snippet',
        type: 'video',
        q: this.searchData,
      }
      axios({
        url: API_URL,
        methods: 'get',
        params
      })
        .then(response => {
          this.videos = response.data.items
        })
        .catch(error => {
          console.log(error)
        })
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  padding-left: 1rem;
  padding-right: 1rem;
}

section,
header {
  width: 80%;
  margin: 0 auto;
  padding: 1rem 0;
}

section {
  display: flex;
}
</style>
```

```vue
// SearchBar

<template>
  <div class="search-bar">
    <input type="text" @keyup.enter="onInputKeyword">
  </div>
</template>

<script>
export default {
  name: "SearchBar",
  methods: {
    onInputKeyword: function (event) {
      // console.log(event.target.value)
      this.$emit('input-search', event.target.value)
    }
  }

}
</script>

<style>
.search-bar > input {
  width: 100%;
  padding: 0.5rem;
  font-size: 2rem;
}

</style>
```

```vue
// VideoList

<template>
  <ul class="video-list list-group">
    <VideoListItem
      v-for="(video, index) in videos"
      :key="index"
      :video="video"
      @select-video="onSelectVideo"
    />
  </ul>
</template>

<script>
import VideoListItem from '@/components/VideoListItem.vue'

export default {
  name: 'VideoList',
  components: {
    VideoListItem,
  },
  props: {
    videos: {
      type: Array,
    }
  },
  methods: {
    onSelectVideo: function (video) {
      console.log(video)
      this.$emit('select-video', video)
    }
  }
}
</script>

<style>
.video-list {
  padding: 0;
  margin: 0;
  list-style-type: none;
  width: 30%;
}


</style>
```

```vue
// VideoListItem

<template>
  <li @click="selectVideo" class="list-group-item">
    <img :src="youtubeImgSrc" alt="#">
    {{ stringUnescape(video.snippet.title) }}
  </li>
</template>

<script>
import _ from 'lodash'

export default {
  name: 'VideoListItem',
  props: {
    video: {
      type: Object,
    }
  },
  methods: {
    selectVideo: function () {
      this.$emit('select-video', this.video)
    }
  },
  computed: {
    youtubeImgSrc: function () {
      return this.video.snippet.thumbnails.default.url
    },
    stringUnescape() {
      return (rawText) => {
        return _.unescape(rawText)
      }
    }
  },
}
</script>

<style>
.list-group .list-group-item {
  display: flex;
  margin-bottom: 1rem;
  cursor: pointer;
}

.list-group .list-group-item:hover {
  background: #eee;
}

.list-group .list-group-item img {
  height: fit-content;
  margin-right: 0.5rem;
}

</style>

```

```vue
// VideoDetail

<template>
  <div v-if="video" class="video-detail">
    <div class="video-container">
      <iframe :src="videoUrl" frameborder="0"></iframe>
    </div>
    <h2>{{ stringUnescape(video.snippet.title) }}</h2>
    <p>{{ stringUnescape(video.snippet.description) }}</p>
  </div>
</template>

<script>
import _ from 'lodash'

export default {
  name: 'VideoDetail',
  props: {
    video: {
      type: [String, Object],
    }
  },
  computed: {
    videoUrl: function () {
      const videoId = this.video.id.videoId
      return `https://www.youtube.com/embed/${videoId}`
    },
    stringUnescape() {
      return (rawText) => {
        return _.unescape(rawText)
      }
    }
  }
}
</script>

<style>
.video-detail {
  width: 70%;
  padding-right: 1rem;
}

.video-container {
  position: relative;
  padding-top: 56.25%;
}

.video-container > iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>
```

#### ✨RESULT.png

![image-20210711175019789](README.assets/image-20210711175019789.png)



#### 🤦‍♂️remove build-test 

```
// deply.yml

      # - name: Test unit
      #   run: yarn test:unit


// test\unit 
// import { shallowMount } from "@vue/test-utils";
// import HelloWorld from "@/components/HelloWorld.vue";

// describe("HelloWorld.vue", () => {
//   it("renders props.msg when passed", () => {
//     const msg = "new message";
//     const wrapper = shallowMount(HelloWorld, {
//       props: { msg },
//     });
//     expect(wrapper.text()).toMatch(msg);
//   });
// });
```

