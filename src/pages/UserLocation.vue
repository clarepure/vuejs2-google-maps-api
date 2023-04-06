<template>
    <div>
    <section class="ui two column centered grid" style="position:relative;z-index:1;">
        <div class="column">
            <form class="ui segment large form">
                <div class="ui message red" v-show="error">{{ error }}</div>
                <div class="ui segment">
                <div class="ui field">
                    <!-- loading is a class attribute, do if spinner is true -->
                    <div class="ui right icon input large" :class="{loading:spinner}"> 
                        <input type="text" placeholder="Enter your address" v-model="address" ref="autocomplete"/>
                        <i class="dot circle link icon" @click="locatorButtonPressed"></i>
                    </div>
                </div>
            </div>
            </form>
        </div>
    </section> 

    <section id="map"></section>
</div>
</template>

<script>

import axios from 'axios';

export default {

    // define instance properties
    data() {
        return {
            address: "",  // bind with v-model directive
            error: "", // v-show directive, show on UI when there's an error
            spinner: false, // add/remove loading class dynamically
        }
    },

    mounted() { // will be called when the DOM is ready
        let autocomplete = new google.maps.places.Autocomplete(
            this.$refs["autocomplete"],
            {
                bounds: new google.maps.LatLngBounds(
                    new google.maps.LatLng(45.4215296, -75,6971931) // hard set coordinates to around Ontario, CA
                )
            }
        );
        
        // add event listener for change with input text
        autocomplete.addListener("place_changed", () => {
            // get selected place inside callback
            let place = autocomplete.getPlace();
            this.showUserLocationOnMap(place.geometry.location.lat(), place.geometry.location.lng());
        })
    },
    methods: {
        locatorButtonPressed() {

            this.spinner = true;

            // check if the geolocation api is supported by the users browser
            if(navigator.geolocation) {
                // which takes 3 arguments - (successFunc(position), errorFunc(error), Options{})
                navigator.geolocation.getCurrentPosition(
                    position => {
                        this.getAddressFrom(position.coords.latitude, position.coords.longitude);

                        this.showUserLocationOnMap(position.coords.latitude, position.coords.longitude);
                    },

                    error => {
                        this.error = 'Unable to find address, please type manually';
                        this.spinner = false;
                        console.log('error:', error.message);
                    }
                ); 
            } else {
                this.error = error.message;
                this.spinner = false;
                console.log("Your browser does not support geolocation API");
            }
        },
        getAddressFrom(lat,long) {
            axios.get("https://maps.googleapis.com/maps/api/geocode/json?latlng="
            + lat + 
            ","
            + long + "&key=AIzaSyCbidMd3TVwZsFk9WSjGpNFWQNCc3mq0zY")
            .then(response => {
                if(response.data.error_message) { // if api is invalid
                    this.error = response.data.error_message;
                    console.log('error', error);
                } else {
                    // reverse geocode
                    this.address = response.data.results[0].formatted_address
                }
                this.spinner = false;
            })
            .catch(error => {
                this.error = error.message;
                this.spinner = false;
                console.log(error);
            });
        },  
        // when button clicked (not autocomplete)
        showUserLocationOnMap(latitude, longitude) {
            // html element where map is going to be added to, js {} with 3 props
            let map = new google.maps.Map(document.getElementById("map"), {
                zoom: 15,
                center: new google.maps.LatLng(latitude, longitude), // center based on user
                mapTypeId: google.maps.MapTypeId.ROADMAP // map type (road map)
            }); 

            // Add marker
            // Pass JS {} with 2 params, position - where to place, map marker for which map to pass to
            new google.maps.Marker({ // 
                position: new google.maps.LatLng(latitude, longitude), // accepts lat,lng
                map: map,
            }) 

        }
    },
}
</script>


<style scoped>
.ui.button,
.dot.circle.icon {
    background-color: #de9b9d;
    color: white;


}

.pac-icon {
    display:none;
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
/* covers the full width */
#map {
    position: absolute;
    top:0;
    right:0;
    bottom:0;
    left:0;
    background:rgba(13, 92, 88, 0.306);
}
</style>