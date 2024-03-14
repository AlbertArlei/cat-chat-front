<template>
    <div class="flex flex-col justify-center items-center gap-2 relative">
        <canvas id="game" class="bg-zinc-950"></canvas>
        <div class="absolute top-2 left-2">
            <div class="flex relative">
                <p class="absolute top-7 left-52 text-xs">{{ inputMessage.length }}/40</p>
                <input class="h-11 outline-none border-none bg-zinc-100 p-1" type="text" v-model="inputMessage"
                    maxlength="40" @focus="inputActive = true" @blur="inputActive = false">
                <input class="bg-zinc-100 cursor-pointer outline-none p-2" type="button" value="Enviar"
                    @click="sendMessage">
            </div>
        </div>
    </div>
</template>

<script lang="ts">
import io from "socket.io-client";

interface IPlayerData {
    id: string;
    x: number;
    y: number;
    input: string;
}

interface IMessageData {
    id: string;
    message: string;
}

export default {
    setup() {

    },
    data() {
        return {
            room: "",
            socket: io('https://cat-chat-backend.vercel.app/'),
            socketId: "",
            playerData: {} as IPlayerData,
            enemyData: {} as IPlayerData,
            playerMessageData: {} as IMessageData,
            enemyMessageData: {} as IMessageData,
            inputMessage: '',
            inputActive: false,
        };
    },
    methods: {
        draw() {
            this.ground();
            this.player();
            this.enemy();
        },
        async ground() {
            const canvas: HTMLCanvasElement | null = document.getElementById("game") as HTMLCanvasElement;
            const ctx = canvas.getContext("2d");
            if (ctx) ctx.fillStyle = '#37c234';
            ctx?.fillRect(0, 0, 1980, 700);
            if (ctx) ctx.fillStyle = '#913a20';
            ctx?.fillRect(0, 700, 1980, 380);
            if (ctx) ctx.fillStyle = '#218a1e';
            ctx?.fillRect(0, 700, 1980, 16);
        },
        async player() {
            const canvas: HTMLCanvasElement | null = document.getElementById("game") as HTMLCanvasElement;
            const ctx = canvas.getContext("2d");
            const img = new Image();

            switch (true) {
                case this.playerData.input === "w":
                    img.src = "https://i.imgur.com/zB2DVf3.png"
                    break;
                case this.playerData.input === "a":
                    img.src = "https://i.imgur.com/vO5Y6h0.png"
                    break;
                case this.playerData.input === "s":
                    img.src = "https://i.imgur.com/7KX2QJL.png"
                    break;
                case this.playerData.input === "d":
                    img.src = "https://i.imgur.com/SAwEAUS.png"
                    break;
                case this.playerData.input === "wa":
                    img.src = "https://i.imgur.com/zB2DVf3.png"
                    break;
                case this.playerData.input === "wd":
                    img.src = "https://i.imgur.com/zB2DVf3.png"
                    break;
                case this.playerData.input === "sa":
                    img.src = "https://i.imgur.com/7KX2QJL.png"
                    break;
                case this.playerData.input === "sd":
                    img.src = "https://i.imgur.com/7KX2QJL.png"
                    break;
            }
            img.onload = () => {
                ctx?.drawImage(img, this.playerData.x, this.playerData.y);
                this.playerMessage()
            }

        },
        enemy() {
            const canvas: HTMLCanvasElement | null = document.getElementById("game") as HTMLCanvasElement;
            const ctx = canvas.getContext("2d");
            const img = new Image();

            switch (true) {
                case this.enemyData.input === "w":
                    img.src = "https://i.imgur.com/bkinXvO.png"
                    break;
                case this.enemyData.input === "a":
                    img.src = "https://i.imgur.com/CX7rRkx.png"
                    break;
                case this.enemyData.input === "s":
                    img.src = "https://i.imgur.com/OfS5gH4.png"
                    break;
                case this.enemyData.input === "d":
                    img.src = "https://i.imgur.com/deMxdhh.png"
                    break;
                case this.enemyData.input === "wa":
                    img.src = "https://i.imgur.com/bkinXvO.png"
                    break;
                case this.enemyData.input === "wd":
                    img.src = "https://i.imgur.com/bkinXvO.png"
                    break;
                case this.enemyData.input === "sa":
                    img.src = "https://i.imgur.com/OfS5gH4.png"
                    break;
                case this.enemyData.input === "sd":
                    img.src = "https://i.imgur.com/OfS5gH4.png"
                    break;
            }
            img.onload = () => {
                ctx?.drawImage(img, this.enemyData.x, this.enemyData.y);
                this.enemyMessage();
            }

        },
        keyboardInputs() {
            const keysPressed: { [key: string]: boolean } = {};
            const move = 6;

            document.addEventListener("keydown", (event) => {
                const x = this.playerData.x;
                const y = this.playerData.y;
                const key = event.code;
                if (key === 'Enter') this.sendMessage();
                if (this.inputActive) return;
                let input = "";
                keysPressed[event.code] = true;
                let newX = x;
                let newY = y;
                switch (true) {
                    case keysPressed["KeyW"] && keysPressed["KeyD"]:
                        newX = x + move;
                        newY = y - move;
                        input = "wd"
                        if (newX > 1890 || newY < 0) {
                            this.socket.emit('player', { newX: x, newY: y, input });
                            break;
                        }
                        this.socket.emit('player', { newX, newY, input });
                        break;
                    case keysPressed["KeyW"] && keysPressed["KeyA"]:
                        newX = x - move;
                        newY = y - move;
                        input = "wa";
                        if (newX <= 0 || newY <= 0) {
                            this.socket.emit('player', { newX: x, newY: y, input });
                            break;
                        }
                        this.socket.emit('player', { newX, newY, input });
                        break;
                    case keysPressed["KeyS"] && keysPressed["KeyD"]:
                        newX = x + move;
                        newY = y + move;
                        input = "sd";
                        if (newX > 1890 || newY > 666) {
                            this.socket.emit('player', { newX: x, newY: y, input });
                            break;
                        }
                        this.socket.emit('player', { newX, newY, input });
                        break;
                    case keysPressed["KeyS"] && keysPressed["KeyA"]:
                        newX = x - move;
                        newY = y + move;
                        input = "sa";
                        if (newX <= 0 || newY > 666) {
                            this.socket.emit('player', { newX: x, newY: y, input });
                            break;
                        }
                        this.socket.emit('player', { newX, newY, input });
                        break;
                    case key === "KeyW":
                        newY = y - move;
                        input = "w";
                        if (newY <= 0) {
                            this.socket.emit('player', { newX, newY: y, input });
                            break;
                        }
                        this.socket.emit('player', { newX, newY, input });
                        break;
                    case key === "KeyA":
                        newX = x - move;
                        input = "a";
                        if (newX <= 0) {
                            this.socket.emit('player', { newX: x, newY, input });
                            break;
                        }
                        this.socket.emit('player', { newX, newY, input });
                        break;
                    case key === "KeyS":
                        newY = y + move;
                        input = "s";
                        if (newY >= 666) {
                            this.socket.emit('player', { newX, newY: y, input });
                            break;
                        }
                        this.socket.emit('player', { newX, newY, input });
                        break;
                    case key === "KeyD":
                        newX = x + move;
                        input = "d";
                        if (newX > 1890) {
                            this.socket.emit('player', { newX: x, newY, input });
                            break;
                        }
                        this.socket.emit('player', { newX, newY, input });
                        break;
                }
            });
            document.addEventListener('keyup', (event) => {
                delete keysPressed[event.code];
            });
        },
        mobileInputs() {

        },
        sendMessage() {
            this.socket.emit('message', { message: this.inputMessage });
            this.inputMessage = '';
        },
        serverData(data: IPlayerData) {
            switch (true) {
                case data.id === this.socketId:
                    this.playerData = data;
                    this.draw()
                    break;
                default:
                    this.enemyData = data
                    this.draw()
                    break;
            }
        },
        playerMessage() {
            if (this.playerMessageData.id === undefined) return;
            const canvas: HTMLCanvasElement | null = document.getElementById("game") as HTMLCanvasElement;
            const ctx = canvas.getContext("2d");
            if (ctx) ctx.font = 'bold 10px Arial';
            if (ctx) ctx.fillStyle = 'white';
            if (ctx) ctx.textAlign = 'center';
            if (ctx) ctx.textBaseline = 'middle'
            const messageWidth = ctx?.measureText(this.playerMessageData.message.toUpperCase()).width;
            if (messageWidth) {
                ctx?.fillRect(this.playerData.x + 16 - ((messageWidth + 2) / 2), this.playerData.y - 16, messageWidth + 2, 12);
                if (ctx) ctx.fillStyle = 'black';
                ctx?.fillText(this.playerMessageData.message.toUpperCase(), this.playerData.x + 16, this.playerData.y - 9);
            }
        },
        enemyMessage() {
            console.log('a')
            if (this.enemyMessageData.id === undefined) return;
            const canvas: HTMLCanvasElement | null = document.getElementById("game") as HTMLCanvasElement;
            const ctx = canvas.getContext("2d");
            if (ctx) ctx.font = 'bold 10px Arial';
            if (ctx) ctx.fillStyle = 'white';
            if (ctx) ctx.textAlign = 'center';
            if (ctx) ctx.textBaseline = 'middle'
            const messageWidth = ctx?.measureText(this.enemyMessageData.message.toUpperCase()).width;
            if (messageWidth) {
                ctx?.fillRect(this.enemyData.x + 16 - ((messageWidth + 2) / 2), this.enemyData.y - 16, messageWidth + 2, 12);
                if (ctx) ctx.fillStyle = 'black';
                console.log(this.enemyMessageData.message.toUpperCase())
                ctx?.fillText(this.enemyMessageData.message.toUpperCase(), this.enemyData.x + 16, this.enemyData.y - 9);
            }
        },
        messageData(data: IMessageData) {
            switch (true) {
                case data.id === this.socketId:
                    this.playerMessageData = data;
                    this.draw();
                    break;
                default:
                    this.enemyMessageData = data;
                    this.draw();
                    break;
            }
        },
        socketInit() {
            this.socket.on("connect", () => {
                if (this.socket.id !== undefined)
                    this.socketId = this.socket.id;
            });
            this.socket.on('playerData', data => {
                this.serverData(data)
            });
            this.socket.on('messageData', data => {
                this.messageData(data)
            });

        },
        getScreenSize() {
            const width = window.innerWidth;
            const height = window.innerHeight;
            const canvas = document.getElementById('game') as HTMLCanvasElement;
            if (width > 1920 || height > 1080) {
                canvas.width = 1920;
                canvas.height = 1080;
                return
            }
            canvas.width = width;
            canvas.height = height;

        }
    },
    async mounted() {
        this.getScreenSize();
        this.socketInit();
        this.keyboardInputs();
        const sf = await fetch('https://cat-chat-backend.vercel.app/', {method: "get"});
console.log(sf)
    },
};
</script>
