<template>
  <div class="playlist-container">
    <div class="top-wrap">
      <div class="img-wrap">
        <!-- 封面 -->
        <img :src="playlist.coverImgUrl" alt="" />
      </div>
      <div class="info-wrap">
        <!-- 名字 -->
        <p class="title">{{ playlist.name }}</p>
        <div class="author-wrap">
          <!-- 头像 -->
          <img class="avatar" :src="playlist.coverImgUrl" alt="" />
          <span class="name">{{ playlist.creator.nickname }}</span>
          <span class="time">{{ playlist.createTime }}创建</span>
        </div>
        <div class="play-wrap">
          <span class="iconfont icon-circle-play"></span>
          <span class="text">播放全部</span>
        </div>
        <div class="tag-wrap">
          <span class="title">标签:</span>
          <ul>
            <li v-for="(item, index) in playlist.tags" :key="index">{{ item }}</li>
          </ul>
        </div>
        <div class="desc-wrap">
          <span class="title">简介:</span>
          <span class="desc">{{ playlist.description }}</span>
        </div>
      </div>
    </div>
    <el-tabs v-model="activeIndex">
      <el-tab-pane label="歌曲列表" name="1">
        <table class="el-table playlit-table">
          <thead>
            <th></th>
            <th></th>
            <th>音乐标题</th>
            <th>歌手</th>
            <th>专辑</th>
            <th>时长</th>
          </thead>
          <tbody>
            <tr class="el-table__row" v-for="(item, index) in songLists" :key="index">
              <td>{{ index + 1 }}</td>
              <td>
                <div class="img-wrap" >
                  <img :src="item.al.picUrl" alt="" />
                  <span  @click="playMusic(item.id)" class="iconfont icon-play"></span>
                </div>
              </td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <span>{{item.name}}</span>
                    <span class="iconfont icon-mv"></span>
                  </div>
                  <span>{{item.al.name}}</span>
                </div>
              </td>
              <td>{{item.ar['0'].name}}</td>
              <td>{{item.al.name}}</td>
              <td>{{ item.dt }}</td>
            </tr>
           
          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane label="评论" name="2">
        <!-- 精彩评论 -->
        <div class="comment-wrap">
          <p class="title">精彩评论<span class="number">({{hotCount}})</span></p>
          <div class="comments-wrap">
            <!-- 评论 -->
            <div v-for="(item,index) in hotComment" :key="index" class="item">
              <div class="icon-wrap">
                <!-- 头像 -->
                <img :src="item.user.avatarUrl" alt="" />
              </div>
              <div class="content-wrap">
                <div class="content">
                  <!-- 昵称 -->
                  <span class="name">{{item.user.nickname}}</span>
                  <span class="comment">{{item.content}}}</span>
                </div>
                <!-- 评论的回复 -->
                <div  v-if="item.beReplied.length!=0" class="re-content" >
                  <span class="name">{{item.beReplied[0].user.nickname}}</span>
                  <span class="comment">{{item.beReplied[0].content}}</span>
                </div>
                <div class="date">2020-02-12 17:26:11</div>
              </div>
            </div>
          </div>
        </div>
        <!-- 最新评论 -->
        <div class="comment-wrap">
          <p class="title">最新评论<span class="number">({{total}})</span></p>
          <div class="comments-wrap">
            <div class="item" v-for="(item,index) in comments" :key="index">
              <div class="icon-wrap">
                <img :src="item.user.avatarUrl" alt="" />
              </div>
              <div class="content-wrap">
                <div class="content">
                  <span class="name">{{item.user.nickname}}</span>
                  <span class="comment">{{item.content}}</span>
                </div>
                <div class="re-content" v-if="item.beReplied.length!=0">
                  <span class="name">{{item.beReplied[0].user.nickname}}</span>
                  <span class="comment">{{item.beReplied[0].content}}</span>
                </div>
                <div class="date">2020-02-12 17:26:11</div>
              </div>
            </div>
          </div>
        </div>
        <!-- 分页器 -->
        <el-pagination @current-change="handleCurrentChange" background layout="prev, pager, next" :total="total"
          :current-page="page" >
        </el-pagination>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'playlist',
  data() {
    return {
      activeIndex: '1',
      // 总条数
      total: 0,
      // 页码
      page: 1,
      //歌曲列表
      playlist: {},
      //热门评论
      hotComment:[],
      //热门评论总数
      hotCount:0,
      //普通评论
      comments:[],
      songLists: []



    };
  },
  created() {
    //获取歌单歌曲
    axios({
    url: "http://47.108.189.41:3000/playlist/track/all",
    method: "get",
    params: {
    id:this.$route.query.q
    },
    }).then((res) => {
      console.log(res);
      this.songLists=res.data.songs
      for (let i = 0; i < this.songLists.length; i++) {
          //总时长
          let dt = this.songLists[i].dt
          let min = parseInt(dt / 1000 / 60)
          if (min < 10) {
            min = '0' + min

          }
          let sec = parseInt(dt / 1000 % 60)
          if (sec < 10) {
            sec = '0' + sec

          }
          this.songLists[i].dt = `${min}:${sec}`
        }
    
    });
    //获取歌单详情
    axios({
      url: "http://47.108.189.41:3000/playlist/detail",
      method: "get",
      params: {
        id: this.$route.query.q
      },
    }).then(res => {
      // console.log(res);
      this.playlist = res.data.playlist
    });
    //获取歌单评论hot
  axios({
  url: "http://47.108.189.41:3000/comment/hot",
  method: "get",
  params: {
  id:this.$route.query.q,
  //传递类型
  type:2
  },
  }).then(res => {
    // console.log(res);
    this.hotComment=res.data.hotComments
    this.hotCount=res.data.total
  });
  //获取最新评论
  axios({
  url: "http://47.108.189.41:3000/comment/playlist",
  method: "get",
  params: {
   id:this.$route.query.q,
   limit:10,
   offset:0
  },
  }).then((res) => {
    //总个数
    this.total=res.data.total
    //评论数据
  this.comments=res.data.comments
  });
  },
  methods: {
     //播放歌曲
    //播放歌曲
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
    },
    handleCurrentChange(val) {
      //保存页码
      this.page=val
      axios({
  url: "http://47.108.189.41:3000/comment/playlist",
  method: "get",
  params: {
   id:this.$route.query.q,
   limit:10,
   offset:(this.page-1)*5
  },
  }).then((res) => {
    //总个数
    this.total=res.data.total
    //评论数据
  this.comments=res.data.comments
  });
    }
  }
};
</script>

<style >
</style>
