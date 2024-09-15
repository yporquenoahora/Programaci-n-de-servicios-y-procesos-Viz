<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";

    let philosophers = [
        { id: 1, state: "pensando", leftFork: true, rightFork: true },
        { id: 2, state: "pensando", leftFork: true, rightFork: true },
        { id: 3, state: "pensando", leftFork: true, rightFork: true },
        { id: 4, state: "pensando", leftFork: true, rightFork: true },
        { id: 5, state: "pensando", leftFork: true, rightFork: true }
    ];

    let forks = [
        { id: 1, available: true },
        { id: 2, available: true },
        { id: 3, available: true },
        { id: 4, available: true },
        { id: 5, available: true }
    ];

    let food = 10;  // Cantidad de comida en la mesa
    let isRunning = true;
    let initialFood = 10; // Valor inicial para reiniciar la comida
    let hasFinished = false; // Variable para saber si la simulación ha finalizado

    const startEating = async () => {
        // Cada filósofo intentará comer de manera concurrente
        philosophers.forEach(async (philosopher, index) => {
            while (food > 0) {
                // Detener si isRunning es falso
                if (!isRunning) return;
                await think(philosopher, index);
                await eat(philosopher, index);
            }
             // Si no hay más comida, la simulación ha terminado
             if (food === 0) {
                hasFinished = true;
                updateVisualization(); // Actualizar la visualización cuando termine
            }
        });
    };

    const think = (philosopher, index) => {
        return new Promise((resolve) => {
            philosopher.state = "pensando";
            updateVisualization();
            setTimeout(() => {
                // Comprobar si se ha detenido la simulación
                if (!isRunning) return resolve();
                resolve();
            }, Math.random() * 2000 + 1000);  // Filósofo piensa por 1-3 segundos
        });
    };

    const eat = (philosopher, index) => {
        return new Promise(async (resolve) => {
            const leftFork = forks[index];
            const rightFork = forks[(index + 1) % 5];

            // Esperar hasta que ambos palillos estén disponibles
            while (!leftFork.available || !rightFork.available) {
                if (!isRunning) return resolve();  // Salir si la simulación se ha detenido
                await waitForForks();
            }

            // Tomar los palillos
            leftFork.available = false;
            rightFork.available = false;
            philosopher.state = "comiendo";
            updateVisualization();

            // Comer por 1-3 segundos y reducir la comida
            setTimeout(() => {
                if (food > 0) {
                    food--;
                }
                // Filósofo libera los palillos
                leftFork.available = true;
                rightFork.available = true;
                philosopher.state = "pensando";
                updateVisualization();
                resolve();
            }, Math.random() * 2000 + 1000);  // Comer entre 1 y 3 segundos
        });
    };

    const waitForForks = () => {
        return new Promise((resolve) => {
            setTimeout(() => {
                if (!isRunning) return resolve();
                resolve();
            }, 100);  // Esperar 100ms antes de volver a intentar tomar los palillos
        });
    };

    const updateVisualization = () => {
        const svg = d3.select("svg");

        // Actualizar filósofos (círculos)
        svg.selectAll("circle.philosopher")
            .data(philosophers)
            .attr("fill", d => d.state === "comiendo" ? "green" : "blue");

        // Actualizar palillos (rectángulos)
        svg.selectAll("rect")
            .data(forks)
            .attr("fill", d => d.available ? "gray" : "red");

        // Actualizar comida (círculos naranjas)
        const foodCircles = svg.selectAll("circle.food").data(d3.range(food));

        foodCircles.exit().remove(); // Remover comida cuando disminuye
        foodCircles.enter()
            .append("circle")
            .attr("class", "food")
            .attr("r", 10)
            .attr("fill", "orange")
            .merge(foodCircles)
            .attr("cx", (d, i) => 250 + 30 * Math.cos((i * 2 * Math.PI) / food))
            .attr("cy", (d, i) => 250 + 30 * Math.sin((i * 2 * Math.PI) / food));
    };

    onMount(() => {
        const svg = d3.select("svg");

        // Dibujar filósofos (círculos)
        svg.selectAll("circle.philosopher")
            .data(philosophers)
            .enter()
            .append("circle")
            .attr("class", "philosopher")
            .attr("cx", (d, i) => 250 + 150 * Math.cos((i * 2 * Math.PI) / 5))
            .attr("cy", (d, i) => 250 + 150 * Math.sin((i * 2 * Math.PI) / 5))
            .attr("r", 40)
            .attr("fill", "blue");

        // Dibujar palillos (rectángulos)
        svg.selectAll("rect")
            .data(forks)
            .enter()
            .append("rect")
            .attr("x", (d, i) => 240 + 170 * Math.cos(((i + 0.5) * 2 * Math.PI) / 5))
            .attr("y", (d, i) => 240 + 170 * Math.sin(((i + 0.5) * 2 * Math.PI) / 5))
            .attr("width", 10)
            .attr("height", 60)
            .attr("fill", "gray");

        startEating();  // Iniciar el ciclo de comer/pensar
    });

    const toggleSimulation = () => {
        if (hasFinished) {
            // Si ha terminado, reiniciar la comida y la simulación
            food = initialFood;
            hasFinished = false;
            isRunning = true;
            startEating();
        } else {
            // Alternar entre parar y reanudar
            isRunning = !isRunning;
            if (isRunning) {
                startEating();  // Reanudar la simulación
            }
        }
    };
</script>

<main>
    <h1>Problema de los Filósofos Comensales</h1>
    <div>
        <button on:click={toggleSimulation}>
            {#if hasFinished}
             Reanudar
            {:else}
                {isRunning ? 'Parar' : 'Reanudar'}
            {/if}
        </button>
    </div>

    <svg width="500" height="500" style="border: 1px solid black"></svg>

    {#if food === 0}
        <h2>¡Todos los filósofos han terminado de comer!</h2>
    {/if}
</main>

<style>
    main {
        text-align: center;
        padding: 1em;
    }

    button {
        padding: 10px 20px;
        margin-bottom: 20px;
    }

    svg {
        display: block;
        margin: 0 auto;
    }
</style>
