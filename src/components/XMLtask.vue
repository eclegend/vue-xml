<template>
  <div class="xml-task">
    <h3>LitRes</h3>
    <ol>
      <li v-if="count()">{{ list[1] + ' = ' }} <p class="xml-task__result">{{ count() }}</p> </li>
      <li v-if="texts()">{{ list[2] + ' = ' }} <p class="xml-task__result">{{ texts() }}</p></li>
      <li v-if="textsWithSpace()">{{ `${list[3]} =  `}} <p class="xml-task__result">{{ textsWithSpace() }}</p></li>
      <li v-if="wrongLinks() >= 0">{{ `${list[4]} =  `}} <p class="xml-task__result">{{ wrongLinks() }}</p></li>
    </ol>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'XMLtask',
  props: {
    msg: String
  },
  data() {
    return {
      book: null,
      list: {
        1: 'Число внутренних ссылок (теги <a href="#id">)',
        2: 'Суммарное число букв внутри тегов, не включая пробельные символы (<aaa dd="ddd">text</aaa> - четыре буквы)',
        3: 'Суммарное число букв нормализованного текста внутри тегов, включая и пробелы',
        4: 'Число битых внутренних ссылок (ссылки на несуществующие ID элементов)',
      }
    };
  },

  methods: {
    
    createElementFromHTML(htmlString) {
      var div = document.createElement('div');
      div.innerHTML = htmlString;
      return div; 
    },
    wrongLinks() {
      const html = this.createElementFromHTML(this.book);
      const localLinks = Array.from(html.querySelectorAll('a')).filter((el) => {
        return el.getAttribute('l:href') && el.getAttribute('l:href').indexOf('#') === 0;
      });
      let wrongLinksCount = 0;

      localLinks.forEach(el => {
        const href = el.getAttribute('l:href').trim();
        if (!html.querySelector(href)) {
          wrongLinksCount += 1;
        }
      });
      return wrongLinksCount;
    },
    count() {
      const links = String(this.book).match(/<a l:href="#/g);
      return Array.isArray(links) ? links.length : null;
    },
    texts() {
       /*eslint-disable */
      const html = this.createElementFromHTML(this.book);
      const sum = [].map.call(html.querySelectorAll('*'), function(el){
        return el.textContent
          .replace(/<\/?[^>]+>/g,'')
          .replace(/[.,\/#!$%\^&\*;:{}=\-_`~()]/g,"")
          .replace(/\s+/g,'')
          .replace(/[0-9]/g, '')
          .replace(/[\])}[{(]/g, '')
      })
      .filter(el => {
        return el.length > 0 && el !== '[]';
      })
      .map(el => el.length)
      .reduce((sum, a) => sum + a, 0);
      return sum || null;
      /*eslint-enable */
    },
    textsWithSpace() {
      /*eslint-disable */
      const html = this.createElementFromHTML(this.book);
      const sum = [].map.call(html.querySelectorAll('*'), function(el){
        return el.textContent
          .replace(/<\/?[^>]+>/g,'')
          .replace(/[.,\/#!$%\^&\*;:{}=\-_`~()]/g,"")
          .replace(/[0-9]/g, '')
          .replace(/[\])}[{(]/g, '')
      })
      .filter(el => {
        return el.length > 0 && el !== '[]';
      })
      .map(el => el.length)
      .reduce((sum, a) => sum + a, 0);
      return sum || null;
       /*eslint-enable */
    },
  },

  mounted() {
    axios
      .get('http://localhost:8080/litres.xml')
      .then(response => {
        this.book = response.data;
      })
  }
}
</script>

<style scoped>
  .xml-task {
    text-align: left;
    padding: 8px;
  }
  .xml-task > h3 {
    text-align: center;
    font-size: 24px;
  }
  .xml-task__result {
    display: inline-block;
    background-color: rgb(95, 160, 124);
    color: azure;
    padding: 4px;
    border-radius: 2px;
  }
</style>
