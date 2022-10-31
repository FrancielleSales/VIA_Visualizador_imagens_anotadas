<template>
  <div id="app" class="d-flex flex-column"> 
    <Navbar />
    <div class="container flex-grow-1 mt-5 mb-5 py-2">
      <div class="col-md-4 col-sm-12 mx-auto">
        <div class="card">
          <div class="card-header">
            <h4>
              Faça o upload dos arquivos aqui
            </h4>
          </div>
          <div class="card-body">
            <form>
              <FileSelect />
            </form>
          </div>
        </div>
      </div>
    </div>
    <Footer /> 
  </div>  
</template>

<script>
import Navbar from './components/Layouts/Navbar.vue';
import FileSelect from './components/FileSelect.vue';
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