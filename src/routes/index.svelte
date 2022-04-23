<script>
  import Navbar from '../components/navbar.svelte'
</script>

<Navbar/>
<div class="w3-container">
  <h1>Welcome to Ryanauj Site</h1>
  <hr>
  <h2 class="header">Apps</h2>
  <h4 class="header direct_header">On Site Apps</h4>
  <ul>
    <li><a href="https://timesense.tech">TimeSense</a></li>
    <li><a href="/touchtyper">TouchTyper</a></li>
  </ul>
</div>

<style>
  ul {
    list-style-type: '- ';
  }
  hr {
    height: 2px;
    background-color: lightgrey;
  }
  ul {
    margin-top: 0px;
    padding-top: 0px;
  }
  .header {
    text-decoration: underline;
    text-decoration-color: darkcyan;
  }
  .direct_header {
    padding-bottom: 0px;
    margin-bottom: 0px;
  }
</style>


