<template>
    <div class="box">
        <ul id="chat">
            <li class="text-inner" :class="item.idFrom === currentUserId ? 'you' : 'me'" v-for="item in listMessage"
                :key="item.id">
                <div class="entete">
                    <span class="status green"></span>
                    <h2>Vincent</h2>
                    <h3>10:12AM, Today</h3>
                </div>
                <div class="triangle"></div>
                <div class="message">
                    Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Aenean commodo ligula eget dolor.
                </div>
            </li>
            <li class="me">
                <div class="entete">
                    <h3>10:12AM, Today</h3>
                    <h2>Vincent</h2>
                    <span class="status blue"></span>
                </div>
                <div class="triangle"></div>
                <div class="message">
                    Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Aenean commodo ligula eget dolor.
                </div>
            </li>
        </ul>
    </div>
</template>

<script>
export default {
    props: ["currentPeerUser"],
    data() {
        return {
            currentUserName: localStorage.getItem("name"),
            currentUserId: localStorage.getItem("id"),
            currentUserPhoto: localStorage.getItem("photoURL"),
            inputValue: "",
            photoURL: localStorage.getItem("photoURL"),
            listMessage: [],
            groupChatId: null
        }
    },
    watch: {
        currentPeerUser: function (newVal, oldVal) {
            if (newVal !== oldVal) {
                this.getMessages();
            }
        }
    },
    methods: {
        sendMessage(content) {
            if (content.trim() === "") {
                return;
            }
            const timestamp = moment()
                .valueOf()
                .toString();
            const message = {
                id: timestamp,
                idFrom: this.currentUserId,
                idTo: this.currentPeerUser.id,
                timestamp: timestamp,
                content: content.trim()
            };
            firebase
                .firestore()
                .collection("Messages")
                .doc(this.groupChatId)
                .collection(this.groupChatId)
                .doc(timestamp)
                .set(message)
                .then(() => {
                    this.inputValue = "";
                });
        },
        getMessages() {
            console.log("call coming");
            this.listMessage = [];
            // here we are creating a room
            let groupChatId = `${this.currentPeerUser.id} + ${this.currentUserId}`;
            firebase
                .firestore()
                .collection("Messages")
                .doc(groupChatId)
                .collection(groupChatId)
                .onSnapshot(Snapshot => {
                    if (Snapshot.docChanges().length > 0) {
                        this.groupChatId = groupChatId;
                        Snapshot.docChanges().forEach(res => {
                            this.listMessage.push(res.doc.data());
                        });
                    } else {
                        console.log("call cmng 1");
                        this.groupChatId = `${this.currentUserId} + ${this.currentPeerUser.id}`;
                        firebase
                            .firestore()
                            .collection("Messages")
                            .doc(this.groupChatId)
                            .collection(this.groupChatId)
                            .onSnapshot(Snapshot => {
                                Snapshot.docChanges().forEach(res => {
                                    console.log("res", res);
                                    if (res.type === "added") {
                                        this.listMessage.push(res.doc.data());
                                    }
                                });
                            });
                    }
                });
        }
    },
    mounted() {
        this.getMessages();
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#chat {
    padding-left: 0;
    margin: 0;
    list-style-type: none;
    overflow-y: scroll;
    height: 535px;
    border-top: 2px solid #fff;
    border-bottom: 2px solid #fff;
}

#chat li {
    padding: 10px 30px;
}

#chat h2,
#chat h3 {
    display: inline-block;
    font-size: 13px;
    font-weight: normal;
}

#chat h3 {
    color: #bbb;
}

#chat .entete {
    margin-bottom: 5px;
}

#chat .message {
    padding: 20px;
    color: #fff;
    line-height: 25px;
    max-width: 90%;
    display: inline-block;
    text-align: left;
    border-radius: 5px;
}

#chat .me {
    text-align: right;
}

#chat .you .message {
    background-color: #58b666;
}

#chat .me .message {
    background-color: #6fbced;
}

#chat .triangle {
    width: 0;
    height: 0;
    border-style: solid;
    border-width: 0 10px 10px 10px;
}

#chat .you .triangle {
    border-color: transparent transparent #58b666 transparent;
    margin-left: 15px;
}

#chat .me .triangle {
    border-color: transparent transparent #6fbced transparent;
    margin-left: 375px;
}
</style>
