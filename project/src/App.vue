<template>
  <div id="app" class="d-flex flex-column"> 
    <Navbar />
    <FileSelect />
    <Footer /> 
  </div>  
</template>

<script>
import Navbar from './components/Layouts/Navbar.vue';
import FileSelect from './components/VIA/FileTreatment.vue';
import Footer from './components/Layouts/Footer.vue';

export default {
  name: "App",
  components:{
    Navbar,
    FileSelect,
    Footer,
  }
}
</script>