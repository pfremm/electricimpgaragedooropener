// Garage Door Controller
server.log("miHomeGarage Started");

// Sensors are active low 
doorEnabled      <- 0;
openSensor       <- 1;
closedSensor     <- 0;
currentDoorState <- "Closed";
triggerCount     <- 0;


const OPEN_STATE    = "Open";
const OPENING_STATE = "Opening";
const CLOSED_STATE  = "Closed";
const CLOSING_STATE = "Closing";
const PARTIAL_STATE = "Partially Open";

//--------------------------------------------------------------------------------------------------------
// Turn relay on for 1 second and then off
//--------------------------------------------------------------------------------------------------------
function pulseRelay()
{
    hardware.pin7.write(1);
    imp.sleep(0.5);
    hardware.pin7.write(0);
}

//--------------------------------------------------------------------------------------------------------
//
//--------------------------------------------------------------------------------------------------------
agent.on("buttonPressed", function( value ) 
{
    if( value )
    {
        server.log("button pressed event - HTTP: " + value );
        value = value.tolower();
        //server.log(value);
        local startIdx = value.find("button");
        server.log("IDX: " + startIdx );
        if( startIdx != null )
        {
            server.log("inside if for startIdx");
           pulseRelay();
        } else  {
           server.log("else if for startIdx"); 
        }
    }
    
}); 

//--------------------------------------------------------------------------------------------------------
// Event that handles the sensor inputs. The door can be in one of four states
// Open    - "open sensor" is triggered
// Closing - Previous state was Open and no sensors are triggered
// Closed  - "closed sensor" is triggered
// Opening - Previous state was Closed and no sensors are triggered
// Status is sent to a HTTP Request port configured to call an ASP.NET page which will log the data into 
// a SQL database and send an SMS message to a bunch of cellphones.
//--------------------------------------------------------------------------------------------------------
function checkSensorStates()
{
    local portString;
    local stateChanged = false;

    // little debounce
    imp.sleep(0.5);
    server.log("Trigger Count : " + triggerCount++ );
    
    // If the open sensor has changed state
    if( hardware.pin8.read() != openSensor )
    {
        openSensor = hardware.pin8.read();
        currentDoorState = ( openSensor == 0 ) ? OPEN_STATE : CLOSING_STATE;
        stateChanged = true;
    }

    // If the closed sensor has changed state
    if( hardware.pin9.read() != closedSensor )
    {
        closedSensor = hardware.pin9.read();
        currentDoorState = ( closedSensor == 0 ) ? CLOSED_STATE : OPENING_STATE;
        stateChanged = true;
    }
    
    portString = format("%s,%d,%d,%s", hardware.getimpeeid(), openSensor, closedSensor, currentDoorState );
   
    
    // Update the status variable in the agent object
    if( stateChanged )
        agent.send( "doorStatus", currentDoorState );
    
}

//--------------------------------------------------------------------------------------------------------
// Configure pins. PIN7 = Relay Control, PIN1 = Open Sensor, PIN 2 = Closed Sensor
//--------------------------------------------------------------------------------------------------------
hardware.pin7.configure(DIGITAL_OUT);
hardware.pin8.configure(DIGITAL_IN_PULLUP, checkSensorStates );
hardware.pin9.configure(DIGITAL_IN_PULLUP, checkSensorStates );

// Make sure pin is low - relay off
hardware.pin7.write(0);

// Grab initial sensor states
openSensor   = hardware.pin8.read();
closedSensor = hardware.pin9.read();

// Set the initial status of the door so that Web Queries have value
if( openSensor == 0 )
    currentDoorState = OPEN_STATE;
else if( closedSensor == 0 )
    currentDoorState = CLOSED_STATE;
else
    currentDoorState = PARTIAL_STATE;

agent.send( "initialDoorStatus", currentDoorState );

checkSensorStates();