<template>
  <a href="https://crosby.solutions" target="_blank" class="logo-cs"
    ><span>Created with 💚 by</span>
    <object
      id="logo"
      :data="require('@/assets/cs-logo-one-line.svg')"
      type="image/svg+xml"
      alt="crosby solutions"
      height="16px"
      width="127px"
    ></object>
  </a>
</template>

<style scoped>
span {
  padding-right: 0.5rem;
}
.logo-cs {
  margin: auto auto 5px;
  /* position: absolute;
  bottom: 5px;
  right: 5px; */
  cursor: pointer;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  font-size: small;
  font-family: sans-serif;
}
#logo {
  pointer-events: none;
  vertical-align: bottom;
}
a,
a:hover,
a:visited,
a:active {
  color: black;
  text-decoration: none;
}
</style>
