<script setup>
import "../assets/footer.css";
import "../assets/gameOverview.css";
import "../assets/header.css";
import "../assets/home.css";
import navbar from "../components/navbarView.vue";
import bottomNavbar from '@/components/bottomNavbarView.vue';
import { useRoute } from 'vue-router';
import { ref } from "vue";
import router from "@/router";
</script>
<script>
    let loggedInId = ref(sessionStorage.getItem('loggedInId'));
    let loggedIn = ref(sessionStorage.getItem('loggedIn') === 'true');
    let added = ref([]);
    let likedDiscussions = ref([])
    
    const updateSessionData = async () => {
        const route = useRoute();
        loggedIn.value = sessionStorage.getItem('loggedIn') === 'true';
        loggedInId.value = sessionStorage.getItem('loggedInId');
        const response = await fetch("http://localhost:8080/game/existsTopGame?userid="+loggedInId.value+"&gameid="+route.params.id, {method: "GET"});
        
        added.value = false;
        if (response.status == 200){
            const data = await response.json();
            if (data.length > 0){
                added.value = true;
            }
        }
        return;
    }; 

    const fetchLikedDiscussions = async() => {
        const response = await fetch('http://localhost:8080/discussion/likedDiscussionsByUserID?userid=' + loggedInId.value);
        const data = await response.json();
        likedDiscussions.value = await data;
    };

    export default {
        setup(){
            const route = useRoute();
            return {route};
        },
        mounted() {
            updateSessionData();

            let bRoute = "http://localhost:8080/game/getGameById?id="+ this.$route.params.id; 
            fetch(bRoute, {method: "GET"})
            .then(res => res.json())
            .then(data => this.game = data[0])
            .catch(err => console.log(err.message));

            fetch("http://localhost:8080/discussion/getDiscussionsByGame?id="+ this.$route.params.id, {method: "GET"})
            .then(res => res.json())
            .then(data => {this.discussions = data
            console.log(data)})
            .catch(err => console.log(err.message));
            
            fetchLikedDiscussions();
        },
        data() {
            return{
                game: '',
                discussions: []
            }
        },
        methods: {
            handleDiscussionClick: function(id){
                console.log(id)
                router.push({ name: 'discussionpost', params: { discussionId: id }})
            },

            handleViewAll: function(id){
                router.push({ name: 'allDiscussions', params: {gameid: id} });
            },
            handleNewDiscussion: function(id) {
                if(loggedIn.value){
                    router.push({ name: 'addDiscussion', params: {gameid: id} });
                } else {
                    router.push('/login');
                }
            },
            addToTopGames: async function(gameid){
                const response = await fetch("http://localhost:8080/game/addTopGame?userid="+loggedInId.value+"&gameid="+gameid, {method: "POST"});
                if (response.status == 200){
                    alert("Added game");
                    window.location.reload();
                    return;
                } else {
                    alert("Unable to add to Top Games");
                    return;
                }
            },
            toggleLike: async function(event, discussion_id) {
                if (loggedIn.value != true) {
                    alert("You must be logged in to like discussions");
                } else {
                    // get the button that was pressed
                    const button = event.currentTarget;
                    // toggle the button
                    button.classList.toggle("liked-button");
                    button.classList.toggle("unliked-button");
                    if (button.classList.contains("unliked-button")) {
                        button.classList.remove("like-animation");
                        // if button is now unliked, remove from likes
                        const response = await fetch("http://localhost:8080/discussion/removeLikedDiscussion?userid=" + loggedInId.value 
                        +"&discussionid=" + discussion_id, { method: "POST" });
                        if (response.status != 200) {
                        alert("Liked discussion was not removed successfully.");
                        }
                    } else if (button.classList.contains("liked-button")) {
                        // trigger like animation
                        button.classList.add("like-animation");
                        // if button is now liked, add to likes
                        const response = await fetch("http://localhost:8080/discussion/addLikedDiscussion?userid=" + loggedInId.value 
                        +"&discussionid=" + discussion_id, { method: "POST" });
                        if (response.status != 200) {
                        alert("Liked discussion was not added successfully.");
                        }
                    }
                }               
            },
            isLiked: function(discussion_id) {
                if (loggedIn.value != true) {
                    return false;
                } else {
                    return likedDiscussions.value.some(d => d.discussion_id == discussion_id);
                }
            }
        }
    }
</script>
<template>
    <head>
        <link href='https://fonts.googleapis.com/css?family=Kanit' rel='stylesheet'>
    </head>
    <header class = "header-section">
        <navbar></navbar>
    </header>
    <div class="body-container">
        <div class="game-overview">
            <div id="cover-image">
                <img :src="'http://localhost:8080' + game.thumbnail" alt="Image Unavailable"/>
            </div>
            <div id="game-information">
                <div id="game-details">
                    <div id="basic-details">
                        <div id="game-publisher">
                            <h1 id="game-name">{{ game.name }}</h1>
                            <p id="publisher-name">By: {{ game.publisher }}</p>
                        </div>
                        <div>
                            <p id="game-categories">{{ game.genre }}</p>
                        </div>
                    </div>
                    <div id="advanced-details">
                        <div class="store-info">
                            <a href="https://www.apple.com/ca/app-store/">
                                <img id="app-store" src="../components/icons/app-store-icon.jpg"/>
                            </a>
                            <p>${{ game.appstoreprice }}</p>
                        </div>
                        <div class="store-info">
                            <a href="https://play.google.com/store/games?hl=en&gl=US">
                                <img id="play-store" src="../components/icons/playstore-icon.svg"/>
                            </a>
                            <p>${{ game.playstoreprice }}</p>
                        </div>
                    </div>
                </div>
                <div id="game-description">
                    <p>{{ game.description }}</p>
                    <button v-if="loggedIn && !added" id="addToTopGames" v-on:click="addToTopGames(game.id)" >Add to your Top Games</button>
                    <button v-else-if="loggedIn && added" id="addedToTopGames" disabled="true" v-on:click="addToTopGames(game.id)" >In your Top Games</button>
                </div>
            </div>
        </div>
        <div class="discussion-list-section" id="overview-discussion-section">
            <h1 id="discussion-label">Discussions Board</h1>
            <div v-if="discussions.length != 0">
                <div class="discussion-list-box" v-for="discussion in discussions.slice(0, 3)" :key="discussion.discussion_id">
                    <div v-on:click="handleDiscussionClick(discussion.discussion_id)" class="discussionTitle"><h3>{{ discussion.title }}</h3></div>
                    <div class="likes-container">
                    <button :class="[isLiked(discussion.discussion_id) ? 'liked-button' : 'unliked-button']" 
                    @click="toggleLike($event, discussion.discussion_id)">❤︎</button>
                    </div>
                </div>
            </div>
            <div class="discussion-list-box" v-else>
                <h3>Be the first to talk about {{ game.name }}!</h3>
                <div class="likes-container">
                    <button class="add-discussion-button" v-on:click="handleNewDiscussion(game.id)">+</button>
                </div>
            </div>
            <div class="discussion-view-all-section">
                <button class="discussion-view-all-button" v-on:click="handleViewAll(game.id)">↪ VIEW ALL</button>
            </div>
        </div>
    </div>
    <footer class="footer-section">
    <bottomNavbar></bottomNavbar>
  </footer>
</template>