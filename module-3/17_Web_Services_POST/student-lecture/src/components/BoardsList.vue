<template>
  <div id="sideNav">
    <h1>My Kanban Boards</h1>
    <div class="boards">
      <div class="status-message error" v-show="errorMsg !== ''">
        {{ errorMsg }}
      </div>
      <div class="loading" v-if="isLoading">
        <img src="../assets/ping_pong_loader.gif" />
      </div>
      <router-link
        :to="{ name: 'Board', params: { id: board.id } }"
        class="board"
        v-for="board in this.$store.state.boards"
        v-bind:key="board.id"
        v-bind:style="{ 'background-color': board.backgroundColor }"
        v-else
        tag="div"
      >
        {{ board.title }}
      </router-link>
      <button
        class="btn addBoard"
        v-if="!isLoading && !showAddBoard"
        v-on:click="showAddBoard = !showAddBoard"
      >
        Add Board
      </button>
      <form v-if="showAddBoard && !isLoading" v-on:submit.prevent="saveNewBoard"> <!--add !isLoading to v-if to make form disappear duringthe load-->
        <!--this form is only shown when button above clicked-->
        Board Title:
        <input type="text" class="form-control" v-model="newBoard.title" />
        <!--v-model b/c input-->
        Background Color:
        <input
          type="text"
          class="form-control"
          v-model="newBoard.backgroundColor"
        />
        <button class="btn btn-submit">Save</button>
        <button class="btn btn-cancel" v-on:click.prevent="cancelAddBoard">
          Cancel
        </button>
      </form>
    </div>
  </div>
</template>

<script>
import boardsService from "../services/BoardService";

export default {
  data() {
    return {
      isLoading: true,
      showAddBoard: false,
      newBoard: {
        title: "",
        backgroundColor: this.randomBackgroundColor(),
      },
      errorMsg: "",
    };
  },
  created() {
    this.retrieveBoards();
  },
  methods: {
    retrieveBoards() {
      boardsService.getBoards().then((response) => {
        this.$store.commit("SET_BOARDS", response.data);
        this.isLoading = false;

        if (
          this.$route.name == "Home" &&
          response.status === 200 &&
          response.data.length > 0
        ) {
          this.$router.push(`/board/${response.data[0].id}`);
        }
      });
    },
    cancelAddBoard() {
      this.showAddBoard = !this.showAddBoard;
      this.errorMsg = ""; //if cancel adding, whipe out error
    },
    saveNewBoard() {
      this.isLoading = true; //will cause page to black out & should gif
      //boardService.addBoard;    //then? hide the form
      boardsService
        .addBoard(this.newBoard)
        .then((response) => {
          if (response.status === 201) { //ensure that response returned is created to move forward with success
            this.retrieveBoards(); //redisplay boards; updates Vuex datastore
            this.showAddBoard = false;
            (this.newBoard = {
              //resetting newBoard object to show
              title: "",
              backgroundColor: this.backgroundColor(),
            }), //router push to new board
              (this.errorMsg = ""); //takes error message away when board successfully added
            this.$$router.push({
              name: "Board",
              params: { id: response.data.id },
            }); //id of new board given back & will send to board page for this new board
          }
        }) //catch?
        .catch((error) => {
          console.error(error);
          //print error message
          if (error.response) {
            this.errorMsg = `Error submitting new board. Response received was "${error.response.statusText}".`; //use back ticks b/c string interlopation
          } else if (error.request) {
            //request made but no response received; couldn't talk to server
            this.errorMsg =
              "Error submitting new board. Server could not be reached.";
          } else {
            this.errorMsg =
              "Error subbmitting new board. Request could not be made.";
          }
          this.isLoading = false; //removes ping pong gif
        });
    },
    randomBackgroundColor() {
      return "#" + this.generateHexCode();
    },
    generateHexCode() {
      var bg = Math.floor(Math.random() * 16777215).toString(16);
      if (bg.length !== 6) bg = this.generateHexCode();
      return bg;
    },
  },
};
</script>

<style scoped>
div#sideNav {
  height: 100%;
  width: 20%;
  position: fixed;
  z-index: 1;
  left: 0;
  top: 0;
  padding-top: 20px;
  padding-bottom: 20px;
  overflow-x: hidden;
  border-right: solid lightgrey 1px;
}

h1 {
  text-align: center;
}

.boards {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
}
.board {
  color: #f7fafc;
  border-radius: 10px;
  padding: 40px;
  flex: 1;
  margin: 10px;
  text-align: center;
  cursor: pointer;
  width: 60%;
}
.addBoard {
  color: #f7fafc;
  border-radius: 10px;
  background-color: #28a745;
  font-size: 16px;
  width: 60%;
  margin: 10px;
  padding: 20px;
  cursor: pointer;
}
.form-control {
  margin-bottom: 10px;
}
.btn {
  margin-bottom: 35px;
}
.loading {
  flex: 3;
}
.board:hover:not(.router-link-active),
.addBoard:hover {
  font-weight: bold;
}
.router-link-active {
  font-weight: bold;
  border: solid blue 5px;
}
</style>
