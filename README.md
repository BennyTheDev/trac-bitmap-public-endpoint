# Bitmap Tracking with Trac
This document is supposed to help retrieving indexed & tracked Bitmap data for further processing.
After public release, you may want to self-host Trac. In this case, the described endpoint below will need to get changed into your own endpoint location.

> #### Requirements
> Some Javascript knowledge (Node or Browser)
> Socket.io 4.6.1

#### Setup

HTML/JS
`<script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/4.6.1/socket.io.min.js" integrity="sha512-AI5A3zIoeRSEEX9z3Vyir8NqSMC1pY7r5h2cE+9J6FLsoEmSSGLFaqMQw8SWvoONXogkfFrkQiJfLeHLz3+HOg==" crossorigin="anonymous" referrerpolicy="no-referrer"></script>`

Node

`npm install socket.io-client`

Code Anatomy

`// node only!
const { io } = require("socket.io-client");

// connect to public Bitmap endpoint with Trac

const trac = io("https://bitmap.trac.network", {
    autoConnect : true,
    reconnection: true,
    reconnectionDelay: 500,
    econnectionDelayMax : 500,
    randomizationFactor : 0
});

trac.connect();

// default response event for all endpoint calls.
// this event handles all incoming results for requests performed using "emit".
// Example response object: 
// {
//    "error": "",
//    "func": ORIGINALLY-CALLED-FUNCTION-NAME,
//    "args": ARRAY-WITH-ARGUMENTS,
//    "call_id": OPTIONAL-CUSTOM-VALUE,
//    "result": RETURNED-MIXED-TYPE-VALUE
// }

trac.on('response', async function(msg){
  console.log(msg);
});

// default error event for internal Trac errors, if any

trac.on('error', async function(msg){
    console.log(msg);
});

// example getter to get the transaction size of a block
// the results for this call will be triggered by the 'response' event above.
// this structure is exactly the same for all available getters of this endpoint.

trac.emit('get',
{
    func : 'transactionLength', // the endpoints function to call
    args : [780000],            // the arguments for the function (in this case only 1 argument, the block)
    call_id : ''                // a custom id that is passed through in the 'response' event above to identify for which call the response has been.
});`
