<template>
  <!--图片展示-->
  <div class="weibo-media" v-if="card.card_info.big_card.object_type=='picture'">
    <!--四宫格\九宫图-->
    <ul class="m-auto-list" :class="{'m-auto-list4':card.card_info.big_card.object.small_pics.length==4}" v-if="card.card_info.big_card.object.small_pics.length >1">
      <li class="m-auto-box" v-for="(pic, index) in card.card_info.big_card.object.small_pics">
        <div class="m-img-box" @click="thumbnails($event, index, card.card_info.big_card.object.large_pics)">
          <mv-img :needlazy="true" :src="pic.url"></mv-img>
          <span class="feed-mark" v-if="pic.pic_type=='gif'">GIF</span>
          <span class="feed-mark" v-else-if="pic.pic_type=='long'">长图</span>
          <span class="feed-mark" v-else-if="pic.pic_type=='live'">LIVE</span>
        </div>
      </li>
    </ul>
    <div v-else-if="card.card_info.big_card.object.small_pics.length == 1" @click="thumbnails($event, 0, card.card_info.big_card.object.large_pics)">
      <div class="single-img gif" v-if="card.card_info.big_card.object.small_pics[0].pic_type=='gif'">
        <mv-img :needlazy="true" :src="card.card_info.big_card.object.small_pics[0].url"></mv-img>
        <span class="feed-mark">GIF</span>
      </div>
      <div class="single-img long" v-else-if="card.card_info.big_card.object.small_pics[0].pic_type=='long'">
        <mv-img :needlazy="true" :src="card.card_info.big_card.object.small_pics[0].url"></mv-img>
        <span class="feed-mark">长图</span>
      </div>
      <div class="single-img live" v-else-if="card.card_info.big_card.object.small_pics[0].pic_type=='live'">
        <mv-img :needlazy="true" :src="card.card_info.big_card.object.small_pics[0].url"></mv-img>
        <span class="feed-mark">LIVE</span>
      </div>
      <div class="single-img" v-else>
        <mv-img :needlazy="true" :src="card.card_info.big_card.object.small_pics[0].url"></mv-img>
      </div>
    </div>
  </div>
  <!--视频展示-->
  <div class="weibo-media" v-else-if="card.card_info.big_card.object_type=='video'">
    <weibo-video :card='card'></weibo-video>
  </div>
  <!--图片文章 推荐-->
  <div class="weibo-media" v-else-if="card.card_info.big_card.object_type=='article'">
    <page-article :card="card.card_info.big_card"></page-article>
  </div>

</template>
<style lang="scss" scoped="scoped">
	@import "../../scss/_sassCore";
	@import "../../scss/_var";
  .weibo-media{
    /*gif、long、Live小图标*/
    margin-top: 0.15rem;
    .feed-mark{
      position: absolute;
      right: 0;
      bottom: 0;
      border-radius: 0.04rem 0 0 0;
      color: #fff;
      background-color: rgba(80,125,175,0.75);
      font-size: 0.2rem;
    }
    /*九宫图*/
    .m-auto-list{
      max-width: 400px;
      overflow: hidden;
      .m-img-box{
        position: relative;
        padding-bottom: 100%;
        height: 100%;
        width: 100%;
        img{
          width: 100%;
          height: 100%;
          position: absolute;
          z-index: 0;
          vertical-align: top;
          object-fit: cover;
        }

      }
      .m-auto-box{
        display: inline-block;
        width: 33.33%;
        vertical-align: top;
        padding-left: 0.04rem;
        padding-right: 0.04rem;
        margin-bottom: 0.08rem;

      }
    }
    .m-auto-list.m-auto-list4{
      max-width: 664px;
      .m-auto-box{
        width:auto;
        padding: 0;
        margin-bottom: 0.1rem;
        margin-right: 0.1rem;
        .m-img-box{
          position: relative;
          width: 2.26rem;
          height: 2.26rem;
        }
      }
    }
    /*单张图*/
    .single-img {
      position: relative;
      width: 50%;
      height: 0;
      padding-bottom: 50%;
      overflow: hidden;
      .gif {
        width: 61%;
        padding-bottom: 61%;
      }
      .long{
        width: 46%;
        padding-bottom: 60%;
      }
      .live{
        width: 61%;
        padding-bottom: 45.6%;
      }
      >img{
        width: 100%;
        /*display: block;*/
      }
    }


  }

</style>
<script>
	import Bus from '../../marvel/bus';
	export default {
		data() {
			return {
				showMark: false,
			};
		},
		props: ['card'],
		computed: {
			// 多图情况下的布局模式。4张图为四宫格(type=a),其他为九宫格(type=b)。
			type() {
				if(this.card.card_info.big_card.object.small_pics && this.card.card_info.big_card.object.small_pics.length === 4) {
					return 'a';
				}
				return 'b';
			},
			styles: function() {
				const W = 150;
				const w = this.card.pics[0].geo.width;
				const h = this.card.pics[0].geo.height;
				if(h / w > 16 / 9) {
					this.showMark = true;
				}
				return {
					singleImg: {
						width: w > W ? W / 16 + 'rem' : w / 16 + 'rem',
						height: w > W ? ((W * h / w) / 16) + 'rem' : h / 16 + 'rem',
					}
				};
			}
		},
		components: {
			'weibo-video': require('./weibo-video.vue'),
			'page-article': require('./page-article.vue'),
		},
		methods: {
			thumbnails(e, index, cardList) {
				const eTarget = e.target || e.srcElement;
				const list = this.formatThumbItem(eTarget, cardList, index);
				Bus.$emit('moviePic', index, list);
			},
			// 拼出pswp需要的数据格式
			formatThumbItem(target, cardList, index) {
				// 找出父结点
				const clickedListItem = this.closest(target, (el) => {
					return el.classList.contains('weibo-media');
				});

				const cards = cardList.map((card, i) => {
					if(!card.width) {
						const style = window.getComputedStyle ? window.getComputedStyle(target, null) : null || target.currentStyle;
						const windowWidth = document.documentElement.clientWidth || window.innerWidth;
						card.width = windowWidth;
						card.height = parseFloat(style.height) / parseFloat(style.width) * windowWidth;
					}
					return {
						src: card.url, // 大图
						w: card.width, // 大图宽度
						h: card.height, // 大图高度
						msrc: card.url, // 缩略图
						el: clickedListItem.getElementsByTagName('img')[i], // 缩略图对应的img DOM
					};
				});
				return cards;
			},
			closest(el, fn) {
				return el && (fn(el) ? el : this.closest(el.parentNode, fn));
			}
		},
	};
</script>
