<template>
  <table>
    <tr>
      <th>Species</th>
      <th>Colour</th>
      <th>First Appearance</th>
      <th>Weight [g]</th>
      <th>Context action</th>
    </tr>
    <tr v-for="model in models" :key="model.id">
      <td @click="browseWikipedia(model.species)" id="hyperlink">
        {{ model.species }}
      </td>
      <td>
        {{ model.colour }}
      </td>
      <td>
        {{ model.firstAppearance }}
      </td>
      <td>
        {{ model.weight }}
      </td>
      <td>
        <button class="btn btn-secondary" v-on:click="deleteModel(model.id)">DELETE</button>
      </td>
    </tr>
    <tr>
      <td>
        <input type="text" placeholder="Species..." v-model="newSpecies">
      </td>
      <td>
        <input type="text" placeholder="Colour..." v-model="newColour">
      </td>
      <td>
        <input type="number" v-model="firstAppearance">
      </td>
      <td>
        <input type="number" v-model="newWeight">
      </td>
      <td>
        <button class="btn btn-primary" v-on:click="add">ADD</button>
      </td>
    </tr>
  </table>
</template>

<script>
import axios from 'axios'
import {_} from 'vue-underscore'

let s = require("underscore.string");
export default {
  // eslint-disable-next-line vue/multi-word-component-names
  name: "Models",
  data() {
    return {
      initialModels: [],
      models: [],
      newSpecies: '',
      newColour: '',
      firstAppearance: 0,
      newWeight: 0,
    }
  },
  mounted() {
    axios.get('http://192.168.0.75:8080/').then((response) => {
      console.log('Models: ', response.data);
      this.initialModels = response.data;
      this.models = this.initialModels;
    });

    this.emitter.on('filter', (data) =>
    {
      let species = data.species.toUpperCase();

      this.models = _.filter(this.initialModels, function (model)
      {
        let currentSpecies = model.species.toUpperCase();
        if(s.include(currentSpecies, species))
        {
          return model;
        }
      })
    });
  },
  methods: {
    deleteModel(id) {
      axios.delete(`http://192.168.0.75:8080/${id}`).then((response) => {
        console.log(response.data);

        axios.get('http://192.168.0.75:8080/').then((response) => {
          console.log('Models: ', response.data);
          this.models = response.data;
        });
      }).catch(error => {
        console.error('ERROR: ', error);

        this.emitter.emit('error', {error: error.response.data});
      });
    },
    add() {
      let modelData = {species: this.newSpecies, colour: this.newColour, firstAppearance: this.firstAppearance, weight: this.newWeight};

      axios.post('http://192.168.0.75:8080/', modelData).then((response) => {
        console.log('Added model: ', response.data);

        axios.get('http://192.168.0.75:8080/').then((response) => {
          console.log('Models: ', response.data);
          this.models = response.data;
        });
      }).catch(error => {
        console.error('ERROR: ', error);

        this.emitter.emit('error', {error: error.response.data});
      });

      this.newSpecies = '';
      this.newColour = '';
      this.firstAppearance = 0;
      this.newWeight = 0;
    },
    browseWikipedia(text)
    {
      const searchURL = `https://en.wikipedia.org/wiki/Special:Search?search=${encodeURIComponent(text)}`;
      window.open(searchURL, '_blank');
    }
  },
}
</script>

<style scoped>
table {
  margin: 1% auto auto;
  border: solid darkturquoise;
  width: 77%;
}

th {
  font-size: 30px;
  border: solid darkturquoise;
}

tr {
  font-size: 23px;
}

td {
  border: solid darkturquoise;
}

button {
  font-size: 18px;
}

#hyperlink{
  cursor: pointer;
  color: blue;
}
</style>
