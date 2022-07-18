<template>
  <div class="discovery-container">
    <!-- 轮播图 -->
    <el-carousel class="" :interval="4000" type="card">
      <el-carousel-item v-for="(item,index) in banners"  :key="index">
        <img :src="item.imageUrl" alt="" />
      </el-carousel-item>
    </el-carousel>
    <!-- 推荐歌单 -->
    <div class="recommend">
      <h3 class="title">
        推荐歌单
      </h3>
      <div class="items">
        <div class="item" v-for="(item,index) in personalized" :key="index" @click="toPlaylist(item.id)">
          <div class="img-wrap">
            <!-- <div class="desc-wrap">
              <span class="desc">{{item}}}</span>
            </div> -->
            <img :src="item.picUrl" alt="" />
            <span class="iconfont icon-play" ></span>
          </div>
          <p class="name">{{item.name}}</p>
        </div>
      </div>
    </div>
    <!-- 最新音乐 -->
    <div class="news">
      <h3 class="title">
        最新音乐
      </h3>
      <div class="items">
        <div class="item" v-for="(item,index) in newsong" :key="index">
          <div class="img-wrap">
            <img :src="item.picUrl" alt="" />
            <span @click="playMusic(item.id)" class="iconfont icon-play"></span>
          </div>
          <div class="song-wrap">
            <div class="song-name">{{item.name}}</div>
            <div class="singer">{{item.song.artists[0].name}}</div>
          </div>
        </div>

      </div>
    </div>
    <!-- 推荐MV -->
    <div class="mvs">
      <h3 class="title">推荐MV</h3>
      <div class="items">
        <div class="item" v-for="(item,index) in mv" :key="index" @click="toMv(item.id)">
          <div class="img-wrap">
            <img :src="item.picUrl" alt="" />
            <span class="iconfont icon-play"></span>
            <div class="num-wrap">
              <div class="iconfont icon-play"></div>
              <div class="num">{{item.playCount}}</div>
            </div>
          </div>
          <div class="info-wrap">
            <div class="name">{{item.name}}</div>
            <div class="singer">{{item.artistsName}}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'discovery',
  data() {
    return {
      banners:[],
      //推荐歌单
      personalized:[],
      //最新音乐
      newsong:[],
      mv:[],
      songlist:[]
    }
  },
  created() {
    //轮播图接口
    axios({
    url: "http://47.108.189.41:3000/banner",
    method: "get",
    }).then((res) => {
this.banners=res.data.banners
    });
    //推荐歌单
    axios({
    url: "http://47.108.189.41:3000/personalized",
    method: "get",
    params: {
    limit: 10,
    },
    }).then((res) => {
      this.personalized=res.data.result
    });
    //推荐音乐
    axios({
    url: "http://47.108.189.41:3000/personalized/newsong",
    method: "get",
    params: {
    limit: 10,
    },
    }).then((res) => {
      this.newsong=res.data.result
    });
    //推荐mv
    axios({
    url: "http://47.108.189.41:3000/personalized/mv",
    method: "get",
    // params: {
    // limit: 10,
    // },
    }).then((res) => {
      // console.log(res);
      this.mv=res.data.result
    });
  
   
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
    playMusic(id){
     axios({
    url: "http://47.108.189.41:3000/song/url",
    method: "get",
    params: {
    id
    },
    }).then((res) => {
      let url=res.data.data[0].url
      // console.log(this.$parent.musicUrl);
      this.$parent.musicUrl=url
    });
    }
  },
};

</script>

<style >

</style>
