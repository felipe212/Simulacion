<template>
  <div class="mx-2">
    <b-row>
      <b-col cols="3" class="text-left mb-3">
        <b-button @click="simulatePolicyB">Simular B</b-button>
      </b-col>
    </b-row>
    <vue-good-table styleClass="vgt-table condensed" theme="black-rhino" :columns="columns" :rows="rows"/>
  </div>
</template>

<script>
export default {
  name: 'my-component',
  props: {
    policy: Object
  },
  data () {
    return {
      policyB: {
        amount: 25,
        period: 20
      },
      demands: [
        {amount: 0, interval: 4},
        {amount: 1, interval: 16},
        {amount: 2, interval: 34},
        {amount: 3, interval: 59},
        {amount: 4, interval: 81},
        {amount: 5, interval: 100},
      ],
      delays: [
        {daysAmount: 1, interval: 14},
        {daysAmount: 2, interval: 34},
        {daysAmount: 3, interval: 74},
        {daysAmount: 4, interval: 100},
      ],

      KOCost: [
        {amount: 15, cost: 20},
        {amount: 40, cost: 30},
        {amount: 100, cost: 40}
      ],
      initialStock: 20,
      currentWeek: 1,
      KMCost: 5,
      KSCost: 5,
      columns: [
        {
          label: 'i',
          field: 'iteration',
          type: 'number'
        },
        {
          label: 'RND',
          field: 'rndDemand',
          type: 'number'
        },
        {
          label: 'Demanda (decenas)',
          field: 'demand',
          type: 'number'
        },
        {
          label: 'Demanda Acum',
          field: 'acumDemand',
          type: 'number'
        },
        {
          label: 'Stock',
          field: 'stock',
          type: 'number',
        },
        {
          label: 'Pedido',
          field: 'order',
          type: 'boolean',
        },
        {
          label: 'RND',
          field: 'rndArrive',
          type: 'number',
        },
        {
          label: 'Demora',
          field: 'delayArrive',
          type: 'number',
        },
        {
          label: 'Llegada Pedido',
          field: 'arrive',
          type: 'number',
        },
        {
          label: 'Cantidad Pedida',
          field: 'amountBuyed',
          type: 'number',
        },
        {
          label: 'KO',
          field: 'ko',
          type: 'number',
        },
        {
          label: 'KM',
          field: 'km',
          type: 'number',
        },
        {
          label: 'KS',
          field: 'ks',
          type: 'number',
        },
        {
          label: 'CT',
          field: 'totalCost',
          type: 'number',
        },
        {
          label: 'CT Acum',
          field: 'totalCostAcum',
          type: 'number',
        }
      ],
      missingAmount: 0,
      rows: [],
      week: {
        iteration: 0,
        rndDemand: null, 
        demand: '-',
        acumDemand: 0,
        stock: 20,
        order: 'Si',
        rndArrive: 0, 
        delayArrive: '-',
        arrive: '-',
        amountBuyed: 0,
        ko: 0,
        km: 0,
        ks: 0,
        totalCost: 0,
        totalCostAcum: 0
      }
    };
  },
  methods: {
    simulatePolicyB () {
      this.rows = []
      this.currentWeek = 1
      this.rows.push({ iteration: 0, rndDemand:'-', demand: '-', acumDemand: 0, stock: 20, order: '-', rndArrive: 0, delayArrive: '-', arrive: '-', amountBuyed: 0, ko: 0, km: 0, ks: 0, totalCost: 0, totalCostAcum: 0})
      while (this.currentWeek <= 300) {
        let weekToPush = Object.assign({}, this.week)
        if (weekToPush.arrive === this.currentWeek) {
          weekToPush.stock += weekToPush.amountBuyed
          weekToPush.amountBuyed = 0
          weekToPush.arrive = '-'
        }
        weekToPush.iteration = this.currentWeek
        weekToPush.rndDemand = this.generateRandomDemand()
        weekToPush.demand = this.getDemand(weekToPush.rndDemand)
        weekToPush.acumDemand += weekToPush.demand
        weekToPush.stock = this.sale(weekToPush.stock, weekToPush.demand)
        if((weekToPush.iteration % 20) == 0 ) {
          weekToPush.order = 'Si'
        } else {
          weekToPush.order = 'No'
          weekToPush.rndArrive = '-'
          weekToPush.delayArrive = '-'
          weekToPush.ko = 0
        }
        if (weekToPush.order === 'Si' || this.currentWeek === 1) {
          weekToPush.rndArrive = this.generateRandomArrive()
          weekToPush.delayArrive = this.getDelayArrive(weekToPush.rndArrive)
          weekToPush.arrive = this.currentWeek + weekToPush.delayArrive
          if (weekToPush.order === 'Si') {
          weekToPush.amountBuyed = weekToPush.acumDemand
          weekToPush.acumDemand = 0 
          }
          else {
            weekToPush.rndArrive = this.generateRandomArrive()
            weekToPush.delayArrive = this.getDelayArrive(weekToPush.rndArrive)
            weekToPush.arrive = this.currentWeek + weekToPush.delayArrive
            weekToPush.amountBuyed = 25
          }
          let selectedAmount = this.KOCost.find(cost => cost.amount >= weekToPush.amountBuyed)
          weekToPush.ko = weekToPush.amountBuyed * selectedAmount.cost
        }
        weekToPush.km = weekToPush.stock * 5
        weekToPush.ks = this.missingAmount * 9 * -1
        weekToPush.totalCost = weekToPush.ko + weekToPush.km + weekToPush.ks
        weekToPush.totalCostAcum += weekToPush.totalCost
        this.rows.push(weekToPush)
        this.missingAmount = 0
        this.week = weekToPush
        this.currentWeek += 1
      }
        this.$emit('clicked', this.rows[this.rows.length - 1].totalCostAcum)
    },
    generateRandomDemand () {
      let random = Math.random()
      return Math.round(random * 100) / 100
    },
    getDemand (random) {
      let selectedDemand = this.demands.find(demand => demand.interval / 100 >= random)
      return selectedDemand.amount
    },
    sale (stock, demand) {
      if (stock - demand >= 0) {
        return stock - demand
      } else {
        this.missingAmount = stock - demand
        return 0
      }
    },
    generateRandomArrive () {
      let random = Math.random()
      return Math.round(random * 100) / 100
    },
    getDelayArrive (random) {
      let selectedDelay = this.delays.find(delay => delay.interval / 100 >= random)
      return selectedDelay.daysAmount
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>