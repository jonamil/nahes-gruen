<template>
  <div class="content-view district-view">
    <div class="inner">
      <section class="inhabitants">
        <div class="included">
          <h1>
            <AnimatedNumber
              :number="includedInhabitants"
            />
          </h1>
          <span>Berliner*innen können unter diesen<br>Bedingungen einen Grünraum erreichen</span>
        </div>
        <div class="excluded">
          <h1>
            <AnimatedNumber
              :number="excludedInhabitants"
            />
          </h1>
          <span>Berliner*innen können unter diesen<br>Bedingungen <u>keinen</u> Grünraum erreichen</span>
        </div>
      </section>
      <section class="legend">
        <div class="figure" /> entspricht 10.000 Berliner*innen
      </section>
      <section class="districts">
        <div
          v-for="(district, index) in rankedDistricts"
          :key="index"
          class="district"
        >
          <mark :style="'width: ' + highlightWidth(district) + 'rem'" />
          <div class="name">{{ district.name }}</div>
          <div class="coverage">
            <div class="percentage">
              {{ formatPercentage(district.percentage) }}<span>%</span>
            </div>
            <div class="figures">
              <div
                v-for="figureNumber in district.figures"
                :key="figureNumber"
                :class="'figure' + (figureNumber / district.figures - 1 / district.figures * 0.5 <= district.percentage / 100 ? '' : ' excluded')"
              />
            </div>
          </div>
        </div>
      </section>
    </div>
  </div>
</template>

<script>
import AnimatedNumber from './AnimatedNumber.vue';

export default {
  name: 'DistrictView',

  components: {
    AnimatedNumber
  },

  data () {
    return {
      figureWidth: 14
    }
  },

  props: {
    coverage: {
      type: Object,
      default: () => {}
    },
    controlState: {
      type: Object,
      default: () => {
        return {
          transportMode: 'walking',
          transportMinutes: 5
        }
      }
    },
  },

  computed: {
    includedInhabitants() {
      return Math.round(this.coverage.city.inhabitants * this.coverage.city.percentages[this.controlState.transportMode][this.controlState.transportMinutes.toString()] / 100 / 1000) * 1000;
    },
    excludedInhabitants() {
      return this.coverage.city.inhabitants - this.includedInhabitants;
    },
    rankedDistricts() {
      const districtProperties = this.coverage.districts.properties;
      const districtPercentages = this.coverage.districts.percentages[this.controlState.transportMode][this.controlState.transportMinutes.toString()];

      let array = Object.keys(districtPercentages).map((key) => {
        return [key, districtPercentages[key]];
      });

      array.sort((a, b) => {
        if (b[1] - a[1] === 0) {
          return districtProperties[b[0]].figures - districtProperties[a[0]].figures;
        } else {
          return b[1] - a[1];
        }
        // if (districtProperties[b[0]].name < districtProperties[a[0]].name) {
        //   return 1;
        // } else {
        //   return -1;
        // }
      });

      let rankedDistricts = [];

      array.forEach(item => {
        rankedDistricts.push({
          name: districtProperties[item[0]].name,
          figures: districtProperties[item[0]].figures,
          percentage: item[1]
        });
      });

      return rankedDistricts;
    }
  },

  methods: {
    formatInhabitants(inhabitants) {
      return new Intl.NumberFormat('de-DE').format(inhabitants);
    },
    formatPercentage(percentage) {
      let digits = 1;
      if (percentage >= 99.95) digits = 0;

      return new Intl.NumberFormat('de-DE', { minimumFractionDigits: digits, maximumFractionDigits: digits }).format(percentage);
    },
    highlightWidth(district) {
      return Math.round(district.percentage / 100 * district.figures) * this.figureWidth;
    }
  }
}
</script>

<style>
.content-view.district-view {
  overflow: auto;
  background: #EBE3DB;
  z-index: 50;
}

#app.intro .content-view.district-view {
  overflow-x: hidden;
}

.district-view .inner {
  margin: 0 auto;
  padding: 105rem 50rem 38rem;
  width: 790rem;
  text-align: center;
  transition: opacity 0.5s ease-in-out, transform 0.5s ease-in-out;
}

#app.intro .district-view .inner {
  opacity: 0.35;
  transform: translateX(200rem);
}

.district-view section.inhabitants .excluded {
  margin-left: 50rem;
  margin-right: -12rem;
}

.district-view section.inhabitants div {
  display: inline-block;
  vertical-align: top;
  text-align: left;
}

.district-view section.inhabitants h1 {
  display: inline-block;
  margin: 0 0 5rem -2rem;
  font-size: 40rem;
  font-weight: 600;
  font-feature-settings: 'tnum';
  line-height: 1.2;
  letter-spacing: -2rem;
}

.district-view section.inhabitants .included h1 {
  margin-left: -18rem;
  padding: 0 20rem 0 16rem;
  border-radius: 100rem;
  background: #C9FE53;
}

.district-view section.inhabitants .excluded h1 {
  color: rgba(0,0,0,0.26);
}

.district-view section.inhabitants span {
  display: block;
  font-size: 12rem;
  font-weight: 600;
  line-height: 1.2;
}

.district-view section.inhabitants .excluded span {
  color: rgba(0,0,0,0.3);
}

.district-view section.inhabitants .excluded span u {
  text-decoration: none;
  border-bottom: 1rem solid rgba(0,0,0,0.22);
}

.district-view section.legend {
  margin: 37rem 0;
  font-size: 12rem;
  font-weight: 600;
}

.district-view section.legend .figure {
  display: inline-block;
  opacity: 0.75;
  vertical-align: middle;
  margin: 0 6rem 1rem -3rem;
  width: 14rem;
  height: 28rem;
  background: url('../assets/icons/figure-legend.svg') center no-repeat;
}

.district-view section.districts {
  text-align: left;
}

.district-view section.districts .district {
  position: relative;
  margin-bottom: 46rem;
}

.district-view section.districts .district mark {
  position: absolute;
  display: block;
  top: -4rem;
  bottom: -4rem;
  left: 130rem;
  padding: 0 6rem 0 84rem;
  border-radius: 100rem;
  background: #C9FE53;
  z-index: -1;
}

.district-view section.districts .district div {
  display: inline-block;
  vertical-align: middle;
}

.district-view section.districts .district .name {
  box-sizing: border-box;
  width: 140rem;
  margin: -20rem 0;
  padding-right: 25rem;
  font-size: 13rem;
  font-weight: 600;
  line-height: 1.2;
}

.district-view section.districts .district .coverage {
  height: 28rem;
}

.district-view section.districts .district .coverage .percentage {
  width: 74rem;
  font-size: 22rem;
  font-weight: 600;
  font-feature-settings: 'tnum';
  line-height: 28rem;
  letter-spacing: -1.5rem;
}

.district-view section.districts .district .coverage .percentage span {
  margin-left: 2rem;
}

.district-view section.districts .district .coverage .figures .figure {
  opacity: 0.75;
  width: 14rem;
  height: 28rem;
  background: url('../assets/icons/figure.svg') center no-repeat;
}

.district-view section.districts .district .coverage .figures .figure.excluded {
  opacity: 0.25;
}
</style>