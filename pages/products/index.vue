<template>
  <section v-if="!$fetchState.pending && !$fetchState.error" id="top" class="products">
    <to-top-btn></to-top-btn>
    <tariff-card :dialog="tariffDialog" :items="tariffLists" @change-tariff-dialog="reseiveTariffDialogFlg">
    </tariff-card>
    <search-refinement
      :dialog="dialog"
      :search-category-lists="searchCategoryLists"
      :search-maker-lists="searchMakerLists"
      :search-tag-lists="searchTagLists"
      :search-price-lists="searchPriceLists"
      @change-dialog="reseiveDialogFlg"
      @received-search-conditions="receivedSearchConditions"
      @update-category-list="updateCategoryLists"
      @received-category-reset="receivedCategoryReset"
      @received-all-reset="receivedAllReset">
    </search-refinement>
    <top-bar title="検索結果一覧" :bread-crumbs="breadCrumbs"></top-bar>
    <div class="products__inner d-lg-flex py-16 px-2 px-lg-0">
      <category-lists></category-lists>
      <div class="content ml-lg-15">
        <div class="product__search d-flex">
          <div class="search__number d-flex flex-column align-center justify-center text-center px-2 px-sm-5 py-3">
            <div class="d-flex align-baseline justify-center">
              <span class="number__val letter-space-015em mr-1">{{ searchProductListCount }}</span>
              <span class="text-body-2">件</span>
            </div>
            <div class="text-caption text-sm-body-2 mt-md-1">見つかりました</div>
          </div>
          <div class="search__condition flex-grow-1 pa-3">
            <div class="condition__head text-caption text-md-body-2 d-flex align-center flex-wrap text-no-wrap">
              <span class="text-gray">現在の検索条件</span>
              <v-icon color="#8e8e8e">mdi-chevron-right</v-icon>
              <v-btn class="text-caption text-md-body-2 px-0" color="primary" text @click.stop="getFilterCondition()">
                <v-icon color="primary" class="mr-1">mdi-text-search</v-icon>変更して絞り込む
              </v-btn>
            </div>
            <div class="condition__foot mt-1">
              <ul class="condition__tags d-flex">
                <li
                  v-if="isCategoryName && !conditionalSearchFlg"
                  class="a search-tag d-flex text-caption text-md-body-2 mr-4 mr-lg-5">
                  <v-icon class="mr-2" small>mdi-check</v-icon>{{ $route.query.categoryName }}
                </li>
                <li
                  v-if="isTagName && !conditionalSearchFlg"
                  class="b search-tag d-flex text-caption text-md-body-2 mr-4 mr-lg-5">
                  <v-icon class="mr-2" small>mdi-check</v-icon>{{ $route.query.tagName }}
                </li>
                <li
                  v-if="isKeyword && !conditionalSearchFlg"
                  class="search-tag d-flex text-caption text-md-body-2 mr-4 mr-lg-5">
                  <v-icon class="mr-2" small>mdi-check</v-icon>{{ $route.query.keyword }}
                </li>
                <li
                  v-for="(condition, index) in presentConditions"
                  :key="index"
                  class="search-tag d-flex text-caption text-md-body-2 mr-4 mr-lg-5">
                  <v-icon class="mr-2" small>mdi-check</v-icon>{{ condition }}
                </li>
              </ul>
              <div class="fade-out mr-n3"></div>
            </div>
          </div>
        </div>
        <div class="product__sort d-flex align-center flex-column-reverse flex-lg-row mt-5 mt-lg-4">
          <div class="sort__inner d-flex justify-space-between">
            <div class="sort__release d-flex flex-column flex-lg-row">
              <div
                class="sort__head d-flex align-center justify-center text-white text-caption text-sm-body-2 text-lg-body-1 px-5 py-1">
                リリース日
              </div>
              <v-btn-toggle
                v-model="orderRelease"
                class="d-flex align-center justify-center pa-2 py-lg-0 px-lg-5"
                dense
                tile
                group
                @change="changeOrderRelease()">
                <v-btn text small :value="$config.ORDER_NEW" class="text-md-body-2 ma-0" color="headingText">
                  新着順
                </v-btn>
                <v-divider class="sort__line mx-2" vertical></v-divider>
                <v-btn text small :value="$config.ORDER_OLD" class="text-md-body-2 ma-0" color="headingText">
                  古い順
                </v-btn>
              </v-btn-toggle>
            </div>
            <div class="sort__price d-flex flex-column flex-lg-row ml-lg-3">
              <div
                class="sort__head d-flex align-center justify-center text-white text-caption text-sm-body-2 text-lg-body-1 px-5 py-1">
                価格
              </div>
              <v-btn-toggle
                v-model="orderPrice"
                class="d-flex align-center justify-center pa-2 py-lg-0 px-lg-5"
                dense
                tile
                group
                @change="changeOrderPrice()">
                <v-btn text small :value="$config.ORDER_ASC" class="text-md-body-2 ma-0" color="headingText">
                  低い順
                </v-btn>
                <v-divider class="sort__line mx-2" vertical></v-divider>
                <v-btn text small :value="$config.ORDER_DESC" class="text-md-body-2 ma-0" color="headingText">
                  高い順
                </v-btn>
              </v-btn-toggle>
            </div>
          </div>
          <v-spacer class="hidden-md-and-down"></v-spacer>
          <v-btn
            class="condition__reset mb-8 mb-lg-0"
            color="lightGray"
            outlined
            small
            :disabled="!presentCategoryID"
            @click="resetConditions()">
            カテゴリ以外の条件をリセット
          </v-btn>
        </div>
        <div class="product__main">
          <div v-for="(list, index) in productLists" :key="index">
            <product-card
              :id="list.ProductID"
              :name="list.ProductName"
              :type-number="list.ProductTypeNumber"
              :image="list.ProductImageURL"
              :maker="list.MakerName"
              :category-name01="list.CategoryNmae01"
              :category-name02="list.CategoryNmae02"
              :price-value="list.PriceValue||''"
              :price-type="list.PriceType"
              :price-unit="list.PriceUnit"
              :tariff-id="list.TariffID"
              :tariff-name="list.TariffName"
              :tag-lists="list.FeatureTagList"
              :description="list.Description"
              :release-date="list.ReleaseDate"
              @send-tariff-data="receiveTariffLists">
            </product-card>
          </div>
        </div>
        <div class="product__pagination text-center mt-15">
          <v-pagination
            v-model="pageNumber"
            :length="pageMaxLength"
            @input="changePage"></v-pagination>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
export default {
  data() {
    return {
      dialog: false,
      page: '',
      pageMaxLength: '',
      productLists: [],
      productTariffList: [],
      categoryLists: [],
      searchCategoryLists: [],
      searchMakerLists: [],
      searchTagLists: [],
      searchPriceLists: [],
      searchProductListCount: '',
      categoryTagID: '',
      keyword: '',
      selectedCategoryLists: [],
      selectedMakerLists: [],
      selectedTagLists: [],
      selectedPriceLists: [],
      conditionJson: {
        CategoryID: null,
        MakerList: null,
        PriceRangeID: null,
        FeatureTagList: null,
      },
      orderRelease: '',
      orderPrice: '',
      tariffDialog: false,
      tariffLists: [],
      conditionalSearchFlg: false,
      searchConditionInfo: [],
      presentConditions: [],
      presentCategoryID: '',
      breadCrumbs: [],
      title: '',
    }
  },
  async fetch() {
    this.$store.commit('loading/changeStatus', true)
    if(this.$route.query.page) this.page = this.$route.query.page
    if(this.$route.query.price||this.$route.query.price===0) this.orderPrice = Number(this.$route.query.price)
    if(this.$route.query.release||this.$route.query.release===0) this.orderRelease = Number(this.$route.query.release)
    if(this.$route.query.type === '0') {
      this.extractPresentCondition(this.$store.getters['searchCondition/getInfo'])
      this.setSelectedData(this.$store.getters['searchCondition/getInfo'])
      this.setCondisionJson()
      await this.searchProductsUsingFilter()
    } else {
      await this.searchProducts()
    }

    this.setBreadCrumbs(this.$route.query.type)
    this.$store.commit('loading/changeStatus', false)
  },
  head () {
    return {
      title: "機材一覧",
      meta: [
        { hid: 'description', name: 'description', content: '機材一覧 | 大阪 東京 名古屋 京都での映像機器・音響機器のレンタルや学会・展示会・式典・試写会などのイベント制作・運営/HDV映像・音楽制作/VJ機器・HDVカメラなどクリエイターに向けの映像設備のプランニング・施工/デジタルサイネージやITネットワーク構築など映像・音響・レンタル・販売・設備に関することは80年の歴史を持つ(株)タケナカにご相談下さい。' }
      ]
    }
  },
  computed: {
    isCategoryName() {
      return this.$route.query.categoryName ? 1 : 0
    },
    isTagName() {
      return this.$route.query.tagName ? 1 : 0
    },
    isKeyword() {
      return this.$route.query.keyword ? (this.$route.query.keyword !== 'undefined' ? 1 : 0) : 0
    },
    pageNumber: {
      get() {
        return Number(this.page)
      },
      set(value) {
        this.page = value
      },
    },
  },
  watch: {
    page(newVal, oldVal) {
      if(Number(newVal) !== Number(oldVal)) {
        this.$router.push({query: {...this.$route.query, page: newVal}})
      }
    },
    orderPrice(newVal, oldVal) {
      if(String(newVal) !== String(oldVal)&&newVal!=='') {
        const query = Object.assign({}, this.$route.query)
        delete query.release
        query.price= newVal
        this.$router.push({query})
      }
    },
    orderRelease(newVal, oldVal) {
      if(String(newVal) !== String(oldVal)&&newVal!=='') {
        const query = Object.assign({}, this.$route.query)
        delete query.price
        query.release= newVal
        this.$router.push({query})
      }
    },
  },
  mounted(){
    window.addEventListener('popstate', this.popstateHook)
  },
  beforeDestroy(){
    window.removeEventListener('popstate', this.popstateHook)
  },
  updated() {
    this.$scrollBackButton()
  },
  methods: {
    popstateHook() {
      if(this.$route.query.type === '0') return
      if(this.$route.query.page) this.page = this.$route.query.page
      this.orderPrice = this.$route.query.price ? Number(this.$route.query.price):''
      this.orderRelease = this.$route.query.release ? Number(this.$route.query.release):''
      this.keyword = this.$route.query.keyword ? this.$route.query.keyword:''
      // if (this.$route.query.type === '0'&&this.conditionalSearchFlg) {
      //   this.searchProductsUsingFilter()
      // } else {
        this.conditionJson.CategoryID = null
        this.conditionJson.MakerList = null
        this.conditionJson.FeatureTagList = null
        this.conditionJson.PriceRangeID = null
        this.initializePresentConditions()
        this.conditionalSearchFlg = false
        this.searchProducts()
      // }
    },
    async getCategoryInfo(categoryID) {
      const param = new URLSearchParams()
      param.append('ProjectKey', this.$config.PROJECT_KEY)
      param.append('LangType', this.$config.LANG_JAPANESE)
      param.append('CategoryID', categoryID)
      const res = await this.$axios.$post('get_category_info.php', param)
      return res.CategoryInfo[0]
    },
    async getCategoryListforSearch() {
      const param = new URLSearchParams()
      param.append('ProjectKey', this.$config.PROJECT_KEY)
      param.append('LangType', this.$config.LANG_JAPANESE)
      const res = await this.$axios.$post('get_category_list_search.php', param)
      this.searchCategoryLists = res.CategoryRootList
    },
    async getMakerListforSearch() {
      const param = new URLSearchParams()
      param.append('ProjectKey', this.$config.PROJECT_KEY)
      param.append('LangType', this.$config.LANG_JAPANESE)
      param.append('CategoryID', this.presentCategoryID === undefined ? '' : this.presentCategoryID)
      const res = await this.$axios.$post('get_maker_list_search.php', param)
      this.searchMakerLists = res.MakerIndexList
    },
    async getTagListforSearch() {
      const param = new URLSearchParams()
      param.append('ProjectKey', this.$config.PROJECT_KEY)
      param.append('LangType', this.$config.LANG_JAPANESE)
      param.append('CategoryID', this.presentCategoryID === undefined ? '' : this.presentCategoryID)
      const res = await this.$axios.$post('get_tag_list_search.php', param)
      this.searchTagLists = res.FeatureCategoryList
    },
    async getPriceListforSearch() {
      const param = new URLSearchParams()
      param.append('ProjectKey', this.$config.PROJECT_KEY)
      param.append('LangType', this.$config.LANG_JAPANESE)
      const res = await this.$axios.$post('get_price_range_list.php', param)
      this.searchPriceLists = res.PriceRangeList
    },
    async getFilterCondition() {
      await Promise.all([
        this.getCategoryListforSearch(),
        this.getMakerListforSearch(),
        this.getTagListforSearch(),
        this.getPriceListforSearch(),
      ])
      this.dialog = true
    },
    async searchProducts() {
      const conditionJSON = JSON.stringify(this.conditionJson)
      const param = new URLSearchParams()
      param.append('ProjectKey', this.$config.PROJECT_KEY)
      param.append('LangType', this.$config.LANG_JAPANESE)
      param.append('SearchType', this.$route.query.type!=='3'?this.$route.query.type:'0')
      param.append('Keyword', this.$route.query.keyword!=='undefined'?this.$route.query.keyword:"")
      param.append('CategoryTagID', this.$route.query.categoryID)
      param.append('SearchTagID', this.$route.query.tagID)
      param.append('ConditionJSON', conditionJSON)
      param.append('OrderRelase', this.orderRelease)
      param.append('OrderPrice', this.orderPrice)
      param.append('PageRowCnt', this.$config.PAGE_ROW_COUNT)
      param.append('PageNo', this.page)
      // this.$store.commit('loading/changeStatus', true)
      const res = await this.$axios.$post('search_product.php', param)
      this.productLists = res.SearchProductList
      this.searchProductListCount = res.SearchAllCnt
      this.setPresentCategoryID()
      this.page = res.PageNo
      this.pageMaxLength = res.PageNoMax
      // this.$store.commit('loading/changeStatus', false)
    },
    async searchProductsUsingFilter() {
      const conditionJSON = JSON.stringify(this.conditionJson)
      const param = new URLSearchParams()
      param.append('ProjectKey', this.$config.PROJECT_KEY)
      param.append('LangType', this.$config.LANG_JAPANESE)
      param.append('SearchType', '0')
      param.append('Keyword', this.keyword||'')
      // param.append('CategoryTagID', '')
      // param.append('SearchTagID', '')
      param.append('ConditionJSON', conditionJSON)
      param.append('OrderRelase', this.orderRelease)
      param.append('OrderPrice', this.orderPrice)
      param.append('PageRowCnt', this.$config.PAGE_ROW_COUNT)
      param.append('PageNo', this.page)
      // this.$store.commit('loading/changeStatus', true)
      const res = await this.$axios.$post('search_product.php', param)
      this.conditionalSearchFlg = true
      this.productLists = res.SearchProductList
      this.searchProductListCount = res.SearchAllCnt
      this.searchConditionInfo = res.SerchConditionInfo
      this.$store.commit('searchCondition/addInfo', res.SerchConditionInfo)
      this.$store.commit('searchCondition/addKeyword', res.KeyWord)
      this.page = res.PageNo
      this.pageMaxLength = res.PageNoMax
      if(this.$route.query.page&&this.pageMaxLength&&Number(this.$route.query.page) > this.pageMaxLength) {
        this.page = 1
        this.searchProductsUsingFilter()
        return
      }
      this.extractPresentCondition(this.searchConditionInfo)
      this.setPresentCategoryID()
      const query = { type:0, page:this.page }
      if (this.orderPrice !== '') query.price = this.orderPrice
      if (this.orderRelease !== '') query.release = this.orderRelease
      this.$router.push({query})
      await Promise.all([this.getMakerListforSearch(), this.getTagListforSearch()])
      this.setBreadCrumbs(this.$route.query.type)
      // this.$store.commit('loading/changeStatus', false)
    },
    initializeCondisionJson() {
      this.conditionJson.CategoryID = ''
      this.conditionJson.MakerList = []
      this.conditionJson.FeatureTagList = []
      this.conditionJson.PriceRangeID = ''
    },
    initializePresentConditions() {
      this.presentConditions = []
    },
    setBreadCrumbs(type) {
      const breadCrumbsLists = this.$store.getters['breadCrumbs/getLists']
      const isCategoryPage = breadCrumbsLists.some((list) => list.path === '/category')
      if (!isCategoryPage) this.$store.commit('breadCrumbs/deleteList')

      switch (type) {
        case '0':
          this.$store.commit('breadCrumbs/addList', { name: '絞り込み検索', path: '' })
          this.breadCrumbs = this.$store.getters['breadCrumbs/getLists']
          break
        case '1':
          this.$store.commit('breadCrumbs/addList', { name: this.$route.query.tagName, path: '' })
          this.breadCrumbs = this.$store.getters['breadCrumbs/getLists']
          break
        case '2':
          this.$store.commit('breadCrumbs/addList', { name: this.$route.query.categoryName, path: '' })
          this.breadCrumbs = this.$store.getters['breadCrumbs/getLists']
          break
        case '3':
          this.$store.commit('breadCrumbs/addList', { name: 'キーワード検索結果', path: '' })
          this.breadCrumbs = this.$store.getters['breadCrumbs/getLists']
          break

        default:
          break
      }
    },
    setCondisionJson() {
      this.initializeCondisionJson()

      if (this.keyword === undefined) {
        this.keyword = ''
      }

      if (this.selectedCategoryLists.length) {
        this.selectedCategoryLists.forEach((element) => {
          this.conditionJson.CategoryID = element.id
        })
      } else {
        this.conditionJson.CategoryID = null
      }

      if (this.selectedMakerLists.length) {
        this.selectedMakerLists.forEach((element) => {
          this.conditionJson.MakerList.push({ MakerID: element.MakerID })
        })
      } else {
        this.conditionJson.MakerList = null
      }

      if (this.selectedTagLists.length) {
        this.selectedTagLists.forEach((element) => {
          this.conditionJson.FeatureTagList.push({ TagID: element.TagID })
        })
      } else {
        this.conditionJson.FeatureTagList = null
      }

      if (this.selectedPriceLists.length) {
        this.selectedPriceLists.forEach((element) => {
          this.conditionJson.PriceRangeID = element.id
        })
      } else {
        this.conditionJson.PriceRangeID = null
      }
    },
    setPresentCategoryID() {
      if (this.conditionalSearchFlg) {
        this.presentCategoryID =
          this.searchConditionInfo.CategoryID === null ? undefined : this.searchConditionInfo.CategoryID
      } else {
        switch (this.$route.query.type) {
          case '1':
            this.presentCategoryID = undefined
            break
          case '2':
            this.presentCategoryID = this.$route.query.categoryID
            break
          case '3':
            this.presentCategoryID = undefined
            break
          default:
            this.presentCategoryID = undefined
            break
        }
      }
    },
    setSelectedData(searchConditionInfo) {
      if(searchConditionInfo.CategoryFlg) {
        this.selectedCategoryLists.push({id: searchConditionInfo.CategoryID, name: searchConditionInfo.CategoryNmae02})
      } else {
        this.selectedCategoryLists = []
      }

      if(searchConditionInfo.MakerFlg) {
        this.selectedMakerLists = searchConditionInfo.MakerList
      } else {
        this.selectedMakerLists = []
      }

      if(searchConditionInfo.FeatureFlg) {
        this.selectedTagLists = searchConditionInfo.FeatureList
      } else {
        this.selectedTagLists = []
      }

      if(searchConditionInfo.PriceFlg) {
        this.selectedPriceLists.push({id: searchConditionInfo.PriceRangeID, name: searchConditionInfo.PriceRangeName})
      } else {
        this.selectedPriceLists = []
      }
      this.keyword = searchConditionInfo.KeyWordFlg ? this.$store.getters['searchCondition/getKeyword'] : undefined
    },
    changeOrderPrice() {
      this.orderRelease = ''
      this.page = 1
      if (this.conditionalSearchFlg) {
        this.searchProductsUsingFilter()
      } else {
        this.searchProducts()
      }
    },
    changeOrderRelease() {
      this.orderPrice = ''
      this.page = 1
      if (this.conditionalSearchFlg) {
        this.searchProductsUsingFilter()
      } else {
        this.searchProducts()
      }
    },
    async changePage(page) {
      this.page = Number(page)
      if (this.conditionalSearchFlg) {
        await this.searchProductsUsingFilter()
      } else {
        await this.searchProducts()
      }
      window.scrollTo({ top: 0 })
    },
    extractPresentCondition(searchConditionInfo) {
      this.initializePresentConditions()

      if (searchConditionInfo.KeyWordFlg) {
        this.presentConditions.push(this.keyword)
      }

      if (searchConditionInfo.CategoryFlg) {
        if (searchConditionInfo.CategoryNmae01 !== null) {
          this.presentConditions.push(searchConditionInfo.CategoryNmae01)
        }
        if (searchConditionInfo.CategoryNmae02 !== null) {
          this.presentConditions.push(searchConditionInfo.CategoryNmae02)
        }
      }

      if (searchConditionInfo.MakerFlg) {
        searchConditionInfo.MakerList.forEach((element) => {
          this.presentConditions.push(element.MakerName)
        })
      }

      if (searchConditionInfo.FeatureFlg) {
        searchConditionInfo.FeatureList.forEach((element) => {
          this.presentConditions.push(element.TagName)
        })
      }

      if (searchConditionInfo.PriceFlg) {
        this.presentConditions.push(searchConditionInfo.PriceRangeName)
      }
    },
    reseiveDialogFlg(value) {
      this.dialog = value
    },
    receivedSearchConditions(categoryLists, makerLists, tagLists, priceLists, keyword) {
      this.selectedCategoryLists = categoryLists
      this.selectedMakerLists = makerLists
      this.selectedTagLists = tagLists
      this.selectedPriceLists = priceLists
      this.keyword = keyword
      this.page = 1
      this.setCondisionJson()
      this.searchProductsUsingFilter()
      this.dialog = false
    },
    reseiveTariffDialogFlg(value) {
      this.tariffDialog = value
    },
    receiveTariffLists(value) {
      this.tariffLists = value
      this.tariffDialog = true
    },
    async updateCategoryLists(lists) {
      if (lists.length === 0) {
        this.presentCategoryID = undefined
      } else {
        this.presentCategoryID = lists[0].id
      }
      await this.getFilterCondition()
    },
    async receivedCategoryReset() {
      this.presentCategoryID = undefined
      await this.getFilterCondition()
    },
    async receivedAllReset() {
      this.presentCategoryID = undefined
      await this.getFilterCondition()
    },
    async resetConditions() {
      const categoryInfo = await this.getCategoryInfo(this.presentCategoryID)
      window.location.href =
        '/products?type=2&page=1&categoryID=' + categoryInfo.CategoryID + '&categoryName=' + categoryInfo.CategoryName
    },
  },
}
</script>

<style lang="scss" scoped>
@import 'assets/css/common.scss';

.products {
  position: relative;
  height: auto;

  &__inner {
    width: 100%;
    max-width: 1200px;
    margin: 0 auto;
  }

  .content {
    width: 70%;
    flex-grow: 1;
    max-width: 860px;

    @include mq(lg) {
      width: 100%;
      margin: 0 auto;
    }
  }

  .product__search {
    width: 100%;
    border: 1px solid $outline;
    border-radius: 5px;
    overflow: hidden;

    .search__number {
      background-color: $cushion;
      flex-shrink: 0;
      max-width: 185px;
      min-width: 103px;

      .number__val {
        font-size: 30px !important;
        @include mq(md) {
          font-size: 23px !important;
        }
      }
    }

    .search__condition {
      background-color: #ffffff;
      border-radius: 5px;
      border-top-left-radius: 0;
      border-bottom-left-radius: 0;
      overflow: hidden;
    }

    .condition__foot {
      position: relative;

      ul {
        list-style: none;
        display: flex;
      }

      .search-tag {
        white-space: nowrap;
      }

      .fade-out {
        background: linear-gradient(to right, rgba(255, 255, 225, 0) 0%, #fff 50%, #fff 100%);
        width: 24px;
        height: 100%;
        top: 0;
        right: 0;
        position: absolute;
      }
    }
  }

  .product__sort {
    .sort__release,
    .sort__price {
      .release__head,
      .price__head {
        background-color: $primary;

        @include mq(lg) {
          width: 200px;
          text-align: center;
        }

        @include mq(sm) {
          width: 150px;
          text-align: center;
        }
      }

      .v-btn--active {
        font-weight: 600 !important;
      }

      .v-btn--active::before {
        background-color: #ffffff !important;
        border-bottom: 2px solid $primary !important;
        opacity: 1 !important;
      }
    }

    .sort__inner {
      @include mq(lg) {
        width: 80%;
        max-width: 450px;
      }
      @include mq(sm) {
        width: 100%;
      }
    }

    .sort__head {
      background-color: $primary;
      min-width: 95px;
    }

    .sort__release,
    .sort__price {
      @include mq(lg) {
        width: 46%;
      }
    }

    .sort__line {
      border-color: #8e8e8e;
    }
  }

  .condition__reset:disabled {
    opacity: 0.5;
  }
}
</style>
