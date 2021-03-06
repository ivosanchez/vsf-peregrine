<template>
  <div id="home">
    <div v-if="cmsHomeComponents">
      <ComponentRenderer :cmsObject="cmsHomeComponents"/>
      <Onboard />
    </div>
  </div>
</template>

<script>
// query constructor
import { prepareQuery } from '@vue-storefront/core/modules/catalog/queries/common'
import { isServer, onlineHelper } from '@vue-storefront/core/helpers'

// Core pages
import Home from '@vue-storefront/core/pages/Home'
// Theme local components
import Onboard from 'theme/components/theme/blocks/Home/Onboard'
import TileLinks from 'theme/components/theme/blocks/TileLinks/TileLinks'
import { Logger } from '@vue-storefront/core/lib/logger'
import { mapGetters } from 'vuex'
import config from 'config'
import ComponentRenderer from '../components/ComponentRenderer'

export default {
  mixins: [Home],
  components: {
    Onboard,
    TileLinks,
    ComponentRenderer
  },
  computed: {
    ...mapGetters(
      {cmsHomeComponents: 'cmsstore/getHomeComponents'},
      'user',
      ['isLoggedIn']

    ),
    categories () {
      return this.getCategories
    },
    everythingNewCollection () {
      return this.$store.state.homepage.new_collection
    },
    coolBagsCollection () {
      return this.$store.state.homepage.coolbags_collection
    },
    isOnline () {
      return onlineHelper.isOnline
    }
  },
   async created () {
    // Load personal and shipping details for Checkout page from IndexedDB
    this.$store.dispatch('checkout/load')
    await this.$store.dispatch('cmsstore/getCmsHomeComponents')
  },
  async beforeMount () {
    if (this.$store.state.__DEMO_MODE__) {
      const onboardingClaim = await this.$store.dispatch('claims/check', { claimCode: 'onboardingAccepted' })
      if (!onboardingClaim) { // show onboarding info
        this.$bus.$emit('modal-toggle', 'modal-onboard')
        this.$store.dispatch('claims/set', { claimCode: 'onboardingAccepted', value: true })
      }
    }
  },
  async mounted () {
    if (!this.isLoggedIn && localStorage.getItem('redirect')) this.$bus.$emit('modal-show', 'modal-signup')
    // todo: this.cmsHomeComponents.components.push({type: 'product', data: {}})
  },
  data () {
    return {
      CMSComponent: {}
    }
  },
  watch: {
    isLoggedIn () {
      const redirectObj = localStorage.getItem('redirect')
      if (redirectObj) this.$router.push(redirectObj)
      localStorage.removeItem('redirect')
    }
  },
  async asyncData ({ store, route }) { // this is for SSR purposes to prefetch data
    Logger.info('Calling asyncData in Home (theme)')()

    let newProductsQuery = prepareQuery({ queryConfig: 'newProducts' })
    let coolBagsQuery = prepareQuery({ queryConfig: 'coolBags' })

    const newProductsResult = await store.dispatch('product/list', {
      query: newProductsQuery,
      size: 8,
      sort: 'created_at:desc'
    })
    if (newProductsResult) {
      store.state.homepage.new_collection = newProductsResult.items
    }

    const coolBagsResult = await store.dispatch('product/list', {
      query: coolBagsQuery,
      size: 4,
      sort: 'created_at:desc',
      includeFields: config.entities.optimize ? (config.products.setFirstVarianAsDefaultInURL ? config.entities.productListWithChildren.includeFields : config.entities.productList.includeFields) : []
    })
    if (coolBagsResult) {
      store.state.homepage.coolbags_collection = coolBagsResult.items
    }

    await store.dispatch('promoted/updateHeadImage')
    await store.dispatch('promoted/updatePromotedOffers')
  },
  beforeRouteEnter (to, from, next) {
    if (!isServer && !from.name) { // Loading products to cache on SSR render
      next(vm => {
        let newProductsQuery = prepareQuery({ queryConfig: 'newProducts' })
        vm.$store.dispatch('product/list', {
          query: newProductsQuery,
          size: 8,
          sort: 'created_at:desc'
        })
      })
    } else {
      next()
    }
  }
}
</script>

<style lang="scss" scoped>
  .new-collection {
    @media (max-width: 767px) {
      padding-top: 0;
    }
  }
  #home {
        min-height: 100vh !important;
  }
</style>
