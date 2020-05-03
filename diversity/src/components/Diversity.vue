<template>
  <div>
    <b-form-select v-model="selected" :options="options" @change="getData()"
                   style="width: 50%;"></b-form-select>
    <h3>Employee Statistics of {{selected}}</h3>


    <b-row style="margin: 5%">
      <b-col style="margin-right: 10%">
        <label>Gender Diversity</label>
        <apexchart type="bar" :options="chartOptionsBar('Gender')" :series="series('Gender')"></apexchart>
      </b-col>

      <b-col style="margin-right: 10%">
        <label>Ethnicity Diversity</label>
        <apexchart type="bar" :options="chartOptionsBar('Ethnicity')" :series="series('Ethnicity')"></apexchart>
      </b-col>
    </b-row>

    <b-row style="margin: 5%">
      <b-col style="margin-right: 10%">
        <label>Location Diversity</label>
        <apexchart type="bar" :options="chartOptionsBar('Location')" :series="series('Location')"></apexchart>
      </b-col>
      <b-col style="margin-right: 10%">
        <label>Age Diversity</label>
        <apexchart type="bar" :options="chartOptionsBar('Age')" :series="series('Age')"></apexchart>
      </b-col>
    </b-row>

    <b-row style="margin: 5%">
      <b-col style="margin-right: 10%">
        <label>Language Diversity</label>
        <apexchart type="bar" :options="chartOptionsBar('Languages')" :series="series('Languages')"></apexchart>
      </b-col>
      <b-col style="margin-right: 10%">
        <label></label>
        <apexchart type="bar" :options="chartOptionsDonut('Disability')" :series="series('Disability')" height=350></apexchart>
      </b-col>
    </b-row>

    <b-row style="margin: 5%">
      <b-col style="margin-right: 10%">
        <apexchart type="bar" :options="chartOptionsDonut('NativeEng')" :series="series('NativeEng')" height=350></apexchart>
      </b-col>
      <b-col style="margin-right: 10%">
        <apexchart type="bar" :options="chartOptionsDonut('HighQualityCode')" :series="series('HighQualityCode')" height=350></apexchart>
      </b-col>
    </b-row>

    <b-row style="margin: 5%">
      <b-col style="margin-right: 10%">
        <apexchart type="bar" :options="chartOptionsDonut('Work/Life')" :series="series('Work/Life')" height=350></apexchart>
      </b-col>
      <b-col style="margin-right: 10%">
        <apexchart type="bar" :options="chartOptionsDonut('EQ>IQ')" :series="series('EQ>IQ')" height=350></apexchart>
      </b-col>
    </b-row>
  </div>
</template>

<script>
  import axios from 'axios';
  import VueApexCharts from 'vue-apexcharts';

  export default {
    name: "Diversity",
    data: function () {
      return {
        apiUrl: 'https://api.airtable.com/v0/',
        apiKey: 'keyt1yiZ5C6nlR2tQ', // Always use a read-only account token
        base: 'appXM3P6Rv7v8CulP',
        records: [],
        options: [
          {value: 'Amazon', text: 'Amazon'},
          {value: 'Google', text: 'Google'},
          {value: 'Apple', text: 'Apple'},
          {value: 'Facebook', text: 'Facebook'},
          {value: 'Salesforce', text: 'Salesforce'}
        ],
        selected: 'Amazon'
      };
    },
    components: {
      apexchart: VueApexCharts
    },
    methods: {
      getData() {
        axios({
          url: this.apiUrl + this.base + '/' + this.selected,
          headers: {
            'Authorization': `Bearer ${this.apiKey}`
          }
        }).then((res) => {
          this.records = res.data.records
        });
      },
      chartOptionsBar(type) {
        let categories = [];
        if (type === 'Gender') {
          categories = ['Male', 'Female', 'Other'];
        } else if (type === 'Ethnicity') {
          categories = ['White', 'Hispanic/Latinx', 'Asian', 'Black/African', 'Native American']
        } else if (type === 'Location') {
          categories = ['North America', 'South America', 'Asia', 'Europe', 'Africa', 'Australia']
        } else if (type === 'Age') {
          categories = ['40+', '35-40', '30-35', '25-30', '20-25', '20-'];
        } else if (type === 'Languages') {
          categories = ['English', 'Turkish', 'German', 'Spanish', 'Russian', 'Other']
        }
        return {
          chart: {
            type: 'bar',
            height: 350
          },
          plotOptions: {
            bar: {
              horizontal: true,
            }
          },
          dataLabels: {
            enabled: true
          },
          xaxis: {
            categories: categories,
          }
        }
      },
      chartOptionsDonut(type) {
        let categories = [];
        if (type === 'Disability') {
          categories = ['Has disability', 'Doesnt have disability'];
        } else if (type === 'NativeEng') {
          categories = ['Non-Native speaker', 'Native speaker']
        } else if (type === 'HighQualityCode') {
          categories = ['Has high quality code habit', 'Doesnt have quality code habit']
        } else if (type === 'Work/Life') {
          categories = ['Good work life balance', 'Bad work life balance'];
        } else if (type === 'EQ>IQ') {
          categories = ['Has EQ greater than IQ', 'Has IQ greater than EQ']
        }
        return {
          chart: {
            type: 'donut',
          },
          labels: categories,
          responsive: [{
            breakpoint: 480,
            options: {
              legend: {
                position: 'bottom'
              }
            }
          }]
        }
      },
      series(type) {
        let data = [];
        switch (type) {
          case 'Gender':
            data = this.countGenders();
            break;
          case 'Ethnicity':
            data = this.countEthnicity();
            break;
          case 'Location':
            data = this.countLocations();
            break;
          case 'Age':
            data = this.countAges();
            break;
          case 'Languages':
            data = this.countLanguages();
            break;
          case 'Disability':
            return this.count("Has Disability");
          case 'NativeEng':
            return this.count("Non-Native English Speaker");
          case 'HighQualityCode':
            return this.count("High Quality Code");
          case 'Work/Life':
            return this.count("Work/Life Balance");
          case 'EQ>IQ':
            return this.count("EQ>IQ");
        }

        return [{
          data: data
        }]
      },
      countGenders() {
        let male = 0, female = 0, other = 0;
        for (const employee of this.records) {
          if (employee.fields.Gender === 'Male') male++;
          else if (employee.fields.Gender === 'Female') female++;
          else other++;
        }
        return [male, female, other];
      },
      countEthnicity() {
        let white = 0, hispanic = 0, asian = 0, black = 0, nativeAmerican = 0;
        for (const employee of this.records) {
          if (employee.fields.Ethnicity === 'White') white++;
          else if (employee.fields.Ethnicity === 'Hispanic/Latinx') hispanic++;
          else if (employee.fields.Ethnicity === 'Asian') asian++;
          else if (employee.fields.Ethnicity === 'Black/African') black++;
          else nativeAmerican++;
        }
        return [white, hispanic, asian, black, nativeAmerican];
      },
      countLocations() {
        let NA = 0, SA = 0, europe = 0, asia = 0, africa = 0, australia = 0;
        for (const employee of this.records) {
          if (employee.fields.Location === 'North America') NA++;
          else if (employee.fields.Location === 'South America') SA++;
          else if (employee.fields.Location === 'Europe') europe++;
          else if (employee.fields.Location === 'Asia') asia++;
          else if (employee.fields.Location === 'Africa') africa++;
          else australia++;
        }
        return [NA, SA, asia, europe, africa, australia];
      },
      countAges() {
        const counts = [0, 0, 0, 0, 0, 0];
        for (const employee of this.records) {
          if (employee.fields.Age < 20) counts[5]++;
          else if (employee.fields.Age >= 20 && employee.fields.Age < 25) counts[4]++;
          else if (employee.fields.Age >= 25 && employee.fields.Age < 30) counts[3]++;
          else if (employee.fields.Age >= 30 && employee.fields.Age < 35) counts[2]++;
          else if (employee.fields.Age >= 35 && employee.fields.Age < 40) counts[1]++;
          else counts[0]++;
        }
        return counts;
      },
      countLanguages() {
        let english = 0, turkish = 0, german = 0, spanish = 0, russian = 0, other = 0;
        for (const employee of this.records) {
          if (employee.fields.Languages.includes('English')) english++;
          if (employee.fields.Languages.includes('Turkish')) turkish++;
          if (employee.fields.Languages.includes('German')) german++;
          if (employee.fields.Languages.includes('Spanish')) spanish++;
          if (employee.fields.Languages.includes('Russian')) russian++;
          if (employee.fields.Languages.includes('Other')) other++;
        }
        return [english, turkish, german, spanish, russian, other];
      },
      count(key) {
        const data = [0, 0];
        for(const employee of this.records) {
          if(employee.fields[key]) data[0]++;
          else data[1]++;
        }
        return data;
      }
    },
    mounted() {
      this.getData('Amazon');
    }
  }
</script>

<style scoped>

</style>
