<template>
  <div id="app">
    <main class="main">
      <h1 class="main__heading">Рабочий стол руководителя</h1>
      <Menu/>
      <Heading />
      <section class="content">
        <Timeline/>
      </section>
    </main>
  </div>
</template>

<script>

  import "./common.scss";
  import Menu from "./components/Menu/Menu";
  import Timeline from "./components/Timeline/Timeline";
  import Heading from "./components/Heading/Heading";

export default {
  name: 'app',
  components: {
    Menu,
    Timeline,
    Heading
  }
}
</script>
