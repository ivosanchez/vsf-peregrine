<template>
<no-ssr v-if="componentData">
  <section class="head-image w-100 bg-cl-th-accent cl-white pointer" @click="link(componentData.data)" >
    <div class="container w-100 h-100 cl-black" v-lazy:background-image="banner_image" v-if="banner_image">
      <div class="head-image-content">
        <h1 class="title" data-testid="mainSliderTitle" v-html="banner_title">
          {{ banner_title }}
        </h1>
        <p
          class="subtitle mb0 serif h3"
          data-testid="mainSliderSubtitle" v-html="banner_subtitle"
        >
          {{ banner_subtitle }}
        </p>
        <!-- <div class="align-center inline-flex">
          <button-outline :link="currentImage.link" color="light">
            {{ currentImage.button_text }}
          </button-outline>
        </div> -->
      </div>
    </div>
  </section>
</no-ssr>
</template>

<script>
import config from 'config'
import NoSSR from 'vue-no-ssr'
import LinkMixin from '../mixins/LinkMixin'

export default {
  components: {
    // ButtonOutline
    'no-ssr': NoSSR
  },
  props: {
    componentData: {
      type: Object
    }
  },
  data () {
    return {
    }
  },
  computed: {

    banner_image: function () {
      let url = ''
      if(this.componentData['data'].imageLinkType){
      url =  this.componentData['data'].imageLinkType === 'internalLink'? config.cms_peregrine.image_endpoint + this.componentData['data']['bgimage'] : this.componentData['data']['bgimage']
      }
      return url
    },
    banner_title: function () {
      return this.componentData['data']['title'] ? this.componentData['data']['title'] : ''
    },
    banner_subtitle: function () {
      return this.componentData['data']['text'] ? this.componentData['data']['text'] : ''
    }

  },
  mixins:[LinkMixin]
}
</script>

<style lang="scss" scoped>
.head-image {
  display: none;
  @media (min-width: 767px) {
    display: inherit;
  }

  .head-image-content {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    margin-left: 15%;

    .title {
      font-size: 3.4rem;
      margin-bottom: 0;
    }

    .subtitle {
      font-size: 0.8rem;
      max-width: 340px;
      font-family: 'Roboto', sans-serif;
      line-height: 1.2rem;
    }
  }
}

.head-image {
  height: 640px;
}
.container {
  background-size: cover;
  background-position: bottom center;
  background-repeat: no-repeat;
}
.row {
  height: 640px;
}
@media (max-width: 75em) {
  .head-image {
    height: 400px;
  }
  .title {
    font-size: 50px;
  }
  .subtitle {
    font-size: 20px;
  }
  .row {
    height: 400px;
  }
}
@media (max-width: 64em) {
  .head-image {
    height: 359px;
  }
  .container {
    background-position: left;
  }
  .title {
    font-size: 48px;
  }
  .subtitle {
    font-size: 18px;
  }
  .button {
    font-size: 16px;
  }
  .row {
    height: 359px;
  }
}
</style>
