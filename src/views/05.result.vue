<template>
  <div class="result-container">
    <div class="title-wrap">
      <!-- //顶部的标题 -->
      <h2 class="title">{{ $route.query.q }}</h2>
      <span class="sub-title">找到{{ count }}个结果</span>
    </div>
    <el-tabs v-model="activeIndex">
      <el-tab-pane label="歌曲" name="songs">
        <table class="el-table">
          <thead>
            <th></th>
            <th>音乐标题</th>
            <th>歌手</th>
            <th>专辑</th>
            <th>时长</th>
          </thead>
          <tbody>
            <tr v-for="(item, index) in songList" :key="index" clss="el-table__row" @dblclick="playMusic(item.id)">
              <td>{{ index + 1 }}</td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <!-- 歌名 -->
                    <span>{{ item.name }}</span>
                    <!-- mv图标 -->
                    <span v-if="item.mvid" class="iconfont icon-mv"></span>
                  </div>
                  <!-- 二级标题 -->
                  <span v-if="item.alias.length != 0">{{ item.alias[0] }}</span>
                </div>
              </td>
              <td>{{ item.artists[0].name }}</td>
              <td>{{ item.album.name }}</td>
              <td>{{ item.duration }}</td>
            </tr>


          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane label="歌单" name="lists">
        <div class="items">
          <div v-for="(item, index) in playlists" :key="index" class="item" @click="toPlaylist(item.id)">
            <div class="img-wrap">
              <div class="num-wrap">
                播放量:
                <span class="num">{{ item.playCount }}</span>
              </div>
              <img :src="item.coverImgUrl" alt="" />
              <span class="iconfont icon-play"></span>
            </div>
            <p class="name">{{ item.name }}</p>
          </div>
        </div>
      </el-tab-pane>
      <el-tab-pane label="MV" name="mv">
        <div class="items mv">
          <div v-for="(item, index) in mv" :key="index" class="item" @click="toMv(item.id)">
            <div class="img-wrap">
              <img :src="item.cover" alt="" />
              <span class="iconfont icon-play"></span>
              <div class="num-wrap">
                <div class="iconfont icon-play"></div>
                <div class="num">{{ item.playCount }}</div>
              </div>
              <span class="time">{{ item.duration }}</span>
            </div>
            <div class="info-wrap">
              <div class="name">{{ item.name }}</div>
              <div class="singer">{{ item.artistName }}</div>
            </div>
          </div>
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'result',
  data() {
    return {
      activeIndex: 'songs',
      //保存 查询数据
      songList: [],
      //mv
      mv: [],
      //歌单
      playlists: [],
      count: 0
    };
  },
  created() {
    axios({
      url: "http://47.108.189.41:3000/search",
      method: "get",
      params: {
        type: 1,
        keywords: this.$route.query.q,
        limit: 10
      },
    }).then((res) => {
      // console.log(res),
      this.songList = res.data.result.songs
      //计算歌曲时间
      for (let i = 0; i < this.songList.length; i++) {
        let min = parseInt(this.songList[i].duration / 1000 / 60)
        let sec = parseInt(this.songList[i].duration / 1000 % 60)
        if (min < 10) {
          min = '0' + min

        }
        if (sec < 10) {
          sec = '0' + sec

        }
        this.songList[i].duration = min + ':' + sec

      }
      //总条数
      this.count = res.data.result.songCount
    });
  },
  watch: {
    activeIndex() {
      //song 歌曲
      //lists 歌单
      // mv mv
      let type = 1;
      let limit = 10
      switch (this.activeIndex) {
        case 'songs':
          type = 1;
          limit = 10
          break;
        case 'lists':
          type = 1000;
          limit = 10
          break;
        case 'mv':
          type = 1004;
          limit = 8
          break;
        default:
          break;

      }
      axios({
        url: "http://47.108.189.41:3000/search",
        method: "get",
        params: {
          type,
          keywords: this.$route.query.q,
          limit
        },
      }).then(res => {
        // console.log(res)
        if (type == 1) {
          //歌曲
          this.songList = res.data.result.songs
          //计算歌曲时间
          for (let i = 0; i < this.songList.length; i++) {
            let min = parseInt(this.songList[i].duration / 1000 / 60)
            let sec = parseInt(this.songList[i].duration / 1000 % 60)
            if (min < 10) {
              min = '0' + min

            }
            if (sec < 10) {
              sec = '0' + sec

            }
            this.songList[i].duration = min + ':' + sec

          }
          //总条数
          this.count = res.data.result.songCount
        } else if (type == 1000) {
          this.playlists = res.data.result.playlists
          this.count = res.data.result.playlistCount
          //播放次数
          for (let i = 0; i < this.playlists.length; i++) {
            if (this.playlists[i].playCount > 100000) {
              this.playlists[i].playCount = parseInt(this.playlists[i].playCount / 10000) + '万'
            }
          };

        }
        else {
          //保存数据
          this.mv = res.data.result.mvs
          //总数
          this.count = res.data.result.mvCount
          //处理数据
          for (let i = 0; i < this.mv.length; i++) {
            //总时长
            let min = parseInt(this.mv[i].duration / 1000 / 60)
            if (min < 10) {
              min = '0' + min

            }
            let sec = parseInt(this.mv[i].duration / 1000 % 60)
            if (sec < 10) {
              sec = '0' + sec

            }
            this.lists[i].duration = `${min}:${sec}`
            //播放次数
            if (this.mv[i].playCount > 100000) {
              this.mv[i].playCount = parseInt(this.mv[i].playCount / 10000) + '万'
            }
          }
        }

      })

    }
  },
  methods: {
    //跳转到mv详情页
  toMv(id){
this.$router.push(`/mv?q=${id}`)
  },
    //去歌单详情页
    toPlaylist(id) {
      //跳转并携带id
      this.$router.push(`/playlist?q=${id}`)
    },
    playMusic(id) {
      // console.log(id)
      axios({
        url: "http://47.108.189.41:3000/song/url",
        method: "get",
        params: {
          id
        },
      }).then((res) => {
        let url = res.data.data[0].url
        // console.log(this.$parent.musicUrl);
        this.$parent.musicUrl = url
      });
    }
  }
}
</script>

<style >
</style>
