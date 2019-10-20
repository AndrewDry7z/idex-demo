<template>
    <aside class="menu">
        <nav class="nav">
            <ul class="menu-list">
                <template v-for="item in menuItems">
                    <li class="menu__item" :key="item.title">
                        <button :class="['menu__button', {'active' : item.status === 'active'}]">
                            {{ item.title }}
                        </button>
                    </li>
                </template>
            </ul>
        </nav>
    </aside>
</template>

<script>

    import "./Menu.scss";

    export default {
        name: "Menu",
        data: function () {
            return {
                menuItems: [
                    {title: 'Dashboard', status: 'inactive'},
                    {title: 'Team line up', status: 'active'}
                ]
            };
        }
    }
</script>