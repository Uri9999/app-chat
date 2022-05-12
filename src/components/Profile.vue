<template>
    <div class="profile">
    </div>
</template>

<script>
export default {
    data() {
        return {
            firebaseDocId: localStorage.getItem("FirebaseDocumentId"),
            id: localStorage.getItem("id"),
            name: localStorage.getItem("name"),
            aboutMe: localStorage.hasOwnProperty("description")
                ? localStorage.getItem("description")
                : "",
            photoURL: localStorage.getItem("photoURL"),
            newPhoto: null
        }
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
