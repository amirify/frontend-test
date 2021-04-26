<script>
import DatePicker from 'vue3-persian-datepicker'
const persianDate = require('persian-date')

export default {
  name: "ProductList",
  components: {
    DatePicker
  },
  data() {
    return {
      products: [],
      selectOptions: null,
      displayDate: '',
      pictureBaseURL: 'http://recruitment.deploysite.ir/',
      selectedIndex: 0,
      selectedProduct: {
        name: '',
        picture: null,
        text: '',
        radio: null,
        select: null,
        date: null
      }
    }
  },
  methods: {
    setFile(event) {
      this.selectedProduct.picture = event.target.files[0]
    },
    editProduct(index) {
      this.selectedIndex = index
      this.selectedProduct.name = this.products[index].name
      this.selectedProduct.text = this.products[index].text
      this.selectedProduct.radio = this.products[index].radio
      this.selectedProduct.select = this.products[index].select
      this.selectedProduct.id = this.products[index].id

      this.$uikit.modal('#edit-modal').show()
    },
    updateProduct() {
      const URL = `http://recruitment.deploysite.ir/api/product/${this.selectedProduct.id}`
      const formData = new FormData()
      this.selectedProduct.date = this.gregorianDate
      formData.set('name', this.selectedProduct.name)
      formData.set('text', this.selectedProduct.text)
      formData.set('radio', this.selectedProduct.radio)
      formData.set('select', this.selectedProduct.select)
      formData.set('date', this.selectedProduct.date)
      formData.set('_method', 'put')
      formData.append('picture', this.selectedProduct.picture, this.selectedProduct.picture.name)
      this.$axios.post(URL, formData, {headers: {'Content-Type': 'multipart/form-data'}})
          .then(res => {
            this.products.splice(this.selectedIndex, 1, res.data.data)
            this.$uikit.modal('#edit-modal').hide()
            this.$uikit.notification({
              message: 'محصول با موفقیت ثبت شد.',
              status: 'success',
              pos: 'bottom-right',
              timeout: 5000
            })
          })
          .catch(err => {
            this.$uikit.notification({
              message: err.response.data.message,
              status: 'danger',
              pos: 'bottom-right',
              timeout: 5000
            })
          })
    },
    deleteProduct() {
      const deleteURL = `http://recruitment.deploysite.ir/api/product/${this.products[this.selectedIndex].id}`
      this.$axios.delete(deleteURL)
      .then(res => {
        this.products.splice(this.selectedIndex, 1)
        this.$uikit.modal('#confirm-modal').hide()
        this.$uikit.notification({
          message: 'محصول با موفقیت حذف شد.',
          status: 'success',
          pos: 'bottom-right',
          timeout: 5000
        })
      })
      .catch(err => {
        this.$uikit.notification({
          message: err.response.data.message,
          status: 'danger',
          pos: 'bottom-right',
          timeout: 5000
        })
      })
    },
    confirmProductDelete(index) {
      this.selectedIndex = index
      this.$uikit.modal('#confirm-modal').show()
    },
    convertDate(date) {
      return new persianDate(date).toLocale('fa').toCalendar('persian').format('YYYY/MM/DD')
    }
  },
  computed: {
    gregorianDate() {
      const date = this.displayDate.split('/')
      const y = parseInt(date[0])
      const m = parseInt(date[1])
      const d = parseInt(date[2])
      return new persianDate([y, m, d]).toCalendar('gregorian').toLocale('en').format('YYYY-MM-DD')
    }
  },
  created() {
    const URL = 'http://recruitment.deploysite.ir/api/product'
    this.$axios.get(URL)
        .then(res => {
          const productList = res.data.data
          for (let i = 0; i < productList.length; i++) {
            const getURL = `http://recruitment.deploysite.ir/api/product/${productList[i].id}`
            this.$axios.get(getURL)
                .then(res => {
                  this.products.push(res.data.data)
                })
          }
        })
        .catch(err => {
          console.log(err)
        })

    const SelectOptionsURL = 'http://recruitment.deploysite.ir/api/get/select/options'
    this.$axios.get(SelectOptionsURL)
        .then(res => {
          this.selectOptions = res.data.data
        })
        .catch(err => {
          console.log(err)
        })
  }
}
</script>

<template>
  <div class="uk-container uk-margin-large-top">
    <table v-if="products.length" class="uk-table uk-table-middle uk-table-striped uk-table-hover">
      <thead>
      <tr>
        <th>#</th>
        <th>تصویر</th>
        <th>نام</th>
        <th>تاریخ</th>
        <th>ویرایش / حذف</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="(p, index) in products">
        <td>{{ index + 1 }}</td>
        <td>
          <img :data-src="pictureBaseURL + p.picture" width="100" height="100" uk-img>
        </td>
        <td>{{ p.name }}</td>
        <td>{{ convertDate(p.date) }}</td>
        <td>
          <button class="uk-button uk-button-secondary uk-button-small uk-margin-small-left" @click="editProduct(index)">ویرایش</button>
          <button class="uk-button uk-button-danger uk-button-small" @click="confirmProductDelete(index)">حذف</button>
        </td>
      </tr>
      </tbody>
    </table>
  </div>





  <div id="edit-modal" uk-modal>
    <div class="uk-modal-dialog">
      <button class="uk-modal-close-default" type="button" uk-close></button>
      <div class="uk-modal-header">
        <h2 class="uk-modal-title">ویرایش محصول</h2>
      </div>
      <div class="uk-modal-body">
        <div class="uk-width-medium">
          <div class="uk-margin">
            <input v-model="selectedProduct.name" class="uk-input" type="text" placeholder="نام محصول">
          </div>

          <div class="uk-margin" uk-margin>
            <div uk-form-custom="target: true">
              <input @change="setFile" type="file">
              <input class="uk-input" type="text" placeholder="انتخاب عکس" disabled>
            </div>
          </div>

          <div class="uk-margin">
        <textarea style="resize: none;" v-model="selectedProduct.text" class="uk-textarea" rows="5"
                  placeholder="توضیحات"></textarea>
          </div>

          <div class="uk-margin">
            <DatePicker v-model="displayDate"/>
          </div>

          <div class="uk-margin" v-if="selectOptions">
            <select class="uk-select" v-model="selectedProduct.select">
              <option v-for="option in selectOptions" :value="option.value">{{ option.label }}</option>
            </select>
          </div>

          <div class="uk-margin uk-grid-small uk-child-width-auto uk-grid">
            <label><input class="uk-radio" type="radio" value="One" v-model="selectedProduct.radio"> گزینه 1</label>
            <label><input class="uk-radio" type="radio" value="Two" v-model="selectedProduct.radio"> گزینه 2</label>
          </div>
        </div>
      </div>
      <div class="uk-modal-footer uk-text-right">
        <button class="uk-button uk-button-default uk-modal-close" type="button">بستن</button>
        <button class="uk-button uk-button-secondary uk-margin-small-right" type="button" @click="updateProduct">اعمال تغییرات</button>
      </div>
    </div>
  </div>

  <div id="confirm-modal" uk-modal>
    <div class="uk-modal-dialog">
      <button class="uk-modal-close-default" type="button" uk-close></button>
      <div class="uk-modal-header">
        <h2 class="uk-modal-title">حذف محصول</h2>
      </div>
      <div class="uk-modal-body">
        <span>شما در حال حذف محصول هستید. آیا ادامه می دهید؟</span>
      </div>
      <div class="uk-modal-footer uk-text-right">
        <button class="uk-button uk-button-default uk-modal-close" type="button">بستن</button>
        <button class="uk-button uk-button-danger uk-margin-small-right" type="button" @click="deleteProduct">حذف</button>
      </div>
    </div>
  </div>

</template>

<style scoped>

</style>