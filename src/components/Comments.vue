<script>
    export default{
        data(){
            return{
                apartments:[]
            }
        },
        methods:{
            load(){
                fetch("http://127.0.0.1:8000/api/apartments").then(response=>response.json())
                .then(data=>{
                    this.apartments = data;
                    console.log(data);
                })
            }
        },
        mounted(){
            this.load();
        }
    }
</script>
<template>
    <div class="container-xxl py-5">
        <div class="container">
            <div class="text-center mx-auto mb-5 wow fadeInUp" data-wow-delay="0.1s" style="max-width: 600px;">
                <h1 class="mb-3">Apartmanok</h1>
                <p>Eirmod sed ipsum dolor sit rebum labore magna erat. Tempor ut dolore lorem kasd vero ipsum sit eirmod
                    sit. Ipsum diam justo sed rebum vero dolor duo.</p>
            </div>
            <div>
                <div  class="testimonial-item bg-light rounded p-3">
                    <div v-for="apartment in apartments" :key="apartment.id" class="bg-white border rounded p-4">
                        <p>{{apartment.description}}</p>
                        <div class="d-flex align-items-center">
                            <img class="img-fluid flex-shrink-0 rounded" src="" style="width: 45px; height: 45px;">
                            <div class="ps-3">
                                <h6 class="fw-bold mb-1">{{ apartment.name }}</h6>
                                <small>Profession</small>
                                <br>
                                <small>{{apartment.price_per_night}}</small>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<style scoped>

</style>