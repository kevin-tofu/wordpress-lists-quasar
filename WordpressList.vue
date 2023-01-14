<style scoped>
.div-card {
    max-width: 20rem;
    width: 40rem;
    height: 30rem;
}
.my-card {
    width: 20rem;
    /* max-width: 20rem; */
    /* min-height: 50rem; */
    height: 25rem;
    /* width: 20%; */
    margin: 10px;
    text-align: center;
    /* background-color: #AAAAFF */
}
.image {
    width: 20rem;
    height: 10rem;
}
.card_title{
    /* width: 20rem; */
    /* height: 10rem; */
    background-color: #AAAAFF
}
.child_botton {
    position: absolute;
    bottom: 0;
}

</style>
<template>
  <div class='q-pa-md' style='max-width: 30rem'>
    <div class = 'column' style="height: 150pix">
      <div>
        <div class='col-6 col-md-6'>
          <q-select 
            v-model='selected' 
            :options='options' 
            label='Page'
            @update:model-value='event_selected'
          />
        </div>
        <div class='col-6 col-md-6'>
          <q-btn color='deep-orange' glossy label='<<' @click='evenv_btn_left2'/>
          <q-btn color='orange' glossy label='<' @click='evenv_btn_left1'/>
          <q-btn color='orange' glossy label='>' @click='evenv_btn_right1'/>
          <q-btn color='deep-orange' glossy label='>>' @click='evenv_btn_right2'/>
        </div>
      </div>
      <q-space />
      <div class="col-6 col-md-6">
        <q-input outlined v-model='serachKey' label='keyword for search' @keypress.enter="event_search"/>
        <q-btn color='secondary' icon-right='send' label = 'Search' @click='event_search' />
      </div>
    </div>
  </div>
  

  <div class='about'>
    
    <!-- <div class='q-pa-md row items-start q-gutter-lg'> -->
    <div class='q-gutter-md row items-start relative-position justify-center' >
      <div v-for='(item, index) in items' :key='index' class = 'div-card'>
        <q-card  
          class='my-card'
        >
          <q-img 
            class='image'
            :src='item.url_image'
            native-context-menu
            alt='Logo'
            basic
          ></q-img>
          
          <q-card-section>
            <div class='text-h6 card_title'
            >
              {{ item.title }}
            </div>
            <div class='text-subtitle2'>{{item.date}}</div>
          </q-card-section>

          <q-card-section class='q-pt-none'>
            {{ item.sentence }}
          </q-card-section>
                  
          <q-card-actions class='child_botton'>
            <q-btn 
              round 
              @click='bars[index] = true'
              icon='visibility'
              color='red'
            >
            </q-btn>
          </q-card-actions>
          <q-dialog v-model='bars[index]' transition-show='rotate'  transition-hide='rotate'>
            <q-card>
              <q-card-section >
                  <div v-html='item.html_content'></div>
              </q-card-section>
              <q-card-actions align='right'>
                  <q-btn flat label='Decline' color='primary' v-close-popup />
              </q-card-actions>
            </q-card>
          </q-dialog>

        </q-card>
      </div>
    </div>
  </div>

</template>

<script>

import { defineComponent, ref, onMounted, watch } from 'vue'
import axios from 'axios'

export default defineComponent({

  name: 'WordpressList',
  components: {
  },
  
  props: {
    url_base: String,
    id_categories: String
  },

  setup (props) {

    // const url_wpapi = `${props.url_base}/wp-json/wp/v2/posts/?_embed`
    const url_wpapi = `${props.url_base}/wp-json/wp/v2/posts`
    const items = ref([])
    const serachKey = ref('')
    const selected = ref(1) // v-model for q-select
    const currentPage = ref(0) // 1 to 5 if options are [1,2,3,4,5]
    const options = ref([1]) // [1,] [1,2,3,4,5], [5,6]
    const totalpage = ref(5) 
    const bars = ref([])
    
    function make_options(start, n_range, max_length){
      return [...Array(n_range)].map((_, i) => {
        if (5 * start + i + 1 <= max_length){
            return 5 * start + i + 1
        }
      })
    }
    watch(selected, (newval) => {
      get_post(newval)
    })
    watch(currentPage, (newval) => {
      options.value = make_options(newval, 5, totalpage.value)
      selected.value = newval * 5 + 1
    })
    
    
    const get_post = async function (page=1) {
    //   console.log(params)
    //   console.log(url_wpapi)
      const res = await axios.get(
        url_wpapi, 
        { 
          params: {
            page: page,
            categories: props.id_categories,
            search: serachKey.value
        }
      })
      totalpage.value = res.headers['x-wp-totalpages']

      items.value = await Promise.all(
        res.data.map(async (value, idx) => {
          return {
            idx: idx,
            url_image: value.jetpack_featured_media_url, 
            url_article: value.link,
            date: value.date,
            title: value.title.rendered,
            subtitle: value.date,
            html_content: value.content.rendered,
            // html_content: value.excerpt.rendered,
            // html_content: res2.data.content,
            // html_content: value.date,
            sentence: '',
          }
        })
      );
      
      options.value = make_options(currentPage.value, 5, totalpage.value)
      bars.value = res.data.map(() => { return false})
    };

    onMounted(async ()=> {
        await get_post()
      }
    )
    
    function event_search () {
      currentPage.value = 0
      get_post()
    }

    function event_selected () {
      get_post(selected.value)
    }
    function evenv_btn_left1() {
      if (selected.value > 1){
        selected.value -= 1
      }
    }
    function evenv_btn_right1() {
      if (selected.value < totalpage.value) {
        if (selected.value === 5*(currentPage.value + 1)){
            currentPage.value += 1
        } else{
            selected.value += 1
        }
      }
    }
    function evenv_btn_left2() {
      if (currentPage.value > 0){
        currentPage.value -= 1
      } else {
        selected.value = 1
      }
    }
    function evenv_btn_right2() {
      if (totalpage.value >= 5 * (currentPage.value + 1) + 1) {
        currentPage.value += 1
      } else {
        selected.value = totalpage.value
      }
    }

    return {
      items,
      options,
      selected,
      currentPage,
      event_selected,
      evenv_btn_left1,
      evenv_btn_right1,
      evenv_btn_left2,
      evenv_btn_right2,
      bars,
      serachKey,
      event_search
    }
  }
})
</script>
