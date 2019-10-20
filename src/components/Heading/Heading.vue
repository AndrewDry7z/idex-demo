<template>
    <div class="heading-flex">
        <div class="heading__text">
            <h2>График отпусков</h2>
        </div>
        <div class="heading__year">
            <button class="heading__year--decrease" @click="decreaseYear()">
                <font-awesome-icon icon="chevron-left"/>
            </button>
            <input type="number" value="2019" min="2012" max="2019" disabled id="year-input"/>
            <button class="heading__year--increase" @click="increaseYear()">
                <font-awesome-icon icon="chevron-right"/>
            </button>
        </div>
    </div>
</template>

<script>


    function decreaseYear() {
        const year = document.querySelector('#year-input');
        if (year.value > year.getAttribute('min')) {
            document.querySelector('#year-input').value--;
        } else {
            return false;
        }
    }

    function increaseYear() {
        const year = document.querySelector('#year-input');
        if (year.value < year.getAttribute('max')) {
            document.querySelector('#year-input').value++;
        } else {
            return false;
        }
    }

    import './Heading.scss';
    export default {
        name: "Heading",
        methods: {
            decreaseYear,
            increaseYear
        }
    }
</script>