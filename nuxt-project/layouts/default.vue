<template>
    <main role="main" class="site-main">
        <header class="site-header">
            <nav class="navigation">
                <NuxtLink to="/">Accueil</NuxtLink> <!-- Ajout du lien vers la page index -->
            </nav>
        </header>
        <slot />
        <!-- <footer class="site-footer">
            The footer
        </footer> -->
    </main>
</template>

<style scoped>
.site-main {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
}

.site-header {
    background-color: #00838f;
    color: white;
    padding: 20px 40px;
    font-size: 20px;
    text-align: center;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.navigation {
    display: flex;
    justify-content: center;
    gap: 20px;
}

.site-footer {
    background-color: #006064;
    color: white;
    text-align: center;
    padding: 10px 0;
    margin-top: auto; /* Push the footer to the bottom of the page */
    font-size: 16px;
}
</style>
