<template>
  <img :src="lazysrc" />
</template>
<script>
  import Bus from '../bus';
  import nonepng from '../../assets/img/none.png';

  var list = []
  var running = false;
  var lazyFuc = function(e) {

    if(running||list.length==0) {
      return;
    }
    running = true;
    var windowHeight = document.documentElement.clientHeight || window.innerHeight;
    var windowWidth = document.documentElement.clientWidth || window.innerWidth;
    var temp = []
    for(var i = 0; i < list.length; i++) {
      var obj = list[i].$el.getBoundingClientRect();
      if((obj.top < windowHeight + 320) && (obj.left < windowWidth)) {
        list[i].isShow = true
      }
      if(!list[i].isShow) {
        temp.push(list[i])
      }
    }
    list = temp;
    running = false;

  }

  window.addEventListener('scroll', lazyFuc);
  window.addEventListener('touchstart', lazyFuc);
  window.addEventListener('touchmove', lazyFuc);
  window.addEventListener('touchend', lazyFuc);


  var cache=[];
  export default{
    name: 'img',
    data: function() {
      return {
        isShow: false
      }
    },
    mounted: function() {
      if(this.isShow){return;}
      if(this.needlazy && !this.isShow) {
        list.push(this);
      }
      var windowHeight = document.documentElement.clientHeight || window.innerHeight;
      var windowWidth = document.documentElement.clientWidth || window.innerWidth;
      var obj = this.$el.getBoundingClientRect();
      if((obj.top < windowHeight) && (obj.left< windowWidth+100)) {
        this.isShow = true;
      }


    },
    computed: {
      lazysrc() {
        if(cache.indexOf(this.src) > -1) {
          return this.src;
        }

        if(!this.needlazy || this.isShow) {
          this.isShow = true;
          var hsrc = this.src;
          cache.push(hsrc);
          return hsrc;
        }else{
          return nonepng;
        }
      }
    },

    props: {
      needlazy: {
        type: Boolean,
        default: true,
      },
      src: {
        type: String,
        default: '',
      },
    }
  };
</script>

