<template>
    <div class="wrapper">
        <h1>Simon The Game</h1>
        <div class="container">
            <div
                    class="game__container"
                    v-bind:class="{ gameOn: !userTurn}"
            >
                <div class="game">
                    <div
                            class="red"
                            id="1"
                            v-on:click="clickedBox(1)"
                            v-bind:class="[{ active: isBoxOneActive}, {gameOn: !userTurn}]"
                    >
                    </div>
                    <div
                            class="blue"
                            id="2"
                            v-on:click="clickedBox(2)"
                            v-bind:class="[{ active: isBoxTwoActive}, {gameOn: !userTurn}]"
                    >
                    </div>
                    <div
                            class="green"
                            id="3"
                            v-on:click="clickedBox(3)"
                            v-bind:class="[{ active: isBoxThreeActive}, {gameOn: !userTurn}]"
                    >
                    </div>
                    <div
                            class="yellow"
                            id="4"
                            v-on:click="clickedBox(4)"
                            v-bind:class="[{ active: isBoxFourActive}, {gameOn: !userTurn}]"
                    >
                    </div>
                </div>
            </div>
            <div class="game__info">
                <div class="round">
                    Раунд: {{round}}
                </div>
                <div class="game__button">
                    <button
                            class="btn"
                            @click="changeState()">
                        {{state}}
                    </button>
                </div>
                <div v-if="isUserFail">
                    <p>Вы проиграли на {{roundFail}} раунде!</p>
                </div>
                <div class="radio">
                    <h3>Выберите уровень сложности</h3>
                    <div class="radio__btn">
                        <input type="radio" id="easy" value="easy" v-model="level" checked/>
                        <label for="easy">Легкий</label>
                    </div>
                    <div class="radio__btn">
                        <input type="radio" id="medium" value="medium" v-model="level"/>
                        <label for="medium">Средний</label>
                    </div>
                    <div class="radio__btn">
                        <input type="radio" id="hard" value="hard" v-model="level"/>
                        <label for="hard">Тяжелый</label>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import audio_0 from '../assets/sounds/0.ogg'
    import audio_1 from '../assets/sounds/1.ogg'
    import audio_2 from '../assets/sounds/2.ogg'
    import audio_3 from '../assets/sounds/3.ogg'

    let timer;

    export default {
        name: "Simon",

        data() {
            return {
                round: 0,
                roundFail:0,
                helper: 0,
                state: 'Начать игру',
                steps: [],
                isGameOn: false,
                userTurn: false,
                isUserFail: false,
                isBoxOneActive: false,
                isBoxTwoActive: false,
                isBoxThreeActive: false,
                isBoxFourActive: false,
                level: 'easy',
                audiosArray: [
                    audio_0,
                    audio_1,
                    audio_2,
                    audio_3
                ]
            }
        },
        computed: {
            timeout() {
                switch (this.level) {
                    case 'easy':
                        return 1.5;
                    case 'medium':
                        return 1;
                    case 'hard':
                        return 0.4;
                    default:
                        return 1.5;
                }
            }
        },
        methods: {
            changeState() {
                if ((this.state === 'Начать игру') || (this.state === 'Попробовать еще раз') ) {
                    this.state = 'Прекратить игру';
                    this.isGameOn = true;
                    this.isUserFail = false;
                    this.computerTurn(); // запускает игру и инициализирует ход компьютера
                } else {
                    this.resetGame(); // сбрасывает игру
                }
            },
            getRandomNumber() {
                return Math.floor(Math.random() * 4 + 1); // генерирует случайное число от 1 до 4
            },
            computerTurn() {
                this.helper = 0;
                this.round++;
                this.steps.push(this.getRandomNumber()); // добавляет в массив случайно сгенерированные числа
                this.showPattern(() => {
                }); // запускает воспроизведение "копьютером" массива сгенерированных чисел
            },
            showPattern(callback) {
                let self = this;
                let i = 0;
                timer = setInterval(() => {
                    if (i >= self.steps.length) {
                        self.stopInterval(); // прерывает отрабатывание клика, если пройдена вся длина массива
                        this.userTurn = true; // после окончания кликов "компьютера" разрешается делать ход user'y
                    } else {
                        self.clickEffect(self.steps[i]); // инициирует отработку клика в массиве "компьютера" поочередно
                        i++;
                    }
                }, this.timeout * 1000); // в зависимости от выбранного уровня сложности, устанавливается время смены клика
                this.userTurn = false;
                callback(); // возвращает callback-функцию
            },
            playAudio(index) {
                const audio = new Audio(this.audiosArray[index]);
                audio.play(); // воспроизводит массив аудио
            },
            clickEffect(boxNum) {
                let self = this;
                switch (boxNum) {
                    case 1:
                        this.isBoxOneActive = true;
                        this.playAudio(0);
                        setTimeout(() => {
                            self.isBoxOneActive = false;
                        }, 500);
                        break; // отражает клик на красную кнопку
                    case 2:
                        this.isBoxTwoActive = true;
                        this.playAudio(1);
                        setTimeout(() => {
                            self.isBoxTwoActive = false;
                        }, 500);
                        break; // отражает клик на синюю кнопку
                    case 3:
                        this.isBoxThreeActive = true;
                        this.playAudio(2);
                        setTimeout(() => {
                            self.isBoxThreeActive = false;
                        }, 500);
                        break; // отражает клик на зеленую кнопку
                    case 4:
                        this.isBoxFourActive = true;
                        this.playAudio(3);
                        setTimeout(() => {
                            self.isBoxFourActive = false;
                        }, 500);
                        break; // отражает клик на желтую кнопку
                }
            },
            stopInterval() {
                clearInterval(timer); // метод для сбрасывания интервала клика
            },
            clickedBox(boxNum) {
                let self = this;
                if (this.userTurn === false) {
                    return; // не позволяет user'у производить клик, пока не началась игра
                }
                this.clickEffect(boxNum); // производит клик, если нажимает user
                if (!(this.steps[this.helper] === boxNum)) {
                    self.processGameOver(); // сравнение последовательности кликов user'a со случайно сгенерированным массивом, если user допускает ошибку, то user проиграл
                } else {
                    if (this.helper === this.steps.length - 1) {
                        setTimeout(() => {
                            self.userTurn = false;
                            self.computerTurn(); // если user нажал правильно в раунде, то раунд заканчивается и начинается следующий с хода компьютера
                        }, 1000);
                    } else {
                        this.helper++; // увеличавет helper до тех пора пока user не ответит правильно
                    }
                }
            },
            processGameOver() {
                this.roundFail = this.round;
                this.isUserFail = true; // если user допустил ошибку
                this.resetGame();
            },
            resetGame() {
                this.round = 0;
                this.userTurn = false;
                if(this.isUserFail){
                    this.state = 'Попробовать еще раз'
                } else {this.state = 'Начать игру'}
                this.isGameOn = false;
                this.steps = [];
                this.helper = 0;
            }, // сбрасывание игры
        }
    }
</script>

<style lang="scss" scoped>
    .wrapper {
        min-height: 100vh;
        box-sizing: border-box;
        padding: 40px;
        h1 {
            display: flex;
            justify-content: center;
            font-size: 46px;
            padding: 25px;
            margin: 25px;
        }
    }

    .container {
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .game__container {
        min-width: 300px;
        max-width: 300px;
        max-height: 300px;
        min-height: 300px;
        border: 4px solid white;
        box-shadow: 0 0 10px 5px rgba(0, 0, 0, 0.55);
        border-radius: 50%;
        cursor: pointer;
        margin: 50px;
        .gameOn {
            cursor: default;
            &:hover{
                opacity: 0.6 !important;
            }
        }
        .game {
            border-radius: inherit;
            width: 100%;
            height: 100%;
            .red {
                background: red;
                border-radius: inherit;
                width: 300px;
                height: 300px;
                clip-path: polygon(50% 0%, 50% 50%, 0 50%, 0 0);
                position: absolute;
                transition: 0.25s;
                opacity: 0.6;
                &:hover {
                    opacity: 1;
                    @media (hover: none) {
                        opacity: 0.6;
                    }
                }
            }
            .blue {
                background: blue;
                border-radius: inherit;
                width: 300px;
                height: 300px;
                clip-path: polygon(100% 0%, 100% 50%, 50% 50%, 50% 0%);
                position: absolute;
                transition: 0.25s;
                opacity: 0.6;
                &:hover {
                    opacity: 1;
                    @media (hover: none) {
                        opacity: 0.6;
                    }
                }
            }
            .green {
                background: green;
                border-radius: inherit;
                width: 300px;
                height: 300px;
                clip-path: polygon(50% 50%, 50% 100%, 100% 100%, 100% 50%);
                position: absolute;
                transition: 0.25s;
                opacity: 0.6;
                &:hover {
                    opacity: 1;
                    @media (hover: none) {
                        opacity: 0.6;
                    }
                }
            }
            .yellow {
                background: #f7ec14;
                border-radius: inherit;
                width: 300px;
                height: 300px;
                clip-path: polygon(0 50%, 0 100%, 50% 100%, 50% 50%);
                position: absolute;
                transition: 0.25s;
                opacity: 0.6;
                &:hover {
                    opacity: 1;
                    @media (hover: none) {
                        opacity: 0.6;
                    }
                }
            }
            .active {
                opacity: 1 !important;
            }
        }
    }

    .game__info {
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
        .round {
            padding-bottom: 15px;
            font-size: 24px;
        }
        .btn {
            width: 200px;
            height: 70px;
            font-weight: 600;
            font-size: 20px;
            padding: 10px 16px;
            margin: 10px 0;
            color: #ffffff;
            background: #22b2ea;
            border: 1px solid #22b2ea;
            border-radius: 18px;
            cursor: pointer;
            transition: background 0.2s ease-in-out;
            &:hover {
                background: lighten(#22b2ea, 5%);
            }
            &:active {
                background: darken(#22b2ea, 5%)
            }
        }
        p{
            color: red;
            font-size: 20px;
            font-weight: 600;
        }
        .radio {
            h3 {
                margin: 15px 0;
            }
            .radio__btn {
                padding: 5px;
                label{
                    padding-left: 5px;
                }
            }
        }
    }
</style>