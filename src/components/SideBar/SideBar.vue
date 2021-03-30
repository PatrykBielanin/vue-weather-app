<template>
    <div class="SideBar">
        <button>Search for places</button>

        <h1>ELO</h1>

        <h1>15 stopni</h1>

        <p>Raining</p>

        <section class="footerSideBar">
            <p>Today | Mon, 29 March
                <br /><br />
                Jaworzno</p>
        </section>
    </div>
</template>

<script>
    export default {
        name: "SideBar"
    }
</script>

<style lang="scss" scoped>
    .SideBar{
        width: 25vw;
        height: auto;
        background-color: #00203FFF;
        color: whitesmoke;
        /* border-top-right-radius: 20px; */
        /* border-bottom-right-radius: 20px; */

        display: flex;
        flex-direction: column;
        flex-wrap: nowrap;
        justify-content: center;

        .footerSideBar{
            text-align: center;
        }

        *{
            margin: auto;
        }
    }
</style>
