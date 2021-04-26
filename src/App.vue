<script>
import ProductCreate from './components/ProductCreate'
import ProductList from './components/ProductList'

export default {
  name: 'App',
  components: {
    ProductCreate,
    ProductList
  }
}
</script>

<template>

  <div class="uk-container uk-padding-large">
    <ul class="uk-subnav uk-subnav-pill" uk-switcher="animation: uk-animation-fade">
      <li><a href="#">ثبت محصول</a></li>
      <li><a href="#">مشاهده مخصولات</a></li>
    </ul>

    <ul class="uk-switcher uk-margin">
      <li>
        <product-create/>
      </li>
      <li>
        <ProductList/>
      </li>
    </ul>
  </div>
</template>

<style src="uikit/dist/css/uikit-rtl.min.css"></style>
<style>

</style>
