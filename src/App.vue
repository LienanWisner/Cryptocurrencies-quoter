<script setup>
import { ref, reactive, onMounted, computed } from "vue";
import Alerta from "./components/Alerta.vue"
import Spinner from "./components/Spinner.vue"

const monedas = ref([
  { codigo: "USD", texto: "Dolar estadounidense" },
  { codigo: "EUR", texto: "Euros" },
  { codigo: "ARS", texto: "Peso Argentino" },
  { codigo: "CLP", texto: "Peso Chileno" },
  { codigo: "PEN", texto: "Sol Peruano" },
  { codigo: "COP", texto: "Peso Colombiano" },
  { codigo: "UYU", texto: "Peso Uruguayo" },
  { codigo: "PYG", texto: "Peso Paraguayo" },
  { codigo: "BOB", texto: "Peso Boliviano" },
  { codigo: "VES", texto: "Bolivar Venezolano" },
  { codigo: "GTQ", texto: "Quetzal Guatemala" },
  { codigo: "MXN", texto: "Peso Mexicano" },
  { codigo: "GBP", texto: "Libras Esterlinas" },
  
]);

const criptomonedas = ref([]);
const error = ref("")
const cotizar = reactive({
  moneda: "",
  criptomoneda: "",
});
const cotizacion = ref({})
const cargando = ref(false)

onMounted(() => {
  const url =
    "https://min-api.cryptocompare.com/data/top/mktcapfull?limit=50&tsym=USD";
  fetch(url)
    .then((res) => res.json())
    .then(({ Data }) => (criptomonedas.value = Data));
});

const cotizarCripto = () => {
 if(Object.values(cotizar).includes("")){
  error.value= "Todos los campos son obligatorios"
  return
 }
 error.value = ""
 obtenerCotizacion()
}

 const obtenerCotizacion = async()=>{
  cargando.value = true
  cotizacion.value = {}
  try{

    const {moneda, criptomoneda} = cotizar
    const url = `https://min-api.cryptocompare.com/data/pricemultifull?fsyms=${criptomoneda}&tsyms=${moneda}`
    
    const respuesta = await fetch(url)
    const data = await respuesta.json()
    cotizacion.value = data.DISPLAY[criptomoneda][moneda]
    
  } catch(err){
    console.log(err)
  } finally{
    cargando.value = false
  }
  }

  const mostrarResultado = computed(()=>{
    return Object.values(cotizacion.value).length > 0
  })


</script>

<template>
  <div class="contenedor">
    <h1 class="titulo">Cotizador de <span>Criptomonedas</span></h1>
  </div>

  <div class="contenido">
    <Alerta v-if="error">
      {{ error }}
    </Alerta>
    <form class="formulario" @submit.prevent="cotizarCripto">
      <div class="campo">
        <label for="moneda">Moneda: </label>
        <select id="moneda" v-model="cotizar.moneda">
          <option value="">-- Selecciona --</option>
          <option v-for="moneda in monedas" :value="moneda.codigo">
            {{ moneda.texto }}
          </option>
        </select>
      </div>

      <div class="campo">
        <label for="cripto">Criptomoneda: </label>
        <select id="cripto" v-model="cotizar.criptomoneda">
          <option value="">-- Selecciona --</option>
          <option
            v-for="criptomoneda in criptomonedas"
            :value="criptomoneda.CoinInfo.Name"
          >
            {{ criptomoneda.CoinInfo.FullName }}
          </option>
        </select>
      </div>

      <input type="submit" value="Cotizar" />
    </form>

  <Spinner v-if = "cargando"/>
    
    <div v-if="mostrarResultado" class="contenedor-resultado">
      <h2>Cotización</h2>

      <div class="resultado">
        <img :src="'https://cryptocompare.com/' + cotizacion.IMAGEURL" alt="imagen cripto">
        <div>
          <p>El precio es de: <span>{{ cotizacion.PRICE }}</span></p>
          <p>Precio máximo del día: <span>{{ cotizacion.HIGHDAY }}</span></p>
          <p>Precio mínimo del día: <span>{{ cotizacion.LOWDAY }}</span></p>
          <p>Variación ultimas 24 hs: <span>{{ cotizacion.CHANGEPCT24HOUR }}%</span></p>
          <p>Ultima actualización: <span>{{ cotizacion.LASTUPDATE }}</span></p>
        </div>
      </div>

    </div>
  </div>
</template>

<style scoped></style>
