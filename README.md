# Bitmap Tracking with Trac
This document is supposed to help retrieving indexed & tracked Bitmap data for further processing.

After public release, you may want to self-host Trac. In this case, the described endpoint below will need to get changed into your own endpoint location.

The examples below use websockets instead of classic RESTful API endpoints as these usually serve large amounts of data better. With the release of Trac, RESTful APIs will be available but should only be used for smaller sets of data.

#### Requirements
- Some Javascript knowledge (Node or Browser)
- Socket.io 4.6.1

#### Setup

HTML/JS

```javascript
<script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/4.6.1/socket.io.min.js" integrity="sha512-AI5A3zIoeRSEEX9z3Vyir8NqSMC1pY7r5h2cE+9J6FLsoEmSSGLFaqMQw8SWvoONXogkfFrkQiJfLeHLz3+HOg==" crossorigin="anonymous" referrerpolicy="no-referrer"></script>
```

Node

```npm install socket.io-client```

Code Anatomy

```javascript
// node only!
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
});```

#### Available endpoint getters

```javascript

/**
 * Verifies given inscription ids to be valid parcels.
 *
 * Expects an array of inscription ids (not numbers) with max. 5000 items.
 * Returns an object of equal size with inscription ids as keys and values as either an object with tx_number and bitmap_block or null if not a valid parcel.
 *
 * This can be used to build a history of inscribed parcels.
 * It is recommended to build the history starting from inscription number 10,000,000 to the most recent, as all before will be null.
 *
 * Returns an error string instead of an object if the request failed (atomic).
 *
 * A request can fail if:
 * 'timeout' => request could not be satisfied within 60 seconds
 * 'no array' => the inscription ids parameter value is not an array
 * 'request too large' => the array length of the inscription ids parameter is larger than 5000
 * 'invalid result length' => the key length of the returned object is not equal to the length of the inscription ids parameter array
 *
 */

trac.emit('get',
{
    func : 'inscribedParcelsByInscriptionIds',
    args : [id1, id2, id3, ...],
    call_id : ''
});

/**
 * Verifies which of the given blocks have been inscribed already (post-inscription check).
 *
 * Accepts an array of block numbers.
 *
 * Returns an object with block numbers as keys.
 *
 * Each return value for the keys includes an object with the inscription id and number if inscribed already or null if the state is unknown.
 * Returning null doesn't mean that the requested blocks are valid future bitmap inscriptions.
 * Pre-inscription validations can be done client-side and are not the scope of this tracker.
 *
 * Returns an error string instead of an object if the request failed (atomic).
 *
 * A request can fail if:
 *   'timeout' => request could not be satisfied within 60 seconds
 *   'no array' => the blocks parameter value is not an array
 *   'request too large' => the array length of the blocks parameter is larger than 5000
 *   'invalid block number' => the array contains non-numeric values
 *   'invalid result length' => the key length of the returned object is not equal to the length of the blocks parameter array
 */

trac.emit('get',
{
    func : 'inscribedBitmaps',
    args : [block1, block2, block3, ...],
    call_id : ''
});

```
