<script>
import DatePicker from 'vue3-persian-datepicker'

const persianDate = require('persian-date')

export default {
  name: 'ProductCreate',
  components: {
    DatePicker
  },
  data() {
    return {
      selectOptions: null,
      displayDate: '',
      product: {
        name: '',
        picture: null,
        text: '',
        radio: null,
        select: null,
        date: null
      },
      errorsMsg: ''
    }
  },
  methods: {
    setFile(event) {
      this.product.picture = event.target.files[0]
    },
    postProduct() {
      this.product.date = this.gregorianDate
      if (this.isFormValid) {
        this.errorsMsg = ''
        const URL = 'http://recruitment.deploysite.ir/api/product'
        const formData = new FormData()
        formData.set('name', this.product.name)
        formData.set('text', this.product.text)
        formData.set('radio', this.product.radio)
        formData.set('select', this.product.select)
        formData.set('date', this.product.date)
        formData.append('picture', this.product.picture, this.product.picture.name)
        this.$axios.post(URL, formData, {
          config: {headers: {'Content-Type': 'multipart/form-data'}}
        })
            .then(res => {
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
      } else {
        this.errorsMsg = 'تمامی فیلد ها الزامی می باشند.'
      }
    }
  },
  computed: {
    gregorianDate() {
      const date = this.displayDate.split('/')
      const y = parseInt(date[0])
      const m = parseInt(date[1])
      const d = parseInt(date[2])
      return new persianDate([y, m, d]).toCalendar('gregorian').toLocale('en').format('YYYY-MM-DD')
    },
    isNameValid() {
      return this.product.name.length > 0
    },
    isTextValid() {
      return this.product.text.length > 0
    },
    isPictureValid() {
      const validImageTypes = ['image/jpeg', 'image/png']
      return this.product.picture !== null && validImageTypes.includes(this.product.picture['type'])
    },
    isRadioValid() {
      return this.product.radio !== null
    },
    isSelectValid() {
      return this.product.select !== null
    },
    isDateValid() {
      return this.product.date !== null
    },
    isFormValid() {
      return this.isNameValid && this.isTextValid && this.isPictureValid &&
          this.isRadioValid && this.isSelectValid && this.isDateValid
    }
  },
  created() {
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
    <h3>ثبت محصول جدید</h3>
    <div class="uk-width-medium">
      <div class="uk-margin">
        <input v-model="product.name" class="uk-input" type="text" placeholder="نام محصول">
      </div>

      <div class="uk-margin" uk-margin>
        <div uk-form-custom="target: true">
          <input @change="setFile" type="file">
          <input class="uk-input" type="text" placeholder="انتخاب عکس" disabled>
        </div>
      </div>

      <div class="uk-margin">
        <textarea style="resize: none;" v-model="product.text" class="uk-textarea" rows="5"
                  placeholder="توضیحات"></textarea>
      </div>

      <div class="uk-margin">
        <DatePicker v-model="displayDate"/>
      </div>

      <div class="uk-margin" v-if="selectOptions">
        <select class="uk-select" v-model="product.select">
          <option v-for="option in selectOptions" :value="option.value">{{ option.label }}</option>
        </select>
      </div>

      <div class="uk-margin uk-grid-small uk-child-width-auto uk-grid">
        <label><input class="uk-radio" type="radio" value="One" v-model="product.radio"> گزینه 1</label>
        <label><input class="uk-radio" type="radio" value="Two" v-model="product.radio"> گزینه 2</label>
      </div>

      <button class="uk-button uk-button-primary" @click="postProduct">ثبت محصول</button>

      <div class="uk-margin-medium-top">
        <span class="uk-label uk-label-danger">{{ errorsMsg }}</span>
      </div>
    </div>
  </div>
</template>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.datePicker {
  position: relative;
  width: fit-content;
  direction: rtl;
}

.datePicker input {
  border: 1px solid #eee;
}

.datePicker .datePicker__section {
  position: absolute;
  top: 110%;
  left: 50%;
  transform: translateX(-50%);
  box-shadow: 0 0 15px -10px rgba(0, 0, 0, 0.3);
  border: 1px solid #eee;
  z-index: 999;
}

.datePicker table {
  position: relative;
  background: #fff;
}

.datePicker table tr th {
  font-weight: normal;
}

.datePicker table thead tr:last-child {
  margin-bottom: 0.5rem;
  color: #979ca6;
}

.datePicker table thead tr:first-child th:first-child,
.datePicker table thead tr:first-child th:last-child {
  margin: 0 0.25rem;
}

.datePicker table thead tr:first-child th button {
  transition: 0.3s;
  padding: 0.1rem;
  background: #f0f0f0;
}

.datePicker table thead tr:first-child th:first-child button,
.datePicker table thead tr:first-child th:last-child button {
  background: #f1f1f1;
  border-radius: 0;
  border: none;
}

.datePicker table thead tr:first-child th:first-child button:hover,
.datePicker table thead tr:first-child th:last-child button:hover {
  background: #ffa64d;
}

.datePicker table thead tr:first-child th:nth-child(2) button {
  margin-top: 0.3rem;
}

.datePicker table thead tr:first-child th:nth-child(3) button {
  background: #f0f0f0;
  border: none;
  border-radius: 0.5rem;
  font-size: 16px;
}

.datePicker table tbody td.datePicker__td--disable {
  color: #d2d6dc;
}

.datePicker table tr {
  display: flex;
  padding: 0 0.3rem;
}

.datePicker table tr .datePicker__th__button button {
  font-size: 13px;
  padding: 0;
}

.datePicker table tr td,
.datePicker table tr th:not(.datePicker__th__button):not(.datePicker__th) {
  width: 30px;
  height: 30px;
  line-height: 30px;
  text-align: center;
  transition: 0.1s;
  transform: scale(0.9);
  font-weight: normal;
  border-radius: 300px;
}

.datePicker table tr td:hover:not(th):not(.datePicker__td--disable),
.datePicker
table
tr
th:not(.datePicker__th__button):not(.datePicker__th):hover:not(th):not(.datePicker__td--disable) {
  background: #ffa64d;
  color: #fff;
  transform: scale(1);
  cursor: pointer;
}

.datePicker table thead tr:first-child {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin: 0.5rem 0;
}

.datePicker table .datePicker__div {
  position: absolute;
  content: "";
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #fff;
  padding: 0.3rem;
  z-index: 9999;
  overflow-y: auto;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
}

.datePicker table .datePicker__div button {
  margin: 0.2rem 0.1rem;
  width: 60px;
  font-size: 15px;
  padding: 0.4rem 0.3rem;
  text-align: center;
  border-radius: 0.5rem;
  border: none;
  background: #f0f0f0;
  transition: 0.13s;
  cursor: pointer;
}

.datePicker table .datePicker__div button:hover {
  background: #ffa64d;
  color: #fff;
}

.datePicker table tr td.datePicker--highlight {
  background: #fcc996;
}

.datePicker .datePicker--active {
  background: #ffa64d !important;
  color: #fff;
}

.datePicker table tr .datePicker__td--active {
  background: #ffa64d !important;
  color: #fff;
}

.datePicker .datePicker__div--invisible {
  visibility: hidden;
  opacity: 0;
}

.datePicker .datePicker__button {
  margin: 0.75rem 0.25rem 0.5rem;
}

.datePicker .datePicker__button button {
  background: #ffa64d !important;
  color: #fff;
  width: 100%;
  padding: 0.25rem 0;
  border-radius: 5px;
}

.datePicker {
  width: 100%;
}

.datePicker input {
  height: 40px;
  vertical-align: middle;
  display: inline-block;
  max-width: 100%;
  width: 100%;
  padding: 0 10px;
  background: #fff;
  color: #666;
  border: 1px solid #e5e5e5;
  transition: .2s ease-in-out;
  transition-property: color, background-color, border;
  box-sizing: border-box;
  margin: 0;
  border-radius: 0;
}
</style>
