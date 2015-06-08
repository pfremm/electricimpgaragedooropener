const OPEN_STATE    = "Open";
const OPENING_STATE = "Opening";
const CLOSED_STATE  = "Closed";
const CLOSING_STATE = "Closing";
const PARTIAL_STATE = "Partially Open";

a_currentDoorState <- "Uninitialized ";
timezone <- -6;
ApiKey <- "generate_a_uuid4_and_place_here";
//--------------------------------------------------------------------------------------------------------
// Processes external HTTP requests. These requests contain button press and status requests
//-------------------------------------------------------------------------------------------------------- 
function webServer( request, response )
{
    server.log( "HTTP Request Headers: " + request.headers );
    server.log( "HTTP Request Body: " + request.body );
    server.log( "HTTP Request Method:" + request.method);
    server.log( "HTTP path: " + request.path) ;

    // Needed to avoid  Access-Control-Allow-Origin header error due to AJAX browser security requirements
    // that only allows requests from the domain that served the web page
    response.header( "Access-Control-Allow-Origin", "*" );
    if( request.method=="OPTIONS" )
    {
        response.header("Access-Control-Allow-Methods", "POST, GET, OPTIONS");
        response.header("Access-Control-Allow-Headers", "origin, x-csrftoken, content-type, accept,x-apikey"); 
        response.send(200,"OK");
        return;
    }

    // API Key idea from http://forums.electricimp.com/discussion/comment/8281#Comment_8281
    try {
        if(request.headers["x-apikey"] != null) {
         server.log("API Key " +  request.headers["x-apikey"] ); 
        }
        
    } catch(e)    {
        server.log("Unable to parse x-apikey:" + e);
        response.send(401, errorJSON("Authorization Failed. Invalid Api Key"));
    }
    
    if ("x-apikey" in request.headers && request.headers["x-apikey"] == ApiKey) 
    {
        try{
            local splitURL = split(request.path, "/");
            server.log("splitURL:" + splitURL[1]);
            if(splitURL.find( "door".tolower()) == null)   {
                server.log( "door is in URL" );
                response.send(500, errorJSON("Unknown context root"));
            } else  {
              if(splitURL.find("GetStatus".tolower()) != null)   {  
                response.send(200, statusJSON() );
                server.log("response 200");
              } else if (splitURL.find("buttonpressed".tolower()) != null)    {
                // Send the button ID to the imp
                device.send("buttonPressed", "buttonOpen");
                response.send(200, statusJSON() ); 
              } else    {
                server.log( "Unknown Command String" );
                response.send(500, errorJSON("Unknown Command String"));
              }
                  
            }
        } catch(e)  {
            response.send(500, errorJSON("Unknown context root"));
            server.log("error parsing context root." + e);
        }
    } 
}
 
function statusJSON()   {
    local status = "{\"status\": \"" + a_currentDoorState + "\"}";
    //local status = http.jsonencode(a_currentDoorState);
    return status;
}
function errorJSON(errMsg)  {
    local jsonError = "{\"ErrorMsg\": \"" + errMsg + "\"}";
    return jsonError;
}

//--------------------------------------------------------------------------------------------------------
// HTTP Handler
//-------------------------------------------------------------------------------------------------------- 
http.onrequest(webServer);

//--------------------------------------------------------------------------------------------------------
// 
//--------------------------------------------------------------------------------------------------------
device.on("initialDoorStatus", function(value) {
    
    a_currentDoorState = value;
    
});

//--------------------------------------------------------------------------------------------------------
// 
//--------------------------------------------------------------------------------------------------------
device.on("doorStatus", function(value) {
    a_currentDoorState = value;
    server.log("Door State Set to :" + a_currentDoorState);
    local dateTime = date( time() + (timezone*3600) ); // Adjust for timezone by adding the number of seconds from current time.
    local dateString = format("%d/%d/%d %d:%02d:%02d",dateTime.year,dateTime.month+1,dateTime.day, dateTime.hour, dateTime.min, dateTime.sec );
    
});
