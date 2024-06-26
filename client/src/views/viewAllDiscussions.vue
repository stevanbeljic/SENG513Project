<script setup>
    import "../assets/footer.css";
    import "../assets/header.css";
    import "../assets/discussion.css";
    import "../assets/viewAllDiscussions.css"
    import navbar from "../components/navbarView.vue";
    import bottomNavbar from "../components/bottomNavbarView.vue";
    import { onMounted, ref } from "vue";
    import { useRoute } from "vue-router";
    import router from "@/router";

    const game = ref([]);
    const likedDiscussions = ref([]);
    const route = useRoute();
    let loggedIn = ref(sessionStorage.getItem('loggedIn') === 'true');
    let loggedInId = ref(sessionStorage.getItem('loggedInId'));

    const fetchGame = async () => {
        const response = await fetch('http://localhost:8080/game/getGameById?id='+route.params.gameid);
        const games = await response.json();
        game.value = games[0];
        game.value.discussions = await fetchDiscussion(game.value.id);
    };

    const fetchDiscussion = async (id) => {
        const response = await fetch('http://localhost:8080/discussion/getDiscussionsByGame?id='+id);
        return await response.json();
    }

    const fetchLikedDiscussions = async() => {
        const response = await fetch('http://localhost:8080/discussion/likedDiscussionsByUserID?userid=' + loggedInId.value);
        const data = await response.json();
        likedDiscussions.value = await data;
    }

    const handleDiscussionClick = (id) =>{
        router.push({ name: 'discussionpost', params: { discussionId: id }})
    }

    const handleNewDiscussion = (id) => {
        if(loggedIn.value){
            router.push({ name: 'addDiscussion', params: {gameid: id} });
        } else {
            router.push('/login');
        }
    }

    const isLiked = (discussion_id) => {
        if (loggedIn.value != true) {
            return false;
        } else {
            return likedDiscussions.value.some(d => d.discussion_id == discussion_id);
        }
    };

    const toggleLike = async(event, discussion_id) => {
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
  };

    onMounted(async () =>{
        try {
            await fetchGame();
            await fetchLikedDiscussions();
        } catch (error) {
            console.log(error);
        }
    });

    defineExpose({handleDiscussionClick, game, handleNewDiscussion, loggedIn});
</script>
<template>
    <head>
        <link href='https://fonts.googleapis.com/css?family=Kanit' rel='stylesheet'>
    </head>
    <header class = "header-section">
        <navbar></navbar>
    </header>

    <body>
        <div class="discussion-body-container">
            <div v-if="game" class="discussion-card">
                <div id="viewDiscGame" class="discussion-game-section">
                    <img :src="'http://localhost:8080' + game.thumbnail" alt="Image Unavailable"/>
                    <h1 v-text="game.name+' - Discussions'"></h1>
                </div>
                <div class="discussion-list-section">
                    <div v-if="loggedIn" class="discussion-list-box" id="postButtonDiv"><button id="postButton" v-on:click="handleNewDiscussion(game.id)">Start a New Discussion</button></div>
                    <div v-if="game.discussions && game.discussions.length != 0">
                        <div class="discussion-list-box"  v-for="discussion in game.discussions" :key="discussion.discussion_id">
                            <div>
                                <h3 v-on:click="handleDiscussionClick(discussion.discussion_id)" v-text="discussion.title"></h3>
                            </div>
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
                </div>
            </div>
        </div>
    </body>
    <footer class="footer-section">
        <bottomNavbar></bottomNavbar>
    </footer>
</template>