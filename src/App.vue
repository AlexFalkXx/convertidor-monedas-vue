<template>
    <!-- Contenedor principal de la app -->
    <div :class="['app-container', theme]">

        <!-- Botón para cambiar entre modo claro y oscuro -->
        <button class="theme-toggle" @click="toggleTheme">
            {{ theme === 'dark' ? '🌞 Modo Claro' : '🌚 Modo Oscuro' }}
        </button>

        <h1>🌎 Convertidor de Monedas</h1>

        <div class="card">

            <!-- Campo para ingresar el monto -->
            <label>Monto</label>
            <input v-model.number="amount" type="number" placeholder="Ingresa un monto" />

            <!-- Selección de moneda de origen -->
            <label>De</label>
            <select v-model="fromCurrency">
                <option v-for="c in currencies" :key="c">{{ c }}</option>
            </select>

            <!-- Selección de moneda destino -->
            <label>A</label>
            <select v-model="toCurrency">
                <option v-for="c in currencies" :key="c">{{ c }}</option>
            </select>

            <!-- Botón para hacer la conversión manual -->
            <button class="convert-btn" @click="convert">Convertir</button>

            <!-- Mostrar error si ocurre -->
            <p v-if="error" class="error">{{ error }}</p>

            <!-- Mostrar resultado si existe -->
            <div v-if="result !== null" class="result-box">
                <p>Resultado:</p>
                <h2>{{ result }}</h2>
            </div>

            <!-- Historial de conversiones -->
            <h3 v-if="history.length">📌 Historial</h3>
            <ul class="history-list">
                <li v-for="(item, index) in history" :key="index" class="history-item">
                    <span>{{ item.text }}</span>

                    <!-- Botón para borrar una entrada del historial -->
                    <button class="delete-btn" @click="deleteHistory(index)">✖</button>
                </li>
            </ul>

            <!-- Gráfico de variación simulada -->
            <div>
                <h3>📈 Gráfico (última semana)</h3>
                <canvas ref="chartCanvas" class="chart"></canvas>
            </div>
        </div>
    </div>
</template>

<script>
    import Chart from "chart.js/auto"; // Importamos Chart.js

    export default {
        data() {
            return {
                // Tema actual (claro/oscuro) guardado en localStorage
                theme: localStorage.getItem("theme") || "dark",

                // Inputs de conversión
                amount: null,
                fromCurrency: "USD",
                toCurrency: "ARS",

                // Resultado y manejo de errores
                result: null,
                error: null,

                // Historial de conversiones
                history: [],

                // Instancia del gráfico
                chart: null,

                // Todas las monedas disponibles
                currencies: [
                    "USD", "ARS", "EUR", "BRL", "GBP",
                    "JPY", "CHF", "CAD", "AUD", "CNY",
                    "MXN", "CLP", "COP", "UYU", "BOB",
                    "PYG", "PEN", "KRW", "RUB", "INR"
                ],

                // API Key válida del usuario
                apiKey: "199b73b415384476b863c103",
            };
        },

        mounted() {
            // Aplica el tema guardado al cargar la app
            document.body.className = this.theme;
        },

        methods: {
            // Cambia entre modo claro y oscuro
            toggleTheme() {
                this.theme = this.theme === "dark" ? "light" : "dark";
                document.body.className = this.theme;
                localStorage.setItem("theme", this.theme);
            },

            // Función principal para convertir divisas
            async convert() {
                this.error = null;
                this.result = null;

                // Evita convertir si el monto no es válido
                if (!this.amount || this.amount <= 0) {
                    this.error = "Ingresa un monto válido";
                    return;
                }

                try {
                    // URL de la API con moneda base
                    const url = `https://v6.exchangerate-api.com/v6/${this.apiKey}/latest/${this.fromCurrency}`;

                    // Petición HTTP
                    const response = await fetch(url);
                    const data = await response.json();

                    // Si la API responde error
                    if (!data || data.result !== "success") {
                        console.error("API error:", data);
                        throw new Error("API failure");
                    }

                    // Buscar tasa de conversión
                    const rate = data.conversion_rates[this.toCurrency];

                    if (!rate) {
                        throw new Error("Rate not found");
                    }

                    // Calcular resultado final
                    this.result = (this.amount * rate).toFixed(2);

                    // Guardar en historial y actualizar gráfico
                    this.addToHistory();
                    this.updateChart();

                } catch (err) {
                    console.error(err);
                    this.error = "Error al obtener la tasa de cambio";
                }
            },

            // Añadir una línea al historial
            addToHistory() {
                const entry = {
                    text: `${this.amount} ${this.fromCurrency} → ${this.result} ${this.toCurrency}`
                };

                this.history.unshift(entry); // Agrega al inicio

                // Limitar historial a 10 items
                if (this.history.length > 10) this.history.pop();
            },

            // Eliminar una conversión específica del historial
            deleteHistory(index) {
                this.history.splice(index, 1);
            },

            // Gráfico de variación simulada (porque la API no da datos históricos)
            updateChart() {
                // Si ya existe un gráfico previo, destruirlo
                if (this.chart) {
                    this.chart.destroy();
                }

                // Etiquetas para una semana simulada
                const labels = [
                    "Hace 7 días",
                    "Hace 6 días",
                    "Hace 5 días",
                    "Hace 4 días",
                    "Hace 3 días",
                    "Hace 2 días",
                    "Hoy"
                ];

                // Valores simulados cercanos al valor real
                const values = Array(7)
                    .fill(0)
                    .map(() => (Math.random() * (1.1 - 0.9) + 0.9).toFixed(2));

                // Crear gráfico con Chart.js
                this.chart = new Chart(this.$refs.chartCanvas, {
                    type: "line",
                    data: {
                        labels,
                        datasets: [
                            {
                                label: "Variación (simulada)",
                                data: values,
                                borderWidth: 2,
                                tension: 0.3,
                            },
                        ],
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                    },
                });
            },
        },
    };
</script>

<style>
    /* Tema oscuro */
    body.dark {
        background: #1e1e1e;
        color: white;
    }

    /* Tema claro */
    body.light {
        background: #f4f4f4;
        color: black;
    }

    /* Contenedor principal */
    .app-container {
        max-width: 420px;
        margin: 30px auto;
        padding: 20px;
        text-align: center;
    }

    /* Tarjeta */
    .card {
        background: rgba(0, 0, 0, 0.15);
        padding: 20px;
        border-radius: 12px;
        margin-top: 10px;
    }

    body.light .card {
        background: #ffffff;
    }

    /* Inputs */
    input, select {
        width: 100%;
        padding: 8px;
        margin: 5px 0 15px;
        border-radius: 6px;
        border: 1px solid #aaa;
    }

    /* Botón de convertir */
    .convert-btn {
        width: 100%;
        padding: 10px;
        background: #4caf50;
        border: none;
        color: white;
        cursor: pointer;
        border-radius: 6px;
        margin-bottom: 12px;
        font-size: 16px;
    }

    /* Errores */
    .error {
        color: #ff4444;
        font-weight: bold;
    }

    /* Botón de tema */
    .theme-toggle {
        padding: 10px 18px;
        border: none;
        border-radius: 8px;
        cursor: pointer;
        margin-bottom: 15px;
    }

    body.dark .theme-toggle {
        background: #fff;
        color: #000;
    }

    body.light .theme-toggle {
        background: #222;
        color: #fff;
    }

    /* Cuadro del resultado */
    .result-box {
        margin-top: 15px;
        background: rgba(80, 80, 80, 0.2);
        padding: 10px;
        border-radius: 8px;
    }

    .chart {
        margin-top: 15px;
        width: 100% !important;
        height: 200px !important;
    }

    /* Historial */
    .history-list {
        padding-left: 0;
        list-style: none;
        margin-top: 10px;
    }

    .history-item {
        display: flex;
        justify-content: space-between;
        margin-bottom: 6px;
    }

    /* Botón de eliminar */
    .delete-btn {
        background: #ff4444;
        border: none;
        color: white;
        border-radius: 4px;
        cursor: pointer;
        padding: 2px 6px;
        font-size: 14px;
    }
</style>
