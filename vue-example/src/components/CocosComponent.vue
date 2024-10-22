<template>
    <div>
        <div ref="gameContainer" id="gameContainer"></div>
    </div>
</template>

<script>
export default {
    mounted() {
        this.loadCocosGame();
    },
    methods: {
        loadCocosGame() {
            // Inject the HTML content of the game
            fetch('/web-mobile/index.html')
                .then((response) => response.text())
                .then((htmlContent) => {
                    this.$refs.gameContainer.innerHTML = htmlContent;
                    // Load necessary scripts for the game
                    this.loadScripts([
                        '/web-mobile/src/settings.6586c.js',
                        '/web-mobile/cocos2d-js-min.3878e.js', // Path to cocos2d core library
                        '/web-mobile/main.7c6b0.js', // Adjust to your actual game JS path
                    ])
                        .then(() => {
                            // Call window.boot after all scripts are loaded
                            if (typeof window.boot === 'function') {
                                window.boot();
                            } else {
                                console.error('window.boot is not defined');
                            }
                        })
                        .catch((error) => {
                            console.error('Error loading scripts:', error);
                        });
                })
                .catch((error) => {
                    console.error('Error fetching HTML:', error);
                });
        },
        loadScripts(scripts) {
            return new Promise((resolve, reject) => {
                // Recursively load each script and ensure correct order
                const loadScript = (index) => {
                    if (index >= scripts.length) {
                        resolve(); // Resolve the promise when all scripts are loaded
                        return;
                    }
                    const script = document.createElement('script');
                    script.src = scripts[index];
                    script.onload = () => loadScript(index + 1); // Load next script after the current one finishes
                    script.onerror = () => {
                        reject(new Error(`Failed to load script: ${scripts[index]}`));
                    };
                    document.body.appendChild(script);
                };
                loadScript(0);
            });
        },
    },
};
</script>

<style scoped>
#gameContainer {
    width: 100%;
    height: 50vh; /* Adjust height as needed */
    overflow: hidden;
    position: relative;
}
</style>
