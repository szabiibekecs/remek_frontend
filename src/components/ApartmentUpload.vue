<script>
export default {
    data() {
        return {
            apartment: {
                name: "",
                type_id: "",
                max_capacity: 1,
                description: "",
                price_per_night: ""
            },
            filters: {
                apartment_id: null,
                wellness: false,
                breakfast: false,
                parking: false,
                wifi: false,
                all_inclusive: false,
                near_the_beach: false,
                near_the_center: false,
                pet_friendly: false,
                smoking_allowed: false
            },
            location: {
                apartment_id: null,
                postal_code: '',
                city: '',
                street: '',
                address_number: '',
            },
            photos: [
                { url: "" }
            ],
            types: [],
        };
    },
    created() {
        this.fetchTypes();
    },
    methods: {
        fetchTypes() {
            fetch("http://127.0.0.1:8000/api/types")
                .then(response => response.json())
                .then(data => {
                    this.types = data;
                })
                .catch(error => {
                    console.error("Hiba a típusok lekérésekor:", error);
                });
        },

        uploadApartment() {
            fetch("http://127.0.0.1:8000/api/apartments", {
                method: "POST",
                headers: {
                    "Content-Type": "application/json"
                },
                body: JSON.stringify(this.apartment)
            })
                .then(response => response.json())
                .then(data => {
                    if (data.apartment && data.apartment.id) {
                        const apartmentId = data.apartment.id;
                        this.filters.apartment_id = apartmentId;
                        this.location.apartment_id = apartmentId;
                        this.uploadFilters();
                        this.uploadLocation();
                        this.uploadPhotos(apartmentId);
                        this.successMessage = "Szállás, szűrők és helyszín sikeresen mentve!";
                    } else {
                        this.uploadError = "Szállás létrejött, de nem kaptunk ID-t.";
                    }
                })
                .catch(error => {
                    console.error("Hiba a szállás feltöltésénél:", error);
                    this.uploadError = "Hiba a szállás feltöltésénél.";
                });
        },

        uploadLocation() {
            fetch("http://127.0.0.1:8000/api/locations", {
                method: "POST",
                headers: {
                    "Content-Type": "application/json"
                },
                body: JSON.stringify(this.location)
            })
                .then(response => response.json())
                .then(() => {
                    console.log("Helyszín mentve.");
                })
                .catch(error => {
                    console.error("Hiba a helyszín mentésekor:", error);
                    this.uploadError = "Szállás létrejött, de a helyszín mentése sikertelen.";
                });
        },

        uploadFilters() {
            fetch("http://127.0.0.1:8000/api/filters", {
                method: "POST",
                headers: {
                    "Content-Type": "application/json"
                },
                body: JSON.stringify(this.filters)
            })
                .then(response => response.json())
                .then(data => {
                    console.log("Sikeres szűrők mentése:", data);
                })
                .catch(error => {
                    console.error("Hiba a szűrők mentésekor:", error);
                    console.error("Szállás létrejött, de a szűrők mentése sikertelen.");
                });
        },
        addPhoto() {
            this.photos.push({ url: "" });
        },
        uploadPhotos(apartmentId) {
            const photoUploads = this.photos.map(photo => {
                return fetch("http://127.0.0.1:8000/api/photos", {
                    method: "POST",
                    headers: {
                        "Content-Type": "application/json"
                    },
                    body: JSON.stringify({
                        apartment_id: apartmentId,
                        url: photo.url
                    })
                });
            });

            Promise.all(photoUploads)
                .then(() => {
                    console.log("Fotók mentve.");
                })
                .catch(error => {
                    console.error("Hiba a fotók mentésekor:", error);
                    this.uploadError = "Szállás létrejött, de a fotók mentése sikertelen.";
                });
        }
    }
};
</script>

<template>
    <div class="container mt-5">
        <h2>Szálláshely Feltöltés</h2>
        <form @submit.prevent="uploadApartment">
            <div class="mb-3">
                <label class="form-label">Szálláshely neve:</label>
                <input type="text" v-model="apartment.name" class="form-control" required>
            </div>
            <div class="mb-3">
                <label class="form-label">Típus:</label>
                <select v-model="apartment.type_id" class="form-control" required>
                    <option v-for="type in types" :key="type.id" :value="type.id">{{ type.name }}</option>
                </select>
            </div>
            <div class="mb-3">
                <label class="form-label">Maximális kapacitás:</label>
                <input type="number" v-model="apartment.max_capacity" class="form-control" required>
            </div>
            <div class="mb-3">
                <label class="form-label">Leírás:</label>
                <textarea v-model="apartment.description" class="form-control"></textarea>
            </div>
            <div class="mb-3">
                <label class="form-label">Éjszakánkénti ár:</label>
                <input type="number" step="0.01" v-model="apartment.price_per_night" class="form-control" required>
            </div>
            <h5>Helyszín adatok</h5>
            <div class="mb-3">
                <label class="form-label">Irányítószám:</label>
                <input type="text" v-model="location.postal_code" class="form-control" required>
            </div>
            <div class="mb-3">
                <label class="form-label">Város:</label>
                <input type="text" v-model="location.city" class="form-control" required>
            </div>
            <div class="mb-3">
                <label class="form-label">Utca:</label>
                <input type="text" v-model="location.street" class="form-control" required>
            </div>
            <div class="mb-3">
                <label class="form-label">Házszám:</label>
                <input type="text" v-model="location.address_number" class="form-control" required>
            </div>
            <h5>Fotók</h5>
            <div v-for="(photo, index) in photos" :key="index" class="mb-3">
                <label class="form-label">Kép URL:</label>
                <input type="text" v-model="photo.url" class="form-control" required>
            </div>
            <button type="button" class="btn btn-secondary mb-3" @click="addPhoto">+ Új fotó</button>
            <div class="mb-3">
                <h5>Szolgáltatások / Szűrők:</h5>
                <div v-for="(value, key) in filters" :key="key" class="form-check form-switch">
                    <input type="checkbox" class="form-check-input" v-model="filters[key]" :id="key" />
                    <label class="form-check-label" :for="key">
                        {{key.replace(/_/g, " ").replace(/\b\w/g, l => l.toUpperCase())}}
                    </label>
                </div>
            </div>
            <button type="submit" class="btn btn-primary">Feltöltés</button>
        </form>
    </div>
</template>

<style>
.card {
    background-color: #fff;
    border-radius: 10px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}
</style>
