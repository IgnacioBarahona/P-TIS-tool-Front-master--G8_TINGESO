<template>
  <router-view/>
</template>

<style lang="scss">
 @import './assets/stylesheets/bulma.css';
 @import './assets/stylesheets/bulma.min.css';

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 30px;

  a {
    font-weight: bold;
    color: #2c3e50;

    &.router-link-exact-active {
      color: #42b983;
    }
  }
}
abbr {
    position: relative;
    text-decoration: underline;
  }
  abbr:hover{
    cursor: help
  }
  abbr:hover::after {
    position: absolute;
    z-index:3;
    bottom: 100%;
    left: 100%;
    display: block;
    padding: 4px;
    background: #5C7380;
    color: white;
    border-radius: 5px;
    width: 150px;
    content: attr(title);
    text-align: center;
    font-style: normal;
    font-size: 13px;
    opacity: .85;
    box-shadow: 5px 5px 3px rgba(0, 0, 0, 0.5)
  }
</style>
