<template>
  <div id="app" class="d-flex flex-column"> 
    <Navbar />
    <div class="container flex-grow-1 mt-5 mb-5 py-2">
    </div>
    <Footer /> 
  </div>  
</template>

<script>
import Navbar from './components/Layouts/Navbar.vue';
import Footer from './components/Layouts/Footer.vue';

export default {
  name: "App",
  components:{
    Navbar,
    Footer,
  }
}
</script>