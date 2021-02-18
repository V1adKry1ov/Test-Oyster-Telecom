<template>
    <div class="wrapper">
        <div class="wrapper-content wrapper-content--fixed" v-if="isResponseErr">
            <p>Ошибка!!! Код ошибки: 500 Перезагрузите страницу</p>
        </div>
        <div class="wrapper-content wrapper-content--fixed" v-else>
        <section>
            <div class="container">
                <search :value="search" placeholder="Поиск" @search="search = $event"/>
                <table> 

                    <thead>
                        <tr>
                            <th @click="usersSort">Имя и адрес клиента</th>
                            <th>Почтовый адрес</th>
                            <th>Номер телефона</th>
                            <th>Дата создания</th>
                        </tr>
                    </thead>

                    <tbody>
                        <tr v-for="user in usersSearch" :key="user.id">
                            <td>{{ user.name }} <br> {{ user.addresses[0].streetName }} {{ user.addresses[0].streetNumber }}</td>
                            <td>{{ user.contacts[0].emails[0] }}</td>
                            <td>{{ user.contacts[0].phones[0] }}</td>
                            <td>{{ editDate(user.createdAt) }}</td>
                        </tr>
                    </tbody>

                </table>

            </div>
        </section>
        <section>
            <div class="contaier">
                <div class="button-list">
                    <div class="btn btnPrimary" @click="changePagination(5)" > 5 </div>
                    <div class="btn btnPrimary" @click="changePagination(10)"> 10 </div>
                    <div class="btn btnPrimary" @click="changePagination(15)"> 15 </div>
                </div>
                <br>
                <div class="button-list">
                    <div class="btn btnPrimary" @click="prevPage"> &#8592; </div>
                    <div class="btn btnPrimary" @click="nextPage"> &#8594; </div>
                </div> 
            </div>
        </section>
        </div>
    </div>
</template>

<script>
import search from '@/components/Search'

export default {
    components: {
        search
    },
    data() {
    return {
        users: null,
        currentSortDir: 'up',
        isResponseErr: null,
        search: '',
        page: {
            current: 1,
            length: 10
        }
    }
  }, 
  created() { 
    // Api с данными находятся под CORS защитой. Для ее обхода используется CORS Anywhere: https://github.com/Rob--W/cors-anywhere
    // Для того чтобы метод обхода сработал нужно получить доступ временный доступ к демонстрационному серверу. Иначе запрос к Api будет блокироваться
    // Требуется перейти по ссылке и нажать на кнопку https://cors-anywhere.herokuapp.com/corsdemo

    const x = new XMLHttpRequest();
    const cors_api_url = 'https://cors-anywhere.herokuapp.com/';
    const method = 'GET';
    const url = 'https://test-task-api.labilab.dev/clients'
    x.open(method, cors_api_url + url, false);
    x.onload = x.onerror = () => {
      console.log(
        method + ' ' + url + '\n' +
        x.status + ' ' + x.statusText + '\n\n' +
        (x.responseText || '')
      );
      

      if(JSON.parse(x.responseText).status == 500) {
        this.isResponseErr = true
      } else {
        this.isResponseErr = false
        this.users = JSON.parse(x.responseText).data
      }
      
    };
    if (/^POST/i.test(method)) {
      x.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
    }
    x.send();
  },
  computed: {
    usersSearch() {
        // Данное вычисляемое свойство следит за value input, отвечающим за поиск
        

        let array = this.users,
            search = this.search
        // При пустой поисковой строке возвращается обычный массив данных
        if(!search) return this.filterArr(array)
        
        // Если пользователь вводит данные в поисковой строке, возвращается отредакитрованный массив
        search = search.trim().toLowerCase()
        array = array.filter(function (item) {
            if(
                item.name.toLowerCase().indexOf(search) !== -1 || 
                (item.addresses[0].streetName + " " + String(item.addresses[0].streetNumber)).toLowerCase().indexOf(search) !== -1 ||
                item.contacts[0].emails[0].toLowerCase().indexOf(search) !== -1 ||
                item.contacts[0].phones[0].toLowerCase().indexOf(search) !== -1 ||
                (item.contacts[0].name + " " + item.contacts[0].surname).toLowerCase().indexOf(search) !== -1
            ){
                return item
            }
        })
        return array
    }
  },
  methods: {
    filterArr(arr) {
        return arr.sort((a, b) => {
            // Метод сортировки пользователей по имени

            // Счётчик для проверки направления сортировки
            let cur = 1 
            if (this.currentSortDir === 'down') cur = -1 
            // После проверки направления массив изменяется в нужную сторону
            if(a.name < b.name) return -1 * cur
            if(a.name > b.name) return 1 * cur
            return 0
        }).filter((row,index) => {
            // Метод пагинации

            // Высчитывается начало и конец пагинации и выводится массив с нужным количесвтом элементов
            let start = (this.page.current - 1)*this.page.length
            let end = this.page.current*this.page.length
            if (index >= start && index < end) return true
        })
    },
    usersSort() {
        // Метод изменения направления сортировки

        this.currentSortDir = this.currentSortDir === 'up' ? 'down' : 'up'
    },
    editDate(val) {
        // Метод редактирования даты в нужный формат

        let currentDate = new Date(val),
        year = currentDate.getFullYear(),
        month = currentDate.getMonth(),
        day = currentDate.getDate(),
        hours = currentDate.getHours(),
        minutes = currentDate.getMinutes(),
        monthsArr = ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'],
        fullDate

        if(day < 10 ) day = `0${day}`
        if(minutes < 10 ) minutes = `0${minutes}`

        fullDate  = `${day} ${monthsArr[month]} ${year} ${hours}:${minutes}`

        return fullDate
    },
    prevPage () {
        // Перелиствыание страницы вперёд 

        if (this.page.current > 1) this.page.current-=1
    },
    nextPage () {
        // Перелиствыание страницы назад 

        if ((this.page.current * this.page.length) < this.users.length) this.page.current+=1
    },
    changePagination(val) {
        // Изменение количества отображаемых пользователей

        this.page.length = val
    }
  }
} 
</script>
<style lang="scss" scoped>
    .button-list {
        width: 100%;
        text-align: center;

        .btn {
            border-radius: 60px;
            margin: 0 20px;
        }
    }
</style>