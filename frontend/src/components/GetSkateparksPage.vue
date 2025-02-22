<template>
    <center>
        <div class="text-left max-w-4xl mt-14 columns-1">
            <div class="fadeFilter">
                <span><a @click="changeOptions()" class="hover:bg-gray-300 rounded-lg ml-2 py-2 px-2 "><button class="text-black font-semibold">Filter By</button><i v-if="openOptions === false" class="bi bi-arrow-up text-lg"></i><i v-if="openOptions === true" class="bi bi-arrow-down text-lg"></i></a></span><br />
                <div v-if="openOptions === true" class="ml-2">
                    <select
                        class="text-lg mt-2 break-after-column bg-white"
                        v-model="selected"
                        @input="FilterByAdded($event.target.value)">
                        <option>Newest Added</option>
                        <option>Oldest Added</option>
                    </select><br />
                    <select
                        class="text-lg w-40 mt-2 break-after-column bg-white"
                        v-bind:value="location"
                        @input="FilterByLocation($event.target.value)">
                        <option>All Regions</option>
                        <option>Northland, New Zealand</option>
                        <option>Auckland, New Zealand</option>
                        <option>Waikato, New Zealand</option>
                        <option>Bay of Plenty, New Zealand</option>
                        <option>Gisborne, New Zealand</option>
                        <option>Hawke's Bay, New Zealand</option>
                        <option>Taranaki, New Zealand</option>
                        <option>Manawatū-Whanganui, New Zealand</option>
                        <option>Wellington, New Zealand</option>
                        <option>Tasman, New Zealand</option>
                        <option>Nelson, New Zealand</option>
                        <option>Malborough, New Zealand</option>
                        <option>West Coast, New Zealand</option>
                        <option>Canterbury, New Zealand</option>
                        <option>Otago, New Zealand</option>
                        <option>Southland, New Zealand</option>
                    </select><br />
                    <select
                        class="text-lg mt-2 break-after-column bg-white"
                        v-bind:value="category"
                        @input="FilterByCategory($event.target.value)">
                        <option>All Categories</option>
                        <option>Skatepark</option>
                        <option value="Spot">Street Spot</option>
                    </select><br />
                </div>
            </div>
            <button
                v-if="IsLoggedIn === true"
                @click="SetPage('AddSkateparkPage')"
                class="py-2 px-4 mx-2 mt-5 bg-blue-500 rounded-lg text-white font-semibold hover:bg-blue-600">
                Add Skatepark / Spot
            </button>
        </div>
        <div v-if="loading === true">
            <div class="mt-10 fade">
                <div class="shadow text-left bg-white mb-14 ROUNDED max-w-4xl">
                    <img
                        src="data;,"
                        alt=""
                        class="rounded-t h-60 w-full object-cover bg-gray-400"
                    />
                    <!--Card Header-->
                    <header class="text-xl font-extrabold p-3 pl-2 mt-3 ml-4 bg-gray-400 rounded-lg w-20"></header>
                    <header class="text-xl font-extrabold p-3 pl-2 mt-1 ml-4 bg-gray-400 rounded-lg w-64"></header>
                    <!--Card Footer-->
                    <footer class="text-left py-3 px-8 text-gray-500">
                        <button
                        class="py-2 px-4 mt-5 rounded-lg text-gray font-semibold bg-gray-400">
                        Find Out More
                        </button>
                    </footer>
                </div>
            </div>
        </div>
        <AsyncSkateparks v-if="loading === false" @FindMore='FindMore($event)' :posts="posts"/>
    </center>
</template>

<script>
import { defineAsyncComponent } from 'vue';
import axios from 'axios';

const AsyncSkateparks = defineAsyncComponent(() => import('./Skateparks.vue'));

export default {
    props: ['IsLoggedIn'],
    data() {
        return {
            posts: [],
            errors: [],
            selected: 'Newest Added',
            location: 'All Regions',
            category: 'All Categories',
            openOptions: false,
            loading: true,
        };
    },
    components: {
        AsyncSkateparks,
    },
    methods: {
        SetPage(page) {
            this.isOpen = false;
            this.$emit('SetPage', page);
        },
        changeOptions() {
            this.openOptions = !this.openOptions;
        },
        FilterByAdded(selected) {
            this.selected = selected;
            this.posts.reverse();
        },
        FindMore(id) {
            if (window.localStorage.authToken) {
                this.$emit('SetPage', 'GetSkateparkPage');
                this.$emit('FindMore', id);
            } else {
                this.$emit('SetPage', 'LoginPage');
            }
        },
        FilterByCategory(category) {
            this.category = category;
            if (category === 'All Categories') {
                axios.get('/api/skateparks')
                .then((response) => {
                    this.posts = response.data;
                    this.posts.reverse();
                })
                .catch((error) => {
                    this.errors.push(error);
                });
            } else {
                axios({
                    method: 'get',
                    url: `/api/skateparks/category/${category}`,
                })
                .then((response) => {
                    this.posts = response.data;
                    this.posts.reverse();
                })
                .catch((error) => {
                    this.errors.push(error);
                });
            }
        },
        FilterByLocation(location) {
            this.location = location;
            if (location === 'All Regions') {
                axios.get('/api/skateparks')
                .then((response) => {
                    this.posts = response.data;
                    this.posts.reverse();
                })
                .catch((error) => {
                    this.errors.push(error);
                });
            } else {
                axios({
                    method: 'post',
                    url: '/api/skateparks/location',
                    data: {
                        location,
                    },
                })
                .then((response) => {
                    this.posts = response.data;
                    this.posts.reverse();
                })
                .catch((error) => {
                    this.errors.push(error);
                });
            }
        },
    },

    created() {
        axios.get('/api/skateparks')
        .then((response) => {
            this.posts = response.data;
            this.posts.reverse();
            this.loading = false;
        })
        .catch((error) => {
            this.errors.push(error);
        });
    },
};
</script>

<style>
    .fadeFilter {
        animation: fadeInAnimation ease 0.7s;
        animation-iteration-count: 1;
        animation-fill-mode: forwards;
    }
    .fade {
        animation: fadeInAnimation ease 1s;
        animation-iteration-count: 1;
        animation-fill-mode: forwards;
    }
    @keyframes fadeInAnimation {
        0% {
            opacity: 0;
        }
        100% {
            opacity: 1;
        }
    }
</style>
