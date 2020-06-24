<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <p>
      This is an experimental proof of concept web page intended to judge the feasibility/garner feedback on ways to make expungement more accessible. 
      Please DO NOT use this site to determine your eligibility for expungement. Listed below are expungement resources available in D.C.
    
    </p>
   
    <ul>
      <li><a href="https://www.pdsdc.org/need-legal-advice/record-sealing-and-expungement" target="_blank" rel="noopener">The Public Defenders Service For The District of Columbia</a></li>
      <br>
      <li><a href="https://www.dccourts.gov/services/criminal-matters/sealing-criminal-records" target="_blank" rel="noopener">D.C. Courts Site</a></li>
    </ul>
   
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
