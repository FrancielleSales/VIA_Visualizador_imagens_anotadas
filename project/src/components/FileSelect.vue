<template>
    <div class="buttons">
        <form>
            <label for="fileLabel">Label</label><br>
            <input type="file" class="btn" id="fileLabel" name="fileLabel" @change="handleFileUpload($event)" accept=".txt" />
            <br><br>
            <label for="fileImage">Imagem</label><br>
            <input type="file" class="btn" id="fileImage" name="fileImage" @change="handleFileUpload($event)" accept=".jpg, .png" />
            <br><br>
            <button v-on:click="submitFiles()" class="btn-primary">Submit</button>
        </form>
    </div>
</template>