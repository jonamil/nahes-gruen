<template>
  <div class="content-view district-view">
    <div class="inner">
      <section class="inhabitants">
        <div class="included">
          <h1>2.760.000</h1>
          <span>Einwohner*innen können unter diesen<br>Bedingungen einen Grünraum erreichen</span>
        </div>
        <div class="excluded">
          <h1>910.000</h1>
          <span>Einwohner*innen können unter diesen<br>Bedingungen <u>keinen</u> Grünraum erreichen</span>
        </div>
      </section>
      <section class="legend">
        <div class="figure" /> entspricht 10.000 Einwohner*innen
      </section>
      <section class="districts">
        <div
          v-for="(district, index) in sortedDistricts"
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
                :class="'figure' + (figureNumber / district.figures <= Math.round(district.percentage * 10) / 1000 ? '' : ' excluded')"
              />
            </div>
          </div>
        </div>
      </section>
    </div>
  </div>
</template>

<script>
export default {
  name: 'DistrictView',

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
    sortedDistricts() {
      return [
        {
          name: 'Friedrichshain-Kreuzberg',
          percentage: 86.2,
          figures: 29
        },
        {
          name: 'Marzahn-Hellersdorf',
          percentage: 84.3,
          figures: 27
        },
        {
          name: 'Lichtenberg',
          percentage: 80.4,
          figures: 29
        },
        {
          name: 'Mitte',
          percentage: 79.7,
          figures: 39
        },
        {
          name: 'Spandau',
          percentage: 77.2,
          figures: 25
        },
        {
          name: 'Reinickendorf',
          percentage: 76.2,
          figures: 27
        },
        {
          name: 'Pankow',
          percentage: 72.6,
          figures: 41
        },
        {
          name: 'Charlottenburg-Wilmersdorf',
          percentage: 72.5,
          figures: 34
        },
        {
          name: 'Treptow-Köpenick',
          percentage: 72,
          figures: 27
        },
        {
          name: 'Steglitz-Zehlendorf',
          percentage: 71.9,
          figures: 31
        },
        {
          name: 'Neukölln',
          percentage: 66.8,
          figures: 33
        },
        {
          name: 'Tempelhof-Schöneberg',
          percentage: 66.7,
          figures: 35
        }
      ]
    }
  },

  methods: {
    highlightWidth(district) {
      return Math.floor(Math.round(district.percentage * 10) / 1000 * district.figures) * this.figureWidth;
    },
    formatPercentage(percentage) {
      let number = new Intl.NumberFormat('de-DE', { minimumFractionDigits: 1, maximumFractionDigits: 1 }).format(percentage);
      if (number.length > 4) number = number.substring(0, 3);
      return number;
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
  padding-right: 27rem;
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