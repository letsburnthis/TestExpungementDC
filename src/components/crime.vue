<!--order of a vue component is the general html, then the javascript with everything in an object? with the general order of data,
methods, computed. 

Overall setup of this site is that every single question needed to determine eligibility for expungement is in correct order in html.
Only the first question, next question to be answered, and answered questions are visible (to make it impossible to answer questions
out of order or ones that are unnecessary based on those previously answered).

Which question to reveal is determined by "paths" hardcoded into the methods section (probably should throw those in data).
A path is the questions and the associated answers that lead to the possibility of expungement. 
Every time an answer is selected, a method is run that checks all answered questions/values selected and compares it to a path.
If the examined path matches the answers so far, the next question on that path is revealed, if not, another path is checked.

If no path matches, a method is run that looks for questions/answers that mean a charge is not eligible for expungement, if there is a match
a pop up explains why the charge is not eligible.

If the end of a path is reached (meaning all answered questions/answers match the entire path) a method is run that takes the dates that have
been input and determines when the charge will be eligible for expungement, and displays a pop up with that information. -->

<template>
  <div class="everything">
    
    <!--the way the methods work requires that all divs be in the correct order in the html. The method only reveals the divs, it doesn't actually place them-->
    <!--msg is supplied by parent App.vue through prop in this files script section-->
    <h1>{{ msg }}</h1>
    <p>Should explain what will be required to fill this out and how to obtain it</p>
    <!--Every id is the data name+Select so that they can easily be selected in the javascript-->
    <div id="pendingCasesSelect" class="question" @change="stateUpdate('pendingCases')">
      <h1>Do you have any pending cases for a crime other than traffic offenses, disorderly conduct, or an offense that is punishable by a fine only, excluding any ineligible misdemeanor "click for list of ineligable misdemeanors".</h1>
      <form action="">
        <!--the "name" being the same makes it so only one can be selected at a time-->
        <input type="radio" name="pendingCases" value="yes" v-model = "pendingCases"> yes<br>
        <input type="radio" name="pendingCases" value="no" v-model = "pendingCases"> no<br>
        <p> this is {{pendingCases}}</p>
      </form>
    </div>



    <!--if I use @click everything is one "click" behind aka it looks at the value when the click occures rather than the value I am changing it to by clicking-->
    <div id="convictedSelect" class="question" style="display: none" @change="stateUpdate('convicted')">
      <h1>Were you convicted of the crime you are trying to expunge? Please select one.</h1>
      <form action="">
        <!--the "name" being the same makes it so only one can be selected at a time-->
        <input type="radio" name="convicted" value="yes" v-model = "convicted" > yes<br>
        <input type="radio" name="convicted" value="no" v-model = "convicted" > no<br>
        <p> You selected {{convicted}}</p>
      </form>
    </div>



    <!--"stateUpdate" is the method that handles revealing the next question. It's made up of two methods.
    The first hides/resets all answers to questions below the answered question.
    The second reveals the next question in the current path.
    The method knows the current position because statusUpdate passes in a string of the name of the data variable that it modifies when answered-->
    <div id="felonySelect" class="question" style="display: none" @change="stateUpdate('felony')">
      <h1>Was the crime a felony? Please select one.</h1>
      <form action="">
        <!--the "name" being the same makes it so only one can be selected at a time-->
        <input type="radio" name="felony" value="yes" v-model = "felony"> yes<br>
        <input type="radio" name="felony" value="no" v-model = "felony"> no<br>
        <p> this is {{felony}}</p>
      </form>
    </div>


    <!--creates a "searchable" list. all values always exist but are hidden unless the users input letters are contained in the items in the list-->
    <div id="selectedCrimeSelect" class="question" style="display: none" >
        
      <p>Please search the list for the crime. If it on the list, select it. If it is not on the list, select "CRIME NOT IN LIST".</p>
      <br>
      <div class="dropdown" >
        <!--v-model is how vue connects variables in javascript to the html? trim gets rid of white space in input value-->
        <input v-model.trim="inputValue"  class="dropdown-input" type="text" placeholder="Search ineligible misdemeaners" />
        <div class="dropdown-list" @click="stateUpdate('selectedCrime')">
              
          <!--v-show only makes visible when conditions are met. itemVisible is a method-->
          <div @click="selectCrime(item)" v-show="itemVisible(item) && inputValue !=''" v-for="item of ineligibleMisdemeanor" :key="item" class="dropdown-item">
            {{item}}
          </div>
              
          <!--puts this option last in the generated list no matter what is searched for-->
          <div @click="selectCrime('CRIME NOT IN LIST')" v-show="inputValue !=''" class="dropdown-item">
            CRIME NOT IN LIST
          </div>
              
        </div>
      </div>
      <br>
      <br>    
       <p v-show="selectedCrime!=''">You selected {{selectedCrime}}</p>
    </div>

    <div id="failureToAppearSelect" class="question" style="display: none" @change="stateUpdate('failureToAppear')">
      <h1>Was the felony "Failure to appear (§ 23-1327)"?.</h1>
      <form action="">
        <!--the "name" being the same makes it so only one can be selected at a time-->
        <input type="radio" name="failureToAppear" value="yes" v-model = "failureToAppear"> yes<br>
        <input type="radio" name="failureToAppear" value="no" v-model = "failureToAppear"> no<br>
        <p> this is {{failureToAppear}}</p>
      </form>
    </div>

       
    <div id="deferredSentencingAgreementSelect" class="question" style="display: none" @change="stateUpdate('deferredSentencingAgreement')" >
      <h1>Did you complete a deferred sentencing agreement (DSA) in this case?.</h1>
      <form action="">
        <!--the "name" being the same makes it so only one can be selected at a time-->
        <input type="radio" name="deferredSentencingAgreement" value="yes" v-model = "deferredSentencingAgreement"> yes<br>
        <input type="radio" name="deferredSentencingAgreement" value="no" v-model = "deferredSentencingAgreement"> no<br>
        <p> this is {{deferredSentencingAgreement}}</p>
      </form>
    </div>


    <div id="convictionAfterSelect"  class="question" style="display: none" @change="stateUpdate('convictionAfter')">
      <h1>Have you been convicted of any offence (other than a minor offence) AFTER the charge you are trying to expunge?.</h1>
      <form action="">
        <!--the "name" being the same makes it so only one can be selected at a time-->
        <input type="radio" name="convictionAfter" value="yes" v-model = "convictionAfter"> yes<br>
        <input type="radio" name="convictionAfter" value="no" v-model = "convictionAfter"> no<br>
        <p> this is {{convictionAfter}}</p>
      </form>
    </div>


    <div id="seriousConvictionSelect" class="question" style="display: none" @change="stateUpdate('seriousConviction')">
      <h1>Have you EVER been convicted of a felony (other than failure to appear) or an ineligible misdemeanor (see list)?.</h1>
      <form action="">
        <!--the "name" being the same makes it so only one can be selected at a time-->
        <input type="radio" name="seriousConviction" value="yes" v-model = "seriousConviction"> yes<br>
        <input type="radio" name="seriousConviction" value="no" v-model = "seriousConviction"> no<br>
        <p> this is {{seriousConviction}}</p>
      </form>
    </div>


      
    <div id="convictedFailureToAppearSelect" class="question" style="display: none" @change="stateUpdate('convictedFailureToAppear')">
      <h1>Have you ever been convicted of Felony Failure to Appear?.</h1>
      <form action="">
        <!--the "name" being the same makes it so only one can be selected at a time-->
        <input type="radio" name="convictedFailureToAppear" value="yes" v-model = "convictedFailureToAppear"> yes<br>
        <input type="radio" name="convictedFailureToAppear" value="no" v-model = "convictedFailureToAppear"> no<br>
        <p> this is {{convictedFailureToAppear}}</p>
      </form>
    </div>

    <!--built in browser calendar? runs stateUpdate when a button is clicked-->
    <div id="convictedFailureToAppearPaperDateSelect" class="question" style="display: none" >
      <h1>Please enter the date you were last "on paper" for Failure to Appear. Click "Done" when date is correct.</h1>
      <input type="date"  name="convictedFailureToAppearPaperDate" v-model = "convictedFailureToAppearPaperDate"><br>
      <button type="button" @click="stateUpdate('convictedFailureToAppearPaperDate')">Done</button>
      <p>You have entered {{convictedFailureToAppearPaperDate}}</p>
    </div>




      
      
      

    

    <div id="paperedSelect" class="question" style="display: none" @change="stateUpdate('papered')">
      <h1>Was this papered? Please select one.</h1>
      <form action="">
        <!--the "name" being the same makes it so only one can be selected at a time-->
        <input type="radio" name="papered" value="yes" v-model = "papered"> yes<br>
        <input type="radio" name="papered" value="no" v-model = "papered"> no<br>
        <p> this is {{papered}}</p>
      </form>
    </div>



    <div id="offPapersDateExpSelect" class="question" style="display: none" >
      <h1>Please enter the date the crime you want expunged went "off papers" and then click "Done" when date is correct.</h1>
      <input type="date"  name="offPapersDateExp" v-model = "offPapersDateExp"><br>
      <button type="button"  @click="stateUpdate('offPapersDateExp')">Done</button>
      <p>You have entered {{offPapersDateExp}}</p>
    </div>




    <div id="otherMisSelect" class="question" style="display: none" @change="stateUpdate('otherMis')">
      <h1>Have you ever been convicted of a misdemeanor (not including the charge you are trying to expunge)? Please select one.</h1>
      <form action="">
        <!--the "name" being the same makes it so only one can be selected at a time-->
        <input type="radio" name="otherMis" value="yes" v-model = "otherMis"> yes<br>
        <input type="radio" name="otherMis" value="no" v-model = "otherMis"> no<br>
        <p> this is {{otherMis}}</p>
      </form>
    </div>



    
    <div id="offPapersDateMisSelect" class="question" style="display: none" >
      <h1>Please enter the date you were last "on paper" for a misdemeaner other than the charge you want expunged. Click "Done" when date is correct.</h1>
      <input type="date"  name="offPapersDateMis" v-model = "offPapersDateMis"><br>
      <button type="button" @click="stateUpdate('offPapersDateMis')">Done</button>
      <p>You have entered {{offPapersDateMis}}</p>
    </div>



    <div id="otherFelSelect" class="question" style="display: none" @change="stateUpdate('otherFel')">
      <h1>Have you ever been convicted of a Felony (not including the charge you are trying to expunge)? Please select one.</h1>
      <form action="">
        <!--the "name" being the same makes it so only one can be selected at a time-->
        <input type="radio" name="otherFel" value="yes" v-model = "otherFel"> yes<br>
        <input type="radio" name="otherFel" value="no" v-model = "otherFel"> no<br>
        <p> this is {{otherFel}}</p>
      </form>
    </div>



    <div id="offPapersDateFelSelect" class="question" style="display: none" >
      <h1>Please enter the date you were last "on paper" for a Felony other than the charge you want expunged. Click "Done" when date is correct.</h1>
      <input type="date"  name="offPapersDateFel" v-model = "offPapersDateFel"><br>
      <button type="button" @click ="stateUpdate('offPapersDateFel')" >Done</button>
      <p>You have entered {{offPapersDateFel}}</p>
    </div>

    
    <div id="caseTerminatedDateSelect" class="question" style="display: none" >
      <h1>Please enter the date the case you are trying to expunge was terminated. Click "Done" when date is correct.</h1>
      <input type="date"  name="caseTerminatedDate" v-model = "caseTerminatedDate"><br>
      <button type="button" @click="stateUpdate('caseTerminatedDate')">Done</button>
      <p>You have entered {{caseTerminatedDate}}</p>
    </div>






    <div id="tail-er">
      <br>
      <br>
      <br>
      <br>
      <br>
      <br>
      <br>
      <br>
      <br>
      
      <p>Just taking up space</p>

      <br>
      <br>
      <br>

    </div>





      <!--made an id because individual id's can be made to render/not render from methods-->  
    
    



    

  </div>
  

  
</template>

<script>




//testing importing from assets???
//import crimes from './assets/crimes.vue'

//export information so it can be imported into App.vue
export default {
  name: 'crime',
  props: {
    msg: String
  },
  data() {
      return{
      //convicted
      convicted:"",

      //date case terminated
      caseTerminatedDate:"",

      //felony
      felony:"",


      //crime data
      pendingCases:"",
      failureToAppear:"",
      inputValue:"",
      selectedCrime:"",

      //deffered sentencing agreement
      deferredSentencingAgreement:"",

      //blocking convictions?
      convictionAfter:"",
      seriousConviction:"",

      convictedFailureToAppear:"",
      convictedFailureToAppearPaperDate:"",

      //put array in seperate json because original was giant and made it hard to find things in code
      //real lists/legal term from DC 16-801 definitions
      ineligibleMisdemeanor:require("../assets/ineligibleMisdemeanors.json"),
      eligibleFelonies:["failure to appear (§ 23-1327)"],
      minorOffense:["traffic offenses", "disorderly conduct", "offense that is punishable by a fine only, excluding any ineligible misdemeanor"],
      
      selectedCrime:"",
      date:"",

      //papered data
      papered:"",

      otherMis:"",
      otherFel:"",


      offPapersDateExp:"",
      offPapersDateMis:"",
      offPapersDateFel:"",

      
     
    }
  },
  methods:{
   


    //search methods
    //loop exists in v-for in template section. takes every "item" in array, one at a time. makes it lowercase
    //makes user input lowercase, and if userinput is in item, displays item. otherwise item remains invisible because of the v-show conditions
    itemVisible (item) {
      let currentName = item.toLowerCase()
      let currentInput = this.inputValue.toLowerCase()
      return currentName.includes(currentInput)
    },

    selectCrime (theItem) {
      this.selectedCrime = theItem;
      this.inputValue = ""
      
    
  },


    


   


    //decided it would be easier create a single method that is called on every button change that checks the state of buttons to see if there are any combinations that determine expugeability
    
    
   //runs anytime an answer is changed. on a high level it sets all questions that come after the currently answered question to invisible
   //and set the answers to "". it then determines what question comes next and displays it. The reason for the reseting of questions
   //is it allows a user to go back if they have made a mistake without having to completely reset the page/start over. They can go to where the
   //mistake occured and all answers above that one are preserved (since this is set up as a logical tree, changing an answer only impact questions that come after).
    stateUpdate(changedQuestion){
      this.resetBelow(changedQuestion);
      this.displayNext();
    },



    //compares current position (passed in when an answer is selected) to an array of all questions in the same order as they exist in the html
    //it then sets all entries in the array after the current one to display "none" and sets associeted data variables to ""
    resetBelow(currentPositionName){
      //this needs to match the allDiv in displayNext I think since position matters (should probably create a master list in data to  make life easier)
      var allDiv=[[this.pendingCases,"pendingCases"], 
      [this.convicted, "convicted"],
      [this.felony,"felony"], 
      [this.selectedCrime,"selectedCrime"],
      [this.failureToAppear, "failureToAppear"], 
      [this.deferredSentencingAgreement,"deferredSentencingAgreement"], 
      [this.convictionAfter, "convictionAfter"],
      [this.seriousConviction, "seriousConviction"],
      [this.convictedFailureToAppear, "convictedFailureToAppear"], 
      [this.convictedFailureToAppearPaperDate, "convictedFailureToAppearPaperDate"],
      [this.papered,"papered"], 
      [this.offPapersDateExp,"offPapersDateExp"], 
      [this.otherMis,"otherMis"],
      [this.offPapersDateMis,"offPapersDateMis"], 
      [this.otherFel,"otherFel"], 
      [this.offPapersDateFel,"offPapersDateFel"],
      [this.caseTerminatedDate,"caseTerminatedDate"]];


      
     

      
      var currentPosition;
      var i=0;
      //finds current position in list of divs by basically looking at each entry in the array and counting until it finds one that matches
      for (currentPosition of allDiv){
        
        if (currentPositionName == currentPosition[1]){
          i=i+1;
          break;
        }
        else{
          i=i+1;
        }
      }
      //sets all divs (questions) below current div to "none" (making them not exist) and sets the answer values associeted to those divs to ""
      //then takes the previously found position(i) and iterates through the list moving to the next item in the array until it hits the end of the array
      while(i < allDiv.length){
     
        var id = allDiv[i][1]+"Select"
        document.getElementById(id).style.display = "none";
        var toRun = "this."+allDiv[i][1]+"= ''";
       //apparently this is bad for security? but do I need to worry about security? should probably try and find way to prgramatically reset all the values below the value that is changed
       eval(toRun);

        i=i+1;



        
      }

      



    },



    displayNext(){
      //all possible successful paths. array of arrays with name and required value. needs to check which path it currently matches and then display next question in that path
      //a "!" in the required answer section means the answer doesn't matter and it will just continue on to the next question on the path  
      //FOR THIS PROGRAM TO WORK paths must be in same order as allDivs. If one is before in path and after another in allDiv, program won't work
      
      //Actual innocence. This one is always available for a non-conviction but this is the only case where it is the only one available???
      //might need to add a special instruction in loop where if a path has convicted set as no, if there is no viable path based on the answers it then asks for the cases termination Date since that is now the only viable path
      //need to check if there are any paths where it's possible to for it to end like that
      var innocence = [["pendingCases", "yes"], ["convicted", "no"], ["caseTerminatedDate","!"]];
      //need to add in for every possibilty other wise gonn have to add a lot to the check function since it currently works by checking for an entire path being completed
      //Felonies
      //felony non BRA
      var felonyNonBRA1 = [["pendingCases", "no"],["convicted","no"], ["felony","yes"], ["failureToAppear", "no"],
      ["papered", "!"], ["offPapersDateExp","!"], ["otherMis", "yes" ],["offPapersDateMis","!"],["otherFel","yes"],["offPapersDateFel","!"],["caseTerminatedDate","!"]];

      var felonyNonBRA2 = [["pendingCases", "no"],["convicted","no"], ["felony","yes"], ["failureToAppear", "no"],
      ["papered", "!"], ["offPapersDateExp","!"], ["otherMis", "yes" ],["offPapersDateMis","!"],["otherFel","no"],["caseTerminatedDate","!"]];

      var felonyNonBRA3 = [["pendingCases", "no"],["convicted","no"], ["felony","yes"], ["failureToAppear", "no"],
      ["papered", "!"], ["offPapersDateExp","!"], ["otherMis", "no" ],["otherFel","yes"],["offPapersDateFel","!"],["caseTerminatedDate","!"]];

      var felonyNonBRA4 = [["pendingCases", "no"],["convicted","no"], ["felony","yes"], ["failureToAppear", "no"],
      ["papered", "!"], ["offPapersDateExp","!"], ["otherMis", "no" ],["otherFel","no"],["caseTerminatedDate","!"]];

      //felony BRA non conviction no differment
      var felonyBRANoConvictionNoDiff1 = [["pendingCases", "no"],["convicted","no"],["felony","yes"], ["failureToAppear", "yes"],
      ["deferredSentencingAgreement", "no"],["papered", "!"], ["offPapersDateExp","!"], ["otherMis", "yes" ],["offPapersDateMis","!"],
      ["otherFel","yes"],["offPapersDateFel","!"],["caseTerminatedDate","!"]];

      var felonyBRANoConvictionNoDiff2 = [["pendingCases", "no"],["convicted","no"],["felony","yes"], ["failureToAppear", "yes"],
      ["deferredSentencingAgreement", "no"],["papered", "!"], ["offPapersDateExp","!"], ["otherMis", "no" ],
      ["otherFel","yes"],["offPapersDateFel","!"],["caseTerminatedDate","!"]];

      var felonyBRANoConvictionNoDiff3 = [["pendingCases", "no"],["convicted","no"],["felony","yes"], ["failureToAppear", "yes"],
      ["deferredSentencingAgreement", "no"],["papered", "!"], ["offPapersDateExp","!"], ["otherMis", "yes" ],["offPapersDateMis","!"],
      ["otherFel","no"],["caseTerminatedDate","!"]];

      var felonyBRANoConvictionNoDiff4 = [["pendingCases", "no"],["convicted","no"],["felony","yes"], ["failureToAppear", "yes"],
      ["deferredSentencingAgreement", "no"],["papered", "!"], ["offPapersDateExp","!"], ["otherMis", "no" ],
      ["otherFel","no"],["caseTerminatedDate","!"]];

      



      //felony BRA non conviction with differment (can you use innocence analysis if non-conviction was for differment?)
      var felonyBRANoConvictionDiff1 = [["pendingCases", "no"],["convicted","no"],["felony","yes"], ["failureToAppear", "yes"],
      ["deferredSentencingAgreement", "yes"],["convictionAfter", "no"],["seriousConviction", "no"],["papered", "!"], ["offPapersDateExp","!"], 
      ["otherMis", "yes" ],["offPapersDateMis","!"],["otherFel","yes"],["offPapersDateFel","!"]];

      var felonyBRANoConvictionDiff2 = [["pendingCases", "no"],["convicted","no"],["felony","yes"], ["failureToAppear", "yes"],
      ["deferredSentencingAgreement", "yes"],["convictionAfter", "no"],["seriousConviction", "no"],["papered", "!"], ["offPapersDateExp","!"], 
      ["otherMis", "no" ],["otherFel","yes"],["offPapersDateFel","!"]];

      var felonyBRANoConvictionDiff3 = [["pendingCases", "no"],["convicted","no"],["felony","yes"], ["failureToAppear", "yes"],
      ["deferredSentencingAgreement", "yes"],["convictionAfter", "no"],["seriousConviction", "no"],["papered", "!"], ["offPapersDateExp","!"], 
      ["otherMis", "yes" ],["offPapersDateMis","!"],["otherFel","no"]];

      var felonyBRANoConvictionDiff4 = [["pendingCases", "no"],["convicted","no"],["felony","yes"], ["failureToAppear", "yes"],
      ["deferredSentencingAgreement", "yes"],["convictionAfter", "no"],["seriousConviction", "no"],["papered", "!"], ["offPapersDateExp","!"], 
      ["otherMis", "no" ],["otherFel","no"]];


      //felony BRA conviction
      var felonyBRAConviction1 = [["pendingCases", "no"],["convicted","yes"],["felony","yes"], ["failureToAppear", "yes"],
      ["convictionAfter", "no"],["seriousConviction", "no"],["offPapersDateExp","!"]];

      //Misdemeanors

      //misdeanor eligible non-conviction no differement
      var eligibleNonConvictionNoDiff1 = [["pendingCases", "no"],["convicted","no"],["felony","no"], 
      ["selectedCrime", "CRIME NOT IN LIST"],["deferredSentencingAgreement", "no"],["otherMis", "yes" ],["offPapersDateMis","!"],
      ["otherFel","yes"],["offPapersDateFel","!"],["caseTerminatedDate","!"]];

      var eligibleNonConvictionNoDiff2 = [["pendingCases", "no"],["convicted","no"],["felony","no"], 
      ["selectedCrime", "CRIME NOT IN LIST"],["deferredSentencingAgreement", "no"],["otherMis", "no" ],
      ["otherFel","yes"],["offPapersDateFel","!"],["caseTerminatedDate","!"]];

      var eligibleNonConvictionNoDiff3 = [["pendingCases", "no"],["convicted","no"],["felony","no"], 
      ["selectedCrime", "CRIME NOT IN LIST"],["deferredSentencingAgreement", "no"],["otherMis", "yes" ],["offPapersDateMis","!"],
      ["otherFel","no"],["caseTerminatedDate","!"]];

      var eligibleNonConvictionNoDiff4 = [["pendingCases", "no"],["convicted","no"],["felony","no"], 
      ["selectedCrime", "CRIME NOT IN LIST"],["deferredSentencingAgreement", "no"],["otherMis", "no" ],
      ["otherFel","no"],["caseTerminatedDate","!"]];


      //misdeanor eligible non-conviction with differement
      var eligibleNonConvictionDiff1 = [["pendingCases", "no"],["convicted","no"],["felony","no"], 
      ["selectedCrime", "CRIME NOT IN LIST"],["deferredSentencingAgreement", "yes"],["convictionAfter", "no"],["seriousConviction", "no"],
      ["otherMis", "yes" ],["offPapersDateMis","!"],["caseTerminatedDate","!"]];

      var eligibleNonConvictionDiff2 = [["pendingCases", "no"],["convicted","no"],["felony","no"], 
      ["selectedCrime", "CRIME NOT IN LIST"],["deferredSentencingAgreement", "yes"],["convictionAfter", "no"],["seriousConviction", "no"],
      ["otherMis", "no" ],,["caseTerminatedDate","!"]];

      

      //misdeanor eligible conviction 
      //NEED TO CHECK THIS ONE. why would a misdemeanor conviction be eligible immediatly but a non-conviction not????? need to read actual law or talk to someone
      //CHECK THIS ONE AGAINST ACTUAL TEXT OF LAW!!!!!!!!!!
      var eligibleConviction1 = [["pendingCases", "no"],["convicted","yes"],["felony","no"], 
      ["selectedCrime", "CRIME NOT IN LIST"],["convictionAfter", "no"],["seriousConviction", "no"],["convictedFailureToAppear","no"]];

      //misdemeanor eligible conviction past BRA
      var eligibleConvictionPastBRA1 = [["pendingCases", "no"],["convicted","yes"],["felony","no"], 
      ["selectedCrime", "CRIME NOT IN LIST"],["convictionAfter", "no"],["seriousConviction", "no"],["convictedFailureToAppear","yes"],
      ["convictedFailureToAppearPaperDate","!"],["offPapersDateMis","!"]];

      //misdemeanor ineligible non-conviction no differment
      var inelgibleNonConvictionNoDiff1 = [["pendingCases", "no"],["convicted","no"],["felony","no"], 
      ["selectedCrime", "!"], ["deferredSentencingAgreement", "no"],["papered", "!"], ["offPapersDateExp","!"], ["otherMis", "yes" ],
      ["offPapersDateMis","!"],["otherFel","yes"],["offPapersDateFel","!"],["caseTerminatedDate","!"]];

       var inelgibleNonConvictionNoDiff2 = [["pendingCases", "no"],["convicted","no"],["felony","no"], 
      ["selectedCrime", "!"], ["deferredSentencingAgreement", "no"],["papered", "!"], ["offPapersDateExp","!"], ["otherMis", "no" ],
      ["otherFel","yes"],["offPapersDateFel","!"],["caseTerminatedDate","!"]];

       var inelgibleNonConvictionNoDiff3 = [["pendingCases", "no"],["convicted","no"],["felony","no"], 
      ["selectedCrime", "!"], ["deferredSentencingAgreement", "no"],["papered", "!"], ["offPapersDateExp","!"], ["otherMis", "yes" ],
      ["offPapersDateMis","!"],["otherFel","no"],["caseTerminatedDate","!"]];

       var inelgibleNonConvictionNoDiff4 = [["pendingCases", "no"],["convicted","no"],["felony","no"], 
      ["selectedCrime", "!"], ["deferredSentencingAgreement", "no"],["papered", "!"], ["offPapersDateExp","!"], ["otherMis", "no" ],
      ["otherFel","no"],["caseTerminatedDate","!"]];

      //misdemeanor ineligible non-conviction with differment
      //why ask about any past felonies if they block it anyways. and what about BRA???
     var inelgibleNonConvictionWithDiff1 = [["pendingCases", "no"],["convicted","no"],["felony","no"], 
      ["selectedCrime", "!"], ["deferredSentencingAgreement", "yes"],["convictionAfter", "no"],["seriousConviction", "no"],
      ["papered", "!"], ["offPapersDateExp","!"], ["otherMis", "yes" ],["offPapersDateMis","!"],["otherFel","yes"],["offPapersDateFel","!"]];

      var inelgibleNonConvictionWithDiff2 = [["pendingCases", "no"],["convicted","no"],["felony","no"], 
      ["selectedCrime", "!"], ["deferredSentencingAgreement", "yes"],["convictionAfter", "no"],["seriousConviction", "no"],
      ["papered", "!"], ["offPapersDateExp","!"], ["otherMis", "no" ],["otherFel","yes"],["offPapersDateFel","!"]];

      var inelgibleNonConvictionWithDiff3 = [["pendingCases", "no"],["convicted","no"],["felony","no"], 
      ["selectedCrime", "!"], ["deferredSentencingAgreement", "yes"],["convictionAfter", "no"],["seriousConviction", "no"],
      ["papered", "!"], ["offPapersDateExp","!"], ["otherMis", "yes" ],["offPapersDateMis","!"],["otherFel","no"]];

      var inelgibleNonConvictionWithDiff4 = [["pendingCases", "no"],["convicted","no"],["felony","no"], 
      ["selectedCrime", "!"], ["deferredSentencingAgreement", "yes"],["convictionAfter", "no"],["seriousConviction", "no"],
      ["papered", "!"], ["offPapersDateExp","!"], ["otherMis", "no" ],["otherFel","no"]];
      


      


      //this needs to match the allDiv in  resetBelow I think since position matters (should probably create a master list in data to  make life easier)
      var allDiv=[[this.pendingCases,"pendingCases"], [this.convicted, "convicted"],[this.felony,"felony"],  [this.selectedCrime,"selectedCrime"],
      [this.failureToAppear, "failureToAppear"],[this.deferredSentencingAgreement,"deferredSentencingAgreement"], [this.convictionAfter, "convictionAfter"],
      [this.seriousConviction, "seriousConviction"],[this.convictedFailureToAppear, "convictedFailureToAppear"],[this.convictedFailureToAppearPaperDate, "convictedFailureToAppearPaperDate"] ,
      [this.papered,"papered"], [this.offPapersDateExp,"offPapersDateExp"], [this.otherMis,"otherMis"],
      [this.offPapersDateMis,"offPapersDateMis"], [this.otherFel,"otherFel"], [this.offPapersDateFel,"offPapersDateFel"],
      [this.caseTerminatedDate,"caseTerminatedDate"]];

      //might add something to move path currently on to first in list (I don't think optomization is super important in this app but maybe)
      //though, moving path currently in use could be useful in determining which analysis program to run... rather than checking the state of the user selections
      

      //all paths listed in this array
      var allPaths = 
      [innocence, 
      felonyNonBRA1, felonyNonBRA2,felonyNonBRA3,felonyNonBRA4,
      felonyBRANoConvictionNoDiff1,felonyBRANoConvictionNoDiff2,felonyBRANoConvictionNoDiff3,felonyBRANoConvictionNoDiff4,
      felonyBRANoConvictionDiff1,felonyBRANoConvictionDiff2,felonyBRANoConvictionDiff3,felonyBRANoConvictionDiff4,
      felonyBRAConviction1, 
      eligibleNonConvictionNoDiff1,eligibleNonConvictionNoDiff2,eligibleNonConvictionNoDiff3,eligibleNonConvictionNoDiff4,
      eligibleNonConvictionDiff1,eligibleNonConvictionDiff2,
      eligibleConviction1,
      eligibleConvictionPastBRA1,
      inelgibleNonConvictionNoDiff1,inelgibleNonConvictionNoDiff2,inelgibleNonConvictionNoDiff3,inelgibleNonConvictionNoDiff4,
      inelgibleNonConvictionWithDiff1,inelgibleNonConvictionWithDiff2,inelgibleNonConvictionWithDiff3,inelgibleNonConvictionWithDiff4]



      var currentSelections =[];
      var div ="";

      //creates list of divs with selections
      for (div of allDiv){
        if (div[0] != ""){
          var input = [div[1],div[0]]
          currentSelections.push(input);
        }
      };
    
      //compares list of current filled divs with paths and finds the path that matches the current filled divs and displays next div in path
      //outer loops, loops through paths, inner loop checks question/answer within a path working it's way through all answered questions. 
      //either displays next question on path, triggers expungement analysis, goes to previous loop to check another path, or if no path is matched it checks conditions for non-expungeability
      var path;
      
      outer_loop:for (path of allPaths){ 
        
        var i = 0;
        //console.log(path);
        var entry;
        for (entry of path){
          //console.logs are for troubleshooting to see last succesful part of loop
          console.log(entry[0]);
          console.log(currentSelections[i]);
          console.log(currentSelections.length);
          console.log(i);
          
          console.log(entry[1]);
          //console.log(currentSelections.length);
          //checks for completed path which means it's time to run analysis
          if(currentSelections[i][0] == entry[0] && (i+1) == path.length && (entry[1]=="!" || entry[1] == currentSelections[i][1])){
            //analyzes expungeability. Need to make sure it covers end of every path 
            this.calculateExpungeableDate();
            break outer_loop;
          }
          //special cases
          //checks for match and checking for an "!" in latest selection, then tells it to continue on to next step in path
          //does this by creating the element's id and then setting it's display style to block (default is none which makes it invisibile/not take up space)
          //the reason order of the items in the html matters is because that determines what order questions show up as they are revealed
          //if one is before another in html but after in allDiv list, when it is revealed in path/allDiv list it will be above an already answered question
          
          else if (currentSelections[i][0] == entry[0] && (i+1) == currentSelections.length && entry[1]=="!"){
            console.log(path[i+1][0]+"Select");
            //takes next entry/question in path array and adds word select to it's name
            var id=path[i+1][0]+"Select";
            //reveals element and moves screen so it is in view
            document.getElementById(id).style.display = "block";
            document.getElementById(id).scrollIntoView({behavior: "smooth"});
            console.log("going to next question");
            break outer_loop;
            
          }
          //if the entry has an "!" just checks the option on the path
          else if (currentSelections[i][0] == entry[0] &&  entry[1]=="!" ){
            i=i+1;
            console.log("I'm looping!'");
          }
          //if the entry and answer match the path and it's the question just answered, it displays the next option on path and stops check
          else if (currentSelections[i][0] == entry[0] && currentSelections[i][1] == entry[1] && (i+1) == currentSelections.length){
            //i+1 should be next question on path
            console.log(path[i+1][0]+"Select");
            var id=path[i+1][0]+"Select";
            document.getElementById(id).style.display = "block";
            document.getElementById(id).scrollIntoView({behavior: "smooth"});
            console.log("going to next question");
            break outer_loop;
          }
          //if the current question that it's checking in path and answer match (but not on most recent question answered yet) checks next question on path
          else if (currentSelections[i][0] == entry[0] && currentSelections[i][1] == entry[1]){
            i=i+1;
            console.log("I'm looping!'");
            }

            //if current question or current answer on path don't match with answers so far, checks another path by breaking this loop (which is checking in a particular path)
          else if (currentSelections[i][0] != entry[0] || currentSelections[i][1] != entry[1]){
           
            
            console.log("I'm trying another path.");
            break;
            
          }
        }
      }
      //if path of allPath completes without finding a suitable path, checks for non-expungeability condition and gives explanation
      //does my "break outer_loop" also lead to this, if it does, is it a problem?
      //even if it does check these at completion of path, shouldn't matter since completing a path means it's not automatically not expungeable and won't meet the conditions
      this.nonExpungeableCheck();
      
      






    },


     










    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    //runs if current answers don't match any path aka not expungeable
    nonExpungeableCheck() {
  
        //Auto non-expungeable scenerios
      if(this.pendingCases == "yes" && this.convicted == "yes"){
        alert("This is not currently uxpungeable. Laws currently do not allow you to expunge a crime you were convicted of if you have a charge currently pending. MAYBE ADD FEATURE TO DETERMINE WHEN/IF IT WILL BE EXPUNGEABLE AFTER CURRENT CASE IS COMPLETED")
      }
      if(this.pendingCases == "no" && this.convicted == "yes" && this.felony == "yes" && this.failureToAppear == "no"){
        alert("This is not uxpungeable under current law. A felony conviction other than 'failure to appear' is currently never expungeable. MAYBE CREATE A FEATURE TO BE NOTIFIED IF LAW CHANGES")
      }
      if(this.pendingCases == "no" && this.convicted == "yes" && this.felony == "no" && (this.selectedCrime != "" && this.selectedCrime != "CRIME NOT IN LIST" )){
        alert("This is not uxpungeable under current law. Misdemeanor convictions on this list are currently not eligable for expungement. MAYBE CREATE A FEATURE TO BE NOTIFIED IF LAW CHANGES")
      }
      //misdemeanor conviction extra checks (might split these into two so people know why exactly they are ineligible)
      if(this.seriousConviction == "yes" || this.convictionAfter == "yes") {
        alert("Current laws do not allow you to expunge a misdemeanor conviction if you have ever been convicted of a serious offence or if you have been convicted of an offence after the charge you are attempting to expunge.")
      }

    },



    //Do I want this to run automatically or do I want the user to opt into it if it's an option?
    //currently runs at the end of a path if conviction = "no" which makes sense since as long as that is true, this is an option
    innocenceAnalysis(){
        console.log(this.caseTerminatedDate);
        var caseTerminated = new Date(this.caseTerminatedDate);
        console.log(caseTerminated);
        var standardChangeDate = new Date(caseTerminated.setFullYear(caseTerminated.getFullYear()+4));
        console.log(standardChangeDate);
        var standardChangeDateString = standardChangeDate.toDateString();
        var innocenceMessage = "";
        //could make this cleaner but honestly it just needs to work, not be elegant
        if (this.pendingCases == "yes"){
          if (standardChangeDate<=new Date()){
            this.innocenceMessage ="This charge may be expunged by 'Clear and Convincing Evidence under statute 16-802'. There may be other grounds for expunging this charge after the pending case is closed, but that cannot be determined until there is an outcome to the pending case.";
         
          };
        
          if (standardChangeDate>new Date()){
            this.innocenceMessage ="This charge may be expunged by a 'preponderance of the Evidence under statute 16-802' until "+standardChangeDateString+" when the standard will change to 'Clear and Convincing Evidence'. There may be other grounds for expunging this charge after the pending case is closed, but that cannot be determined until there is an outcome to the pending case.";
          }

        }
        else{
          if (standardChangeDate<=new Date()){
            this.innocenceMessage ="This charge may be expunged by 'Clear and Convincing Evidence under statute 16-802'." ;
         
          };
          if (standardChangeDate>new Date()){
            this.innocenceMessage ="This charge may be expunged by a 'preponderance of the Evidence under statute 16-802' until "+standardChangeDateString+" when the standard will change to 'Clear and Convincing Evidence'.";
        
         
          }


        }
        
        
      
    },



    
    calculateExpungeableDate(){
      //checks status of previous entries to check what to do (currently runs same program as long as any of these sets of requirements are met)
      //felony, non-conviction
      console.log("Starting expungement analysis");
      //if((this.pendingCases == "no" && this.convicted == "no" && this.felony == "yes" && this.failureToAppear == "no" 
      // && this.offPapersDateExp !="" )
      //||
      //misdemeanor, eligible, non-conviction
      //(this.pendingCases == "no" && this.convicted == "no" && this.felony == "no" && this.selectedCrime == "CRIME NOT IN LIST" 
      // && this.caseTerminatedDate !="" )
      //||
      //felony, failure to appear, non-conviction, no differed sentencing
      //(this.pendingCases == "no" && this.convicted == "no" && this.felony == "yes" && this.failureToAppear == "yes" 
      // && this.deferredSentencingAgreement == "no" && this.offPapersDateExp !="" )
      //){


      //special case where innocense analysis is the only possibility. In all other cases this is in addition to normal expugement
      if(this.pendingCases == "yes" && this.convicted == "no" && this.caseTerminatedDate != ""){
        this.innocenceAnalysis();
        alert(this.innocenceMessage);
      }

      else{
        console.log("I'm setting the time required for expungement");
        //determine how many years you must be off papers to be eligible for expungement
        if(this.papered == "no"){
          var yearsReqForExp = 3
        };
        if(this.papered == "yes"){
          var yearsReqForExp = 4
        };

    
        //eligible misdemeanor, no conviction time
        if(this.papered == "" && this.convicted == "no"){
          var yearsReqForExp = 2
        };
         //eligible misdemeanor, conviction time
        if(this.papered == "" && this.convicted == "yes"){
          var yearsReqForExp = 8
        };
        //replace blank dates (because user didn't have any other misdemeanor or felony charges) with dates that will absolutely be more than 10 years in the past, basically making them not matter
        if(this.offPapersDateMis == ""){
          this.offPapersDateMis = "1990-01-01";
        }
        if(this.offPapersDateFel == ""){
          this.offPapersDateFel = "1990-01-01";
        }
        if(this.offPapersDateExp == ""){
          this.offPapersDateExp = "1990-01-01";
        }
        if(this.caseTerminatedDate == ""){
          this.caseTerminatedDate = "1990-01-01";
        }
        
       
        
        


        //converts calendar entries to date objects and then also creates variables representing the dates when eligible for expungement
        console.log("starting numbering of days");
        var offPapersExp = new Date(this.offPapersDateExp);
        var expungeableDate = new Date(offPapersExp.setFullYear(offPapersExp.getFullYear()+yearsReqForExp));
        console.log(this.caseTerminatedDate);
        var caseTerminated = new Date(this.caseTerminatedDate);
        console.log(this.caseTerminatedDate);
        var expungeableTerminatedDate = new Date(caseTerminated.setFullYear(caseTerminated.getFullYear()+yearsReqForExp));
        var offPapersMisExp= new Date(this.offPapersDateMis);
        var expungeableMisDate = new Date(offPapersMisExp.setFullYear(offPapersMisExp.getFullYear()+5));
        var offPapersFelExp= new Date(this.offPapersDateFel);
        var expungeableFelDate = new Date(offPapersFelExp.setFullYear(offPapersFelExp.getFullYear()+10));
        var dates = [expungeableDate, expungeableMisDate, expungeableFelDate, expungeableTerminatedDate];
        //finds latest date
        var maximumDate=new Date(Math.max.apply(null, dates));
        var maximumDateString = maximumDate.toDateString();
        //compares latest date to current date and notifies if eligible for expungment or when record will be eligible
        
        //took out check for this.caseTerminatedDate != "" for some reason this seemed to be changing that value to "true" after the check????
        //which was impacting the innocense Analysis
        if( this.deferredSentencingAgreement != "yes" && this.convicted == "no"){
          console.log(this.caseTerminatedDate);
          this.innocenceAnalysis();
          if (maximumDate > new Date()){
          alert("This record will be eligible for expungement "+maximumDateString+"."+
          "\r\n"+
          "\r\n"+
          "Another method of expungement available is statute 16-802. "+this.innocenceMessage);
        
          }
          else{
            alert("This record is eligible for expungement."+
            "\r\n"+
            "\r\n"+
            "Another method of expungement available is statute 16-802. "+this.innocenceMessage)
          
         };
        }
        
        else{
          if (maximumDate > new Date()){
            alert("This record will be eligible for expungement "+maximumDateString+".");
        
          }
          else{
            alert("This record is eligible for expungement.")
          
          };
        
        }

      } 
    }
      
    



  },
  
  computed:{
    
    
    //calendar computed
    //not sure if this is the best place for this
    //not sure if I'm even using this anymore
    currentDate: function(){
      return Date() 
    },

    
  }


}




</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.question{
  border-style: groove;
  border-radius: 20px;
}
.dropdown{
  position: relative;
  width: 100%;
  max-width: 90%;
  margin: 0 auto;
}
.dropdown-input, .dropdown-selected{
  width: 100%;
  padding: 10px 16px;
  border: 1px solid transparent;
  background: #edf2f7;
  line-height: 1.5em;
  outline: none;
  border-radius: 8px;
}
.dropdown-input:focus, .dropdown-selected:hover{
  background: #fff;
  border-color: #e2e8f0;
}
.dropdown-input::placeholder{
  opacity: 0.7;
}
.dropdown-selected{
  font-weight: bold;
  cursor: pointer;
}
.dropdown-list{
  position: absolute;
  width: 100%;
  
  max-height: 500px;
  margin-top: 4px;
  overflow-y: auto;
  background: #ffffff;
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
  border-radius: 8px;
}
.dropdown-item{
  display: flex;
  width: 100%;
  padding: 11px 16px;
  cursor: pointer;
}
.dropdown-item:hover{
  background: #edf2f7;
}
.dropdown-item-flag{
  max-width: 24px;
  max-height: 18px;
  margin: auto 12px auto 0px;
}


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
