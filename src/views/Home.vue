<template>
  <div class="home">
    <div class="home__table-container">
      <table class="table is-fullwidth is-hoverable is-narrow is-striped">
        <thead>
          <tr>
            <th
              v-for="category in categories"
              :key="category.id"
              @click="updateFilters(category.id)"
              class="item-hover has-text-primary"
            >
              {{ category.title }}
            </th>
            <th>
              <div :class="dropdownClasses" class="dropdown" @click="openDropdown">
                <div class="dropdown-trigger">
                  <button class="button" aria-haspopup="true" aria-controls="dropdown-menu">
                    <span>Number of films: {{ limit === 0 ? 'Tous les films' : limit }}</span>
                    <span class="icon is-small">
                      <i class="fas fa-angle-down" aria-hidden="true"></i>
                    </span>
                  </button>
                </div>
                <div class="dropdown-menu" role="menu">
                  <div class="dropdown-content">
                    <p
                      v-for="limit in limitList"
                      :key="limit.title"
                      @click="updateLimitValue(limit.value)"
                      class="dropdown-item"
                    >
                      {{ limit.title }}
                    </p>
                  </div>
                </div>
              </div>
            </th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="film in films"
            :key="film.title"
            class=""
          >
            <td>{{ film.title }}</td>
            <td>{{ film.price || 'Prix non renseigné' }}</td>
            <td>{{ film.rating }}</td>
            <td>{{ film.category }}</td>
            <td>{{ film.nbOfRates }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <nav v-if="limit !== 0" class="pagination is-centered" role="navigation" aria-label="pagination">
      <ul class="pagination-list">
        <li v-if="nbPages > 1" @click="navigate(1)" class="item-hover">
          <p class="pagination-link">1</p>
        </li>
        <li><span class="pagination-ellipsis">&hellip;</span></li>
        <li
          v-for="page in pagination"
          :key="`page-${page}`"
          @click="navigate(page)"
          class="item-hover"
        >
          <p class="pagination-link">{{ page }}</p>
        </li>
        <li><span class="pagination-ellipsis">&hellip;</span></li>
        <li class="item-hover" @click="navigate(nbPages)">
          <p class="pagination-link">{{ nbPages }}</p>
        </li>
      </ul>
    </nav>
  </div>
</template>

<script>
// utils
import axios from 'axios';

export default {

  name: 'Home',

  data() {
    return {
      categories: [
        { id: 1, title: 'Title' },
        { id: 2, title: 'Price' },
        { id: 3, title: 'Rating' },
        { id: 4, title: 'Category' },
        { id: 5, title: 'Number of Rates' },
      ],
      limitList: [
        { value: 10, title: '10' },
        { value: 25, title: '25' },
        { value: 50, title: '50' },
        { value: 100, title: '100' },
        { value: 0, title: 'Display all' },
      ],
      films: [],
      nbPages: 0,
      limit: 10,
      page: 1,
      orderType: 1,
      orderByDesc: 0,
      dropDownIsOpened: false,
      isLoading: false,
    }
  },

  computed: {
    dropdownClasses() {
      return {
        'is-active': this.dropDownIsOpened
      }
    },
    pagination() {
      const pages = [];
      if (this.page - 1 > 1) {
        pages.push(this.page - 1);
      }
      if (this.page !== 1 && this.page !== this.nbPages) {
        pages.push(this.page);
      }
      if (this.page + 1 <= this.nbPages - 1 && this.page !== this.nbPages) {
        pages.push(this.page + 1);
      }
      return pages;
    }
  },

  mounted() {
    this.refresh();
  },

  methods: {
    refresh() {
      const { limit, page, orderType, orderByDesc } = this;
      this.isLoading = true;
      axios
        .get(`https://api-sgbdr-project.herokuapp.com/api/films?limit=${limit}&page=${page}&orderType=${orderType}&orderByDesc=${orderByDesc}`)
        .then(({ data }) => {
          this.films = data.films;
          this.nbPages = data.nbPages;
          setTimeout(() => {
            this.isLoading = false;
          }, 500);
        });
    },
    updateFilters(categoryId) {
      this.orderType = categoryId;
      this.orderByDesc = Number(!this.orderByDesc);
      this.refresh();
    },
    openDropdown() {
      this.dropDownIsOpened = !this.dropDownIsOpened;
    },
    updateLimitValue(value) {
      if (value === this.limit) return;
      this.limit = value;
      this.page = 1;
      this.refresh();
    },
    navigate(page) {
      if (page === this.page) return;
      this.page = page;
      this.refresh();
    }
  },

}
</script>

<style lang="scss" scoped>
  .home {
    height: 100vh;
    width: 100%;

    &__table-container {
      height: 80%;
      overflow-y: scroll;

      thead {
        position: relative;

        th {
          position: sticky;
          top: 0
        }
      }
    }

    .item-hover {

      &:hover {
        cursor: pointer;
        background-color: rgb(219, 219, 219);
        color: hsl(217, 71%, 53%) !important;
      }
    }

    .dropdown {

      .dropdown-item {

        &:hover {
          cursor: pointer;
        }
      }
    }

   
  }
</style>
