<template>
  <div class="main">
    <search-field @searchReports="searchReports" @click="sortByDate"/>
    <p style="color: white; font-size: 30px; margin-bottom: 20px; text-align: center">Список актов</p>
    <div class="reports">
      <card-list :filter="filter" :searchField="searchField"/>
    </div>
  </div>
</template>

<script>

import axios from "axios";
import CardList from "@/components/CardList.vue";
import SearchField from "@/components/SearchField";

export default {
  name: 'MainView',
  data() {
    return {
      reports: [],
      searchField: "",
      search: ""
    }
  },
  components: {
    CardList,
    SearchField
  },
  methods: {
    async fetchData() {
      const userId = localStorage.getItem('user_id'); // Получение user_id из localStorage
      if (parseInt(localStorage.getItem('user_role')) === 2 || parseInt(localStorage.getItem('user_role')) === 3) {
        const res = await axios.get("http://91.222.238.181:5000/reports",
            {
              headers: {
                'Authorization': `Bearer ${this.$store.state.token}`
              }
            });
        this.reports = res.data;
        console.log(res.data);
      } else {

        const res = await axios.post("http://91.222.238.181:5000/reports/admin/",
          { id: userId },
          {
            headers: {
              'Authorization': `Bearer ${this.$store.state.token}`
            }
          });
      this.reports = res.data;
        console.log(res.data);
      }
      console.log(userId);
    },
    searchReports(value) {
      this.search = value;
    },
    sortByDate() {
      if (this.sortedByDate) {
        this.reports.sort((a, b) => new Date(a.date) - new Date(b.date));
        this.sortedByDate = false;
      } else {
        this.reports.sort((a, b) => new Date(b.date) - new Date(a.date));
        this.sortedByDate = true;
      }
    },
  },
  computed: {
    filter() {
      if (!this.search) {
        return this.reports;
      } else {
        return this.reports.filter(report =>
            Object.values(report).some(value => {
              if (typeof value === 'string') {
                return value.toLowerCase().includes(this.search.toLowerCase());
              }
              return false;
            })
        );
      }
    },
  },
  mounted() {
    this.fetchData();
  }
}
</script>

<style>
.main {
  height: 100%;
}

</style>
