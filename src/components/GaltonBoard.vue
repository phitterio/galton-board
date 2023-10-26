<template>
    <section id="galtonCanvasContainer" :width="CANVAS_WIDTH" :height="CANVAS_HEIGHT"></section>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import Matter from "matter-js";

export default defineComponent({
    name: "GaltonBoard",
    setup() {
        const Engine = Matter.Engine;
        const Render = Matter.Render;
        const World = Matter.World;
        const Bodies = Matter.Bodies;
        const Composites = Matter.Composites;
        const Runner = Matter.Runner;
        return { Engine, Render, World, Bodies, Composites, Runner };
    },
    data() {
        return {
            CANVAS_WIDTH: 400,
            CANVAS_HEIGHT: 700,
            CANVAS_BACKGROUND_COLOR: "yellow",
            BORDER_WIDTH: 15,
            BORDER_COLOR: "red",
            RAMP_HEIGHT: 30,
            RAMP_CHANNEL_WIDTH: 30,
            RAMP_ANGLE: 22,
            RAMP_PERCECENTAGE_Y: 0.06,
            NAILS_COLOR: "blue",
            NAILS_Y_MARGIN_INIT: 45,
            NAILS_Y_MARGIN: 2.2,
            NAILS_RADIUS: 6,
            NAILS_X_MARGIN: 24,
            NAILS_STRUCTURE: [
                { id: 1, nails: 9 },
                { id: 2, nails: 12 },
                { id: 3, nails: 15 },
                { id: 4, nails: 14 },
                { id: 5, nails: 15 },
                { id: 6, nails: 14 },
                { id: 7, nails: 15 },
                { id: 8, nails: 14 },
                { id: 9, nails: 15 },
                { id: 10, nails: 14 },
                { id: 11, nails: 15 },
                { id: 12, nails: 14 },
                { id: 13, nails: 15 },
                { id: 14, nails: 14 },
                { id: 15, nails: 15 },
            ],
            WALLS_WIDTH: 8,
            WALLS_COLOR: "gray",
            BALLS_NUMBER: 200,
            BALLS_RADIUS: 3,
            BALLS_COLOR: "green",
            BALLS_FRICTION: 0.00001,
            BALLS_RESTITUTION: 0.7,
            BALLS_DENSITY: 0.001,
            BALLS_FRICTION_AIR: 0.042,
            BALLS_SLEEP_TRESHHOLD: 120,
            BALLS_MICROSECONDS_APPEAR: 50,
            isRunningSimulation: true,
        };
    },
    computed: {
        borderTop() {
            return this.createBorderRectangle(this.CANVAS_WIDTH / 2, 0 + this.BORDER_WIDTH / 2, this.CANVAS_WIDTH, this.BORDER_WIDTH);
        },
        borderBottom() {
            return this.createBorderRectangle(this.CANVAS_WIDTH / 2, this.CANVAS_HEIGHT - this.BORDER_WIDTH / 2, this.CANVAS_WIDTH, this.BORDER_WIDTH);
        },
        borderLeft() {
            return this.createBorderRectangle(0 + this.BORDER_WIDTH / 2, this.CANVAS_HEIGHT / 2, this.BORDER_WIDTH, this.CANVAS_HEIGHT);
        },
        borderRight() {
            return this.createBorderRectangle(this.CANVAS_WIDTH - this.BORDER_WIDTH / 2, this.CANVAS_HEIGHT / 2, this.BORDER_WIDTH, this.CANVAS_HEIGHT);
        },
        RAMP_WIDTH() {
            return (this.CANVAS_WIDTH - this.RAMP_CHANNEL_WIDTH) / 2;
        },
        rampLeftTopCoordinates() {
            const x1 = 0;
            const y1 = this.RAMP_PERCECENTAGE_Y * this.CANVAS_HEIGHT;
            const x2 = x1 + this.RAMP_WIDTH;
            const y2 = y1 + this.RAMP_WIDTH * Math.tan(this.RAMP_ANGLE * (Math.PI / 180));
            const x3 = x1 + this.RAMP_WIDTH;
            const y3 = y2 + this.RAMP_HEIGHT;
            const x4 = 0;
            const y4 = y1 + this.RAMP_HEIGHT;
            const centerX = this.RAMP_WIDTH / 2;
            const centerY = y1 + (this.RAMP_WIDTH * Math.tan(this.RAMP_ANGLE * (Math.PI / 180))) / 2 + this.RAMP_HEIGHT / 2;
            return { x1, y1, x2, y2, x3, y3, x4, y4, centerX, centerY };
        },
        rampLeftTop() {
            return this.createParallelogram({ x: this.rampLeftTopCoordinates.centerX, y: this.rampLeftTopCoordinates.centerY }, [
                { x: this.rampLeftTopCoordinates.x1, y: this.rampLeftTopCoordinates.y1 },
                { x: this.rampLeftTopCoordinates.x2, y: this.rampLeftTopCoordinates.y2 },
                { x: this.rampLeftTopCoordinates.x3, y: this.rampLeftTopCoordinates.y3 },
                { x: this.rampLeftTopCoordinates.x4, y: this.rampLeftTopCoordinates.y4 },
            ]);
        },
        rampLeftBottomCoordinates() {
            const x1 = this.rampLeftTopCoordinates.x2;
            const y1 = this.rampLeftTopCoordinates.y2;
            const x2 = this.rampLeftTopCoordinates.x3;
            const y2 = this.rampLeftTopCoordinates.y3;
            const x3 = 0;
            const y3 = y2 + this.RAMP_WIDTH * Math.tan(this.RAMP_ANGLE * (Math.PI / 180));
            const x4 = 0;
            const y4 = y3 - this.RAMP_HEIGHT;
            const centerX = this.RAMP_WIDTH / 2;
            const centerY = y1 + (this.RAMP_WIDTH * Math.tan(this.RAMP_ANGLE * (Math.PI / 180))) / 2 + this.RAMP_HEIGHT / 2;
            return { x1, y1, x2, y2, x3, y3, x4, y4, centerX, centerY };
        },
        rampLeftBottom() {
            return this.createParallelogram({ x: this.rampLeftBottomCoordinates.centerX, y: this.rampLeftBottomCoordinates.centerY }, [
                { x: this.rampLeftBottomCoordinates.x1, y: this.rampLeftBottomCoordinates.y1 },
                { x: this.rampLeftBottomCoordinates.x2, y: this.rampLeftBottomCoordinates.y2 },
                { x: this.rampLeftBottomCoordinates.x3, y: this.rampLeftBottomCoordinates.y3 },
                { x: this.rampLeftBottomCoordinates.x4, y: this.rampLeftBottomCoordinates.y4 },
            ]);
        },
        rampRightTopCoordinates() {
            const x1 = this.CANVAS_WIDTH;
            const y1 = this.RAMP_PERCECENTAGE_Y * this.CANVAS_HEIGHT;
            const x2 = x1 - this.RAMP_WIDTH;
            const y2 = y1 + this.RAMP_WIDTH * Math.tan(this.RAMP_ANGLE * (Math.PI / 180));
            const x3 = x1 - this.RAMP_WIDTH;
            const y3 = y2 + this.RAMP_HEIGHT;
            const x4 = this.CANVAS_WIDTH;
            const y4 = y1 + this.RAMP_HEIGHT;
            const centerX = this.CANVAS_WIDTH - this.RAMP_WIDTH / 2;
            const centerY = y1 + (this.RAMP_WIDTH * Math.tan(this.RAMP_ANGLE * (Math.PI / 180))) / 2 + this.RAMP_HEIGHT / 2;
            return { x1, y1, x2, y2, x3, y3, x4, y4, centerX, centerY };
        },
        rampRightTop() {
            return this.createParallelogram({ x: this.rampRightTopCoordinates.centerX, y: this.rampRightTopCoordinates.centerY }, [
                { x: this.rampRightTopCoordinates.x1, y: this.rampRightTopCoordinates.y1 },
                { x: this.rampRightTopCoordinates.x2, y: this.rampRightTopCoordinates.y2 },
                { x: this.rampRightTopCoordinates.x3, y: this.rampRightTopCoordinates.y3 },
                { x: this.rampRightTopCoordinates.x4, y: this.rampRightTopCoordinates.y4 },
            ]);
        },
        rampRightBottomCoordinates() {
            const x1 = this.rampRightTopCoordinates.x2;
            const y1 = this.rampRightTopCoordinates.y2;
            const x2 = this.rampRightTopCoordinates.x3;
            const y2 = this.rampRightTopCoordinates.y3;
            const x3 = x2 + this.RAMP_WIDTH;
            const y3 = y2 + this.RAMP_WIDTH * Math.tan(this.RAMP_ANGLE * (Math.PI / 180));
            const x4 = x3;
            const y4 = y3 - this.RAMP_HEIGHT;
            const centerX = this.CANVAS_WIDTH - this.RAMP_WIDTH / 2;
            const centerY = y1 + (this.RAMP_WIDTH * Math.tan(this.RAMP_ANGLE * (Math.PI / 180))) / 2 + this.RAMP_HEIGHT / 2;
            return { x1, y1, x2, y2, x3, y3, x4, y4, centerX, centerY };
        },
        rampRightBottom() {
            return this.createParallelogram({ x: this.rampRightBottomCoordinates.centerX, y: this.rampRightBottomCoordinates.centerY }, [
                { x: this.rampRightBottomCoordinates.x1, y: this.rampRightBottomCoordinates.y1 },
                { x: this.rampRightBottomCoordinates.x2, y: this.rampRightBottomCoordinates.y2 },
                { x: this.rampRightBottomCoordinates.x3, y: this.rampRightBottomCoordinates.y3 },
                { x: this.rampRightBottomCoordinates.x4, y: this.rampRightBottomCoordinates.y4 },
            ]);
        },
        NAILS_Y_INIT() {
            return this.RAMP_PERCECENTAGE_Y * this.CANVAS_HEIGHT + this.RAMP_WIDTH * Math.tan(this.RAMP_ANGLE * (Math.PI / 180)) + this.RAMP_HEIGHT + this.NAILS_Y_MARGIN_INIT;
        },
        nails() {
            const nails: Matter.Body[] = [];
            this.NAILS_STRUCTURE.forEach((row) => nails.push(...this.createRowNails(row)));
            return nails;
        },
        WALLS_N() {
            return this.NAILS_STRUCTURE[this.NAILS_STRUCTURE.length - 1]["nails"];
        },
        WALLS_Y0() {
            return this.NAILS_Y_INIT + 2 * this.NAILS_RADIUS * this.NAILS_STRUCTURE.length + this.NAILS_Y_MARGIN * (this.NAILS_STRUCTURE.length - 1);
        },
        WALLS_YF() {
            return this.CANVAS_HEIGHT - this.BORDER_WIDTH;
        },
        WALLS_CENTER_Y() {
            return (this.WALLS_YF + this.WALLS_Y0) / 2;
        },
        WALLS_DELTA() {
            return (this.WALLS_N - 1) / 2;
        },
        WALLS_X_MARGIN() {
            return this.NAILS_X_MARGIN;
        },
        WALLS_CENTER_X_0() {
            return this.CANVAS_WIDTH / 2 - this.WALLS_X_MARGIN * this.WALLS_DELTA;
        },
        walls() {
            return [...Array(this.WALLS_N).keys()].map((i) => {
                return this.Bodies.rectangle(this.WALLS_CENTER_X_0 + i * this.WALLS_X_MARGIN, this.WALLS_CENTER_Y, this.WALLS_WIDTH, this.WALLS_YF - this.WALLS_Y0, {
                    isStatic: true,
                    sleepThreshold: Infinity,
                    render: {
                        fillStyle: this.WALLS_COLOR,
                        visible: true,
                    },
                });
            });
        },
        BALLS_HEIGHT_STOP() {
            return this.WALLS_Y0 + 40;
        },
    },
    methods: {
        createBorderRectangle: function (x: number, y: number, W: number, H: number): Matter.Body {
            return this.Bodies.rectangle(x, y, W, H, {
                isStatic: true,
                sleepThreshold: Infinity,
                render: {
                    fillStyle: this.BORDER_COLOR,
                    visible: true,
                },
            });
        },
        createParallelogram: function (center: { x: number; y: number }, vertices: { x: number; y: number }[]) {
            return Matter.Body.create({
                position: center,
                vertices: vertices,
                sleepThreshold: Infinity,
                isStatic: true,
                render: {
                    fillStyle: this.BORDER_COLOR,
                    visible: true,
                },
            });
        },
        createRowNails: function (row: { id: number; nails: number }) {
            let delta: number;
            let center_x_0: number;
            let center_y: number;
            if (row["id"] % 2 !== 0) {
                delta = (row["nails"] - 1) / 2;
                center_x_0 = this.CANVAS_WIDTH / 2 - this.NAILS_X_MARGIN * delta;
                center_y = this.NAILS_Y_INIT + this.NAILS_RADIUS + (this.NAILS_Y_MARGIN + 2 * this.NAILS_RADIUS) * (row["id"] - 1);
            } else {
                delta = row["nails"] / 2;
                center_x_0 = this.CANVAS_WIDTH / 2 - this.NAILS_X_MARGIN * delta + this.NAILS_X_MARGIN / 2;
                center_y = this.NAILS_Y_INIT + this.NAILS_RADIUS + (this.NAILS_Y_MARGIN + 2 * this.NAILS_RADIUS) * (row["id"] - 1);
            }
            const rowNails = [...Array(row["nails"]).keys()].map((i) => {
                return this.Bodies.circle(center_x_0 + i * this.NAILS_X_MARGIN, center_y, this.NAILS_RADIUS, {
                    frictionStatic: 0.00001,
                    isStatic: true,
                    sleepThreshold: Infinity,
                    render: {
                        fillStyle: this.NAILS_COLOR,
                    },
                });
            });
            return rowNails;
        },
        aleatorioEntre: function (a: number, b: number) {
            return a + Math.random() * (b - a);
        },
        createBall: function () {
            const ballX = this.aleatorioEntre(this.BORDER_WIDTH, this.CANVAS_WIDTH - this.BORDER_WIDTH);
            const ball = this.Bodies.circle(ballX, this.BORDER_WIDTH, this.BALLS_RADIUS, {
                friction: this.BALLS_FRICTION,
                restitution: this.BALLS_RESTITUTION,
                density: this.BALLS_DENSITY,
                frictionAir: this.BALLS_FRICTION_AIR,
                sleepThreshold: this.BALLS_SLEEP_TRESHHOLD,
            });
            return ball;
        },
        simulate: function (engine: Matter.Engine) {
            setInterval(() => {
                if (this.isRunningSimulation) {
                    const ball = this.createBall();
                    Matter.Events.on(ball, "sleepStart", () => {
                        Matter.Body.setStatic(ball, true);
                        if (ball.position.y < this.BALLS_HEIGHT_STOP) {
                            this.isRunningSimulation = false;
                        }
                    });
                    this.World.add(engine.world, ball);
                }
            }, this.BALLS_MICROSECONDS_APPEAR);
        },
    },
    mounted() {
        const engine = this.Engine.create({ enableSleeping: true });
        const render = this.Render.create({
            element: document.getElementById("galtonCanvasContainer") as HTMLElement,
            engine: engine,
            options: {
                width: this.CANVAS_WIDTH,
                height: this.CANVAS_HEIGHT,
                wireframes: false,
                background: this.CANVAS_BACKGROUND_COLOR,
            },
        });

        this.World.add(engine.world, [
            this.borderTop,
            this.borderBottom,
            this.borderRight,
            this.borderLeft,
            this.rampLeftTop,
            this.rampLeftBottom,
            this.rampRightTop,
            this.rampRightBottom,
            ...this.nails,
            ...this.walls,
        ]);

        this.simulate(engine);

        this.Runner.run(engine);
        this.Render.run(render);
    },
});
</script>

<style scoped>

</style>
