<template>
  <div>
    <v-checkbox
      v-if="userJson.MemberType===1"
      v-model="userJson.TransFlg"
      label="過去に弊社とお取引あり (本人確認書類の提出は不要となります。)"
      hide-details
      class="d-inline-block"
      :readonly="read"
      :false-value="0"
      :true-value="1"
    ></v-checkbox>
    <ul v-if="!userJson.TransFlg" class="pa-4 mb-8 mt-3 cushion">
      <li class="note">
        本人登録確認後にレンタルがご利用可能となります。
      </li>
      <li class="note">
        登録できる添付ファイルはJPG・PNG・PDFで1枚につき3MBまでのファイルとなります。
      </li>
      <li class="note">
        住所確認できる書類が無い場合は保証金をお預かりする場合があります。
      </li>
      <li class="note">
        国民健康保険証(被用者保険証以外)などの場合は以下が必要となります。<br>
        銀行取引の証明できるもの(取引銀行通帳の表紙裏１枚目の写し又は、法人番号指定通知書の写し)
      </li>
      <li class="red--text note">
        マイナンバーカードは顔写真・住所記載面のみ(ナンバー記載面は添付しないでください)。
      </li>
      <li class="red--text note">
        書類の提出等で対応できない場合は、
        <a :href="contactUrl" target="_blank" class="link">お問い合わせ</a>からご相談ください。
      </li>
    </ul>
    <div class="px-2">
      <template v-if="!userJson.TransFlg">
        <v-row class="my-1">
          <v-col cols="12" md="6" class="pb-0 d-flex align-start">
            <span class="white--text red darken-1 px-2 me-2 rounded text-no-wrap body-2 line-height-sm">必須</span>
            <template v-if="user.MemberType===1">名刺</template>
            <span v-else class="line-height-sm">身分証明書<br>
            ( 運転免許証/健康保険証/パスポート )</span>
          </v-col>
          <v-col cols="12" md="6">
            <set-file :file-name="'file1'" :rule="'required|file'"
            :file-obj.sync="fileJson[0]" :read="read"></set-file>
          </v-col>
        </v-row>
        <v-row class="my-1">
          <v-col cols="12" md="6" class="pb-0 d-flex align-start">
            <span class="white--text red darken-1 px-2 me-2 rounded text-no-wrap body-2 line-height-sm">必須</span>
            <template v-if="user.MemberType===1">社会保険被保険証</template>
            <span v-else class="line-height-sm">現住所記載書類<br>( マイナンバーカード/発行後3ヶ月以内の書類<sup>*1</sup> )</span>
          </v-col>
          <v-col cols="12" md="6">
            <set-file :file-name="'file2'" :rule="'required|file'" :file-obj.sync="fileJson[1]" :read="read"></set-file>
          </v-col>
        </v-row>
        <v-row v-if="!(read&&!fileJson[2])" class="my-1">
          <v-col cols="12" md="6" class="pb-0 d-flex align-start">
            <template v-if="user.MemberType===1">
              <span class="white--text red darken-1 px-2 me-2 rounded text-no-wrap body-2 line-height-sm">必須</span>
              <span class="line-height-sm">
                運転免許証・パスポート又はマイナンバーカード
              </span>
            </template>
            <template v-else>
              <span class="white--text secondary px-2 me-2 rounded body-2 line-height-sm">任意</span>
              <span>学生証<sup>*2</sup></span>
            </template>
          </v-col>
          <v-col cols="12" md="6">
            <set-file v-if="user.MemberType===1"
              :file-name="'file3'" :rule="'required|file'"
              :file-obj.sync="fileJson[2]" :read="read"
            ></set-file>
            <set-file v-else
              :file-name="'file3'" :rule="'file'"
              :file-obj.sync="fileJson[2]" :read="read"
            ></set-file>
          </v-col>
        </v-row>

        <v-row v-if="user.MemberType===0" class="note mt-3 pb-8 last-row">
          <p>*1 住民票/公共料金郵便物/国税・地方税領収証書/納税証明書/社会保険料領収書</p>
          <p>*2 学生証を事前に提出いただくことで、学生支援割引をいたします。</p>
        </v-row>
      </template>
      <template v-if="userJson.MemberType===1 && !read">
        <!-- <v-divider class="my-3"></v-divider>
        <v-divider class="my-3"></v-divider> -->
        <v-row class="mt-6 mb-1">
          <v-col cols="12" md="4" class="pb-0">
            <span class="white--text red darken-1 px-2 py-1 rounded body-2">必須</span>
            本社住所
          </v-col>
          <v-col cols="12" md="8">
            <set-address
              :set-zip-code.sync="userJson.HOfficeZipCode"
              :set-prefect.sync="userJson.HOfficePrefect"
              :set-address.sync="userJson.HOfficeAddress">
            </set-address>
          </v-col>
        </v-row>
        <ValidationProvider
          v-slot="{ errors }"
          name="HeadTel"
          rules="required|num">
          <v-row class="my-1">
            <v-col cols="12" md="4" class="pb-0">
              <span class="white--text red darken-1 px-2 py-1 rounded body-2">必須</span>
              本社電話番号
            </v-col>
            <v-col cols="12" md="8">
              <v-text-field
                v-model="userJson.HOfficeTel"
                outlined
                dense
                hide-details="auto"
                placeholder="0123456789"
                inputmode="numeric"
                type="tel"
                autocomplete="tel"
                :error-messages="errors"
                @blur="userJson.HOfficeTel=toNum($event.target.value)"
              ></v-text-field>
            </v-col>
          </v-row>
        </ValidationProvider>

        <ValidationProvider
          v-slot="{ errors }"
          name="HeadFax"
          rules="num">
          <v-row class="my-1">
            <v-col cols="12" md="4" class="pb-0">
              <span class="white--text secondary px-2 py-1 rounded body-2">任意</span>
              本社FAX番号
            </v-col>
            <v-col cols="12" md="8">
              <v-text-field
                v-model="userJson.HOfficeFax"
                outlined
                dense
                hide-details="auto"
                placeholder="0123456789"
                inputmode="numeric"
                type="tel"
                autocomplete="tel"
                :error-messages="errors"
                @blur="userJson.HOfficeFax=toNum($event.target.value)"
              ></v-text-field>
            </v-col>
          </v-row>
        </ValidationProvider>

        <ValidationProvider
          v-slot="{ errors }"
          name="representative"
          rules="required|max:100">
          <v-row class="my-1">
            <v-col cols="12" md="4" class="pb-0">
              <span class="white--text red darken-1 px-2 py-1 rounded body-2">必須</span>
              代表者名
            </v-col>
            <v-col cols="12" md="8">
              <v-text-field
                v-model="userJson.Representative"
                outlined
                dense
                hide-details="auto"
                :error-messages="errors"
              ></v-text-field>
            </v-col>
          </v-row>
        </ValidationProvider>

        <v-row class="my-1">
          <v-col cols="12" md="4" class="pb-0">
            <span class="white--text red darken-1 px-2 py-1 rounded body-2">必須</span>
            設立年月
          </v-col>
          <v-col cols="12" md="8" class="d-flex align-end">
            <v-menu
              v-model="datePicker"
              :close-on-content-click="false"
              :nudge-right="40"
              transition="scale-transition"
              offset-y
              min-width="auto"
            >
              <template #activator="{ on, attrs }">
                <ValidationProvider
                  v-slot="{ errors }"
                  name="incorporationM"
                  rules="required">
                  <v-text-field
                    v-model="incorporation"
                    readonly outlined dense
                    hide-details="auto"
                    :error-messages="errors"
                    v-bind="attrs"
                    v-on="on"
                  ></v-text-field>
                </ValidationProvider>
              </template>
              <v-date-picker
                v-model="incorporation" :readonly="read"
                type="month"
                locale="jp-ja"
                :active-picker.sync="activePicker"
                :max="incorporateMaxDate"
                min="1700-01"
                @input="datePicker = false"
                @change="userJson.Incorporation = dateFormat('remove',incorporation,'-')"
              ></v-date-picker>
            </v-menu>
          </v-col>
        </v-row>

        <ValidationProvider
          v-slot="{ errors }"
          name="CorpNumber"
          rules="max:20">
          <v-row class="my-1">
            <v-col cols="12" md="4" class="pb-0">
              <span class="white--text secondary px-2 py-1 rounded body-2">任意</span>
              法人番号
            </v-col>
            <v-col cols="12" md="8">
              <v-text-field
                v-model="userJson.CorpNumber"
                outlined
                dense
                hide-details="auto"
                inputmode="numeric"
                :error-messages="errors"
              ></v-text-field>
            </v-col>
          </v-row>
        </ValidationProvider>

        <v-row class="my-1">
          <v-col cols="12" md="4" class="pb-0">
            <span class="white--text red darken-1 px-2 py-1 rounded body-2">必須</span> 法人区分
          </v-col>
          <v-col cols="12" md="8">
            <v-radio-group v-model="userJson.CorpType"
              hide-details="auto"
              mandatory
              row
              class="mt-0">
              <v-radio
                label="上場"
                :value="1"
              ></v-radio>
              <v-radio
                label="非上場"
                :value="2"
              ></v-radio>
              <v-radio
                label="公共団体ほか"
                :value="3"
              ></v-radio>
            </v-radio-group>
          </v-col>
        </v-row>
        <v-row class="my-1">
          <v-col cols="12" md="4" class="pb-0">
            <span class="white--text red darken-1 px-2 py-1 rounded body-2">必須</span> 法人種別
          </v-col>
          <v-col cols="12" md="8">
            <v-radio-group v-model="userJson.CorpKind"
              hide-details="auto"
              mandatory
              row
              class="mt-0">
              <v-radio
                label="法人・団体(学術会議系）"
                value="法人・団体(学術会議系）"
                class="mb-1"
              ></v-radio>
              <v-radio
                label="法人・団体(その他）"
                value="法人・団体(その他）"
                class="mb-1"
              ></v-radio>
              <v-radio
                label="学校法人(大学・専門学校など）"
                value="学校法人(大学・専門学校など）"
                class="mb-1"
              ></v-radio>
              <v-radio
                label="個人"
                value="個人"
                class="mb-1"
              ></v-radio>
            </v-radio-group>
          </v-col>
        </v-row>


        <v-row class="my-1">
          <v-col cols="12" md="4" class="pb-0">
            <span class="white--text red darken-1 px-2 py-1 rounded body-2">必須</span> 業種
          </v-col>
          <v-col cols="12" md="8">
            <ValidationProvider
              v-slot="{ errors }"
              name="industry"
              rules="required">
              <v-select
                v-model="userJson.Industry"
                :items="industryList"
                hide-details="false"
                outlined
                dense
                :error-messages="errors"
              ></v-select>
            </ValidationProvider>
            <template v-if="userJson.Industry === '同業・関連会社'">
              <ValidationProvider
                v-slot="{ errors }"
                name="relatedIndustry"
                rules="required">
                <v-select
                  v-model="userJson.SameIndustry"
                  :items="relatedIndustryList"
                  class="mt-1"
                  hide-details="false"
                  outlined
                  dense
                  placeholder="タイプを選択して下さい"
                  :error-messages="errors"
                ></v-select>
              </ValidationProvider>
            </template>
          </v-col>
        </v-row>

        <v-row class="my-1">
          <v-col cols="12" md="4" class="pb-0">
            <span class="white--text red darken-1 px-2 py-1 rounded body-2">必須</span> お支払い方法
          </v-col>
          <v-col cols="12" md="8">
            <v-radio-group v-model.number="userJson.PaymentMethod"
              hide-details="auto"
              mandatory
              row
              class="mt-0 mb-4">
              <v-radio label="事前お振込"
                :value="0"
              ></v-radio>
              <!-- <v-radio label="店頭お支払い(現金)"
                :value="1"
              ></v-radio> -->
            </v-radio-group>
          </v-col>
        </v-row>

        <ValidationProvider
          v-slot="{ errors }"
          name="payName"
          rules="required|max:100">
          <v-row class="my-1">
            <v-col cols="12" md="4" class="pb-0">
              <template v-if="userJson.PaymentMethod===0">
                <span class="white--text red darken-1 px-2 py-1 rounded body-2">必須</span>
              </template>
              <template v-else>
                <span class="white--text secondary px-2 py-1 rounded body-2">任意</span>
              </template>
              お振込名義
            </v-col>
            <v-col cols="12" md="8">
              <v-text-field
                v-model="userJson.PayeeName"
                outlined
                dense
                hide-details="auto"
                :error-messages="errors"
              ></v-text-field>
            </v-col>
          </v-row>
        </ValidationProvider>
        <ValidationProvider
          v-slot="{ errors }"
          name="BillingEmail"
          rules="email|max:50">
          <v-row class="my-1 last-row">
            <v-col cols="12" md="4" class="pb-0">
              <span class="white--text secondary px-2 py-1 rounded body-2">任意</span> 請求書送付先メールアドレス
            </v-col>
            <v-col cols="12" md="8">
              <v-text-field
                v-model="userJson.BillingEmail"
                outlined
                dense
                hide-details="auto"
                type="email"
                autocomplete="email"
                :error-messages="errors"
              ></v-text-field>
            </v-col>
          </v-row>
        </ValidationProvider>
      </template>
    </div>
    <v-dialog v-model="resultDialog" width="780" persistent>
      <v-card class="pa-5 text-center">
        <result-card
          :result="result" :action="'本人確認の登録'"
          :path="'/myaccount'" :dialog.sync="resultDialog">
        </result-card>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
export default {
  props: {
    user: {
      type: Object,
      required: true,
    },
    file: {
      type: null,
      required: false,
      default: () => ([])
    },
    read: {
      type: Boolean,
      required: false,
      default: false
    }
  },
  data() {
    return {
      industryList: [
        '制作会社・プロダクション',
        '広告代理店',
        'イベント企画制作会社',
        '会議運営サービス・代理店（ＰＣＯ)',
        'デザイン・設計事務所',
        '旅行代理店',
        'その他プロダクション',
        '放送局・マスコミ',
        '建装・ディスプレイ（企画制作含む）',
        '印刷会社（企画制作含む）',
        'ホテル',
        'ホール・会館・結婚式場・展示会場',
        '【ホテル・ホール・会館・結婚式場・展示会場】付業者',
        '建装・舞台技術・ディスプレイ会社',
        '映像中継・収録技術',
        '映像・音響メーカー・機器代理店・販売特約店',
        '製薬会社・医療機器メーカー',
        'ＩＴ・システム会社',
        '同業・関連会社',
        'その他・該当なし'
      ],
      relatedIndustryList: [
        '同業者：JVR系',
        '同業者：JVR以外',
        '同業者：総合レンタル',
        '同業者：音響系',
        '同業者：その他',
        '関連業者：映像専門',
        '関連業者：その他',
      ],
      result: null,
      resultDialog: false,
      datePicker: false,
      incorporation: null,
      incorporateMaxDate: this.getMaxDate(),
      activePicker: null
    }
  },
  fetch(){
    if(this.userJson.Incorporation&&!this.read){
      const date = this.userJson.Incorporation
      this.incorporation = this.dateFormat('add', date, '-')
    }
  },
  computed: {
    userJson: {
      get(){
        return this.user
      },
      set(val){
        this.$emit('update:user', val)
      }
    },
    contactUrl() {
      return this.$store.getters['auth/getAuthToken'] ? '/myaccount/other/contact' : 'https://www.takenaka-co.co.jp/contact/'
    },
    fileJson: {
      get(){
        return this.file
      },
      set(val){
        this.$emit('update:file', val)
      }
    },
    isSameFileName(){
      const fileNames = this.fileJson.filter(file => file ).map((file) => file.name)
      const uniqueFileNames = new Set(fileNames)
      return uniqueFileNames.size !== fileNames.length
    }
  },
  watch: {
    datePicker (val) {
      val && setTimeout(() => (this.activePicker = 'YEAR'))
    },
    'userJson.Industry'(val){
      if(val!=='同業・関連会社') this.$set(this.userJson, 'SameIndustry', '')
    }
  },
  methods: {
    async register(){

      if(this.userJson.TransFlg) {
        this.$set(this.userJson, 'NecDocFlg', 1)
        return true
      }
      const formData = new FormData()
      let fileCnt = 0
      const postFileArr = this.$sameFileNameCheck(this.fileJson)
      postFileArr.forEach((file)=>{
        if(!file) return
        fileCnt++
        formData.append(`File_0${fileCnt}`, file, encodeURIComponent(file.name))
        this.$set(this.userJson, `DocFileName${fileCnt}`, file.name)
      })
      formData.append('FileCnt', fileCnt )
      const res = await this.$memberAxios.post(`comm/uploadFile/`, formData, {
      headers: {
          "Content-Type": "multipart/form-data;charset=UTF-8",
        }
      })

      const loginID = this.$store.getters["auth/getUser"]
      if(loginID) await this.$setLog('会員情報', '本人確認ファイル登録', res.data.Status)
      if(res.data.Status === 'TRUE'){
        this.$set(this.userJson, 'UploadKey', res.data.FileUploadKey)
        this.$set(this.userJson, 'NecDocFlg', 1)
        return true
      }else if(res.data.ErrorNo === 100002){
        const res = await this.$getAccessToken()
        if( res ) await this.register()
      }else{
        this.result = String(res.data.ErrorNo)
        this.resultDialog = true
      }
    },
    async updateUser(){
      const accessToken = this.$store.getters["auth/getAccessToken"]
      const loginID = this.$store.getters["auth/getUser"]
      const stringJson = JSON.stringify(this.userJson)
      const param = new URLSearchParams()
      param.append('JsonData', stringJson)
      const res = await this.$memberAxios.put(`member/${loginID}`, param,{
        headers: {
          Authorization: `Bearer ${accessToken}`
        }
      })

      this.$setLog('会員情報', '更新', res.data.Status)
      if(res.data.Status === 'TRUE'){
        this.result = 'success'
        this.resultDialog = true
      }else if(res.data.ErrorNo === 100002){
        const res = await this.$getAccessToken()
        if( res ) return this.updateUser()
      }else{
        this.result = String(res.data.ErrorNo)
        this.resultDialog = true
      }
    },
    toNum(e){
      return e.replace(/[０-９]/g, function(m) {
        return "０１２３４５６７８９".indexOf(m)
      }).replace(/-|ー|－/g,'')
    },
    dateFormat(action, date, separator){
      if(!date) return
      if(action==='add'){
        const year = date.substring(0, 4)
        const month = date.substring(4, 6)
        const formateDate = year+separator+month
        return formateDate
      }else if(action==='remove'){
        return date.replace(new RegExp(separator, 'g'), "")
      }
    },
    getMaxDate(){
      const today = new Date()
      const year = today.getFullYear()
      const month = String(today.getMonth() + 1).padStart(2, '0')
      return `${year+1}-${month}`
    },
  },
}
</script>

<style lang="scss" scoped>
@import 'assets/css/common.scss';
.line-height-sm{
  line-height: 1.5rem !important;
}
.note{
  padding-left: 1.4rem;
  list-style: none;
  @include wordSymbol('※')
}
.row:not(.last-row){
  border-bottom: 1px solid #dddddd;
  padding-bottom: 0.5rem;
  padding-top: 0.2rem;
}
.width-s{
  max-width: 6rem;
}
::v-deep{
  .v-input--checkbox .theme--light.v-label{
    color: $text;
    transform: rotate(0deg);
  }
}
</style>
