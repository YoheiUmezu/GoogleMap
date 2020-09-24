<template>
    <div>
        <section class="ui two column centered grid" style="position: relative; z-index: 1;">
            <div class="column">
                <form class="ui segment large form" action="">
                    <div class="ui message red" v-show="error">{{ error }}</div>
                    <div class="ui segment">
                        <div class="field">
                            <div class="ui right icon input large" :class="{loading: spinner}">
                                <input type="text" placeholder="enter your address" 
                                v-model="address" 
                                id="autocomplete"
                                />
                                <i class="dot circle link icon" @click="locatorButtonPressed"></i>
                            </div>
                        </div>
                        <button class="ui button">Go</button>
                    </div>
                </form>
            </div>
        </section>
        <section id="map">

        </section>
    </div>
</template>

<script>
import axios from 'axios';

    export default {
        name: "UserLocation",
        data() {
            return {
                address: "",
                error: "",
                spinner: false
            }
        },
        mounted() {
            const google = window.google;
            let autocomplete = new google.maps.places.Autocomplete(
                document.getElementById("autocomplete"),
                {
                    bounds: new google.maps.LatLngBounds(
                        new google.maps.LatLng(35.652832, 139.839478)//https://www.latlong.net/place/tokyo-japan-8040.html#:~:text=The%20latitude%20of%20Tokyo%2C%20Japan,%C2%B0%2050'%2022.1208''%20E.
                    )
                }
            );
            autocomplete.addListener("place_changed", () => {
                let place = autocomplete.getPlace()
                this.showUserLocationOnTheMap(place.geometry.location.lat(), place.geometry.location.lng())
            })
        },
        methods: {
            locatorButtonPressed() {
                this.spinner = true

                if(navigator.geolocation) {
                    navigator.geolocation.getCurrentPosition(position => {
                        this.getAddressForm(position.coords.latitude, position.coords.longitude);
                        this.showUserLocationOnTheMap(position.coords.latitude, position.coords.longitude);
                    }, error => {
                        this.error = "Locator is unable to find your location. Please type your address."
                        this.spinner = false
                        //console.log(error.message)
                    })
                } else {
                    this.error = error.message
                    this.spinner = false
                    //console.log('Your browser does not support geolocation API.')
                }
            },
            getAddressForm(lat, long) {
                axios.get("https://maps.googleapis.com/maps/api/geocode/json?latlng=" + lat + "," + long + "&key=AIzaSyAnp23T2tbYj9Ho8uYmcE6W4KnbvZKlEjc")
                .then(response => {
                    if(response.data.error_message) {
                        this.error = error.message
                    } else {
                        this.address = response.data.results[0].formatted_address
                    }
                    this.spinner = false
                })
                .catch(error => {
                    this.error = error.message
                    this.spinner = false
                    //console.log(error.message)
                })
            },

            showUserLocationOnTheMap(latitude, longitude) {
                //Create a map object
                const google = window.google;
                let map = new google.maps.Map(document.getElementById("map"),{
                    zoom: 15,
                    center: new google.maps.LatLng(latitude, longitude),
                    mapTypeId: google.maps.MapTypeId.ROADMAP
                });

                //Add marker
                new google.maps.Marker({
                    position: new google.maps.LatLng(latitude, longitude),
                    map: map
                })
            }
        }
    }
</script>

<style lang="scss" scoped>
.ui.button,
.dot.circle.icon {
    background-color: #ff5a5f;
    color: white;
}

.pac-icon {
    display: none;
}

.pac-item {
    padding: 10px;
    font-size: 16px;
    cursor: pointer;
}

.pac-item:hover {
    background-color: #ececec;
}

.pac-item-query {
    font-size: 16px;
}

#map {
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    background-color: orange;
}
</style>